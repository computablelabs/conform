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

There is some masking here. The `BoundContract` handles some formatting then turns this call over to another
`FilterLogs` method:

https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L275

### Contract.FilterLogs.FilterLogs (yo dawg...)
We know that the interface for BoundContract states it must have a `Contract.filterer` which must in turn
posses its own `FilterLogs`. So where is this `filterer` declared and what exacly is this inner `FilterLogs` doing?

* see `DeployContract` https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L100
* see `NewBoundContract` https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L102

The `abigen` bindings call `DeployContract` with `bind.ContractBackend` as `backend` which gets assigned
as `*Caller`, `*Transactor` and `*Filterer`...

https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/backend.go#L108

#### Finally...
Geth puts the low level `FilterLogs` on any acceptend `backend` abstraction of the EVM. For example, the
`SimulatedBackend` used in tests:

https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/backends/simulated.go#L475

Interesting, and typical, that we are dealing with `deprecation` warnings already here (see line 474).

So, how is this done IRL? Looks like the Geth `ethclient` implements the `Filterer` interface:

https://github.com/ethereum/go-ethereum/blob/master/ethclient/ethclient.go#L377

What's `ethclient` then? Its the Geth wrapper around the lowest-level EVM RPC API. Things to note here:

* the internal `toFilterArg` function
* `ethereum.FilterQuery`

This ball-of-yarn is thus unwound at the topmost package level with the following interfaces:

https://github.com/ethereum/go-ethereum/blob/master/interfaces.go#L133

As mentioned above there seems to be _all momentum_ pointing toward using Geth's internal `Subscription`
object to do *both* past logs and future ones. We must watch this...

## Contract.UnpackLog
https://github.com/ethereum/go-ethereum/blob/master/accounts/abi/bind/base.go#L328

## Geth Log
https://github.com/ethereum/go-ethereum/blob/master/core/types/log.go#L31
