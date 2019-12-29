# Contract Event Log Lifecycle (as it pertains to us)
This is the Red Pill. If you are more of a Blue Pill kind of person - leave now.

## SmartContract
An Event is Declared. It has a Name. It may have *N* arguments. It may have up to *3* Indexed Arguments.

### Technical Points...
An Indexed argument's `keccak(value)` will be present in the Topic array, beginning at index 1
(index 0 is always the hashed canonical signature of the event itself).

    keccak('EventName(argType, argType...)')

Non Indexed argument values are present in the `data` field. It is hashed and concatonated.
TODO: More about `data`...

## Bindings
`abigen` will produce useful constructs for us to access. Take the Computable Datatrust Contract's
`Delivered` Event (notice the contract language here is Vyper):

    Delivered: event({hash: indexed(bytes32), owner: indexed(address), url: bytes32})

Firstly, a `struct` will be generated (notice types have already been mapped):

    type DatatrustDelivered struct {
      Hash  [32]byte
      Owner common.Address
      Url   [32]byte
      Raw   types.Log
    }

Interestingly, `Url` has been hoisted as well - even though it was not Indexed...

Second, an `Iterator` returning method will have been generated specifically for the `Delivered` Event.
It will be located on the Binding's definition of the Datatrust Contract itself, accessible after instantiation:

    // Solidity: event Delivered(bytes32 indexed hash, address indexed owner, bytes32 url)
    func (_Datatrust *DatatrustFilterer) FilterDelivered(opts *bind.FilterOpts, hash [][32]byte, owner []common.Address) (*DatatrustDeliveredIterator, error) {

Funny that the `abigen` created a comment with a Solidity event signature from a Vyper source in a Golang file... anyway.
The iterator will have a `Next` method that, when a `Log` is present, gives you a hydrated `DatatrustDelivered` from
above as the `Iterator.Event`. You will also recieve the lower-level Geth `Log` object itself at `Iterator.Event.Raw`.
In order to fully understand this we need to dive into:

* How the `abigen` generated Contract instance uses its own `FilterLogs` method
* How the `abigen` generated Contract instance uses its own `UnpackLog` method to hydrate the struct
* The Geth `Log` itself

### Aside: The BoundContract
`abigen` produces 3 objects who split responsibilities for _calling_, _transacting_ and _filtering_. They are named for each along the lines of
`FooCaller`, `FooTransactor` and `FooFilterer`. More importantly however, is that each are a pointer to a `bind.BoundContract`

https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L78

#### Aside Aside: Three Interfaces
Worth noting that `caller`, `transactor` and `filterer` are all interfaces here. We *may* want to formalize these ourselves as an
`anti-corruption layer` boundry in a given domain as *reading*, *transacting* and *filtering*. TBD

https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/backend.go#L89

NOTE: that file has a *terribad* name...


## Contract.FilterLogs
https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L247

## Contract.UnpackLog
https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L328

## Geth Log
https://github.com/ethereum/go-ethereum/blob/master/core/types/log.go#L31
