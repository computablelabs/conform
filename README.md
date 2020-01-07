# Conform
**Co**mputable **n**ormalized data **form**at

## Normalized Event Logs
Contract specific Event Logs, referred to as simply an `Event`, are written as:

    | id | address | topics | data | blockNumber | transactionHash | transactionIndex | blockHash | logIndex | removed |

### Normalized EventArguments (indexed and non-indexed[?])
A Smart Contract Event may have `N` number of arguments, 3 of them of which may have been indexed.

    | id | eventLogId | arguments |

#### Assuming...
That Block Number and Index are indeed unique per event. Any other field(s) could be used. TODO

### Field Specifics
Required, types etc...

#### EventLog
* address: required. `type`
* topics: required. `type`
* ...

#### EventArguments
* eventLogId: identifies the Arguments as belonging to a specific SmartContract Event
* arguments: (required - no arg events won't have an entry here). Key, value pairs. JSON?

## Normalized Smart Contract State
We normalize and store state data, provided by specified Smart Contract (callable) methods after any
specified `Event` occurs. A `Snapshot`:

    | id | eventLogId | state |
