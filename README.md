# Conform
**Co**mputable **n**ormalized data **form**at

This is the public facing repository that describes the Conform standard as indicated by the rabbit with a pancake on its head.

<img src="https://i.pinimg.com/originals/40/0b/c3/400bc3cd1aa67d2c81c15f8594e8db16.jpg" width="449" height="337">

## Normalized Event Logs
Contract specific Event Logs, referred to as simply an `Event`, are written as:

    | id | address | topics | data | blockNumber | transactionHash | transactionIndex | blockHash | logIndex | removed |

### Normalized EventArguments (indexed and non-indexed[?])
A Smart Contract Event may have `N` number of arguments, 3 of them of which may have been indexed.

    | id | eventLogId | arguments |

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

### Field Specifics
* eventLogId. identifies this Snapshot as being taken after said Event
* state. Key, value pairs in the form `{method: return}` 
