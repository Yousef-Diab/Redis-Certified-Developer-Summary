# RU101 Week 3

## [How Redis Handles Clients](https://redis.io/docs/reference/clients/)

## [Transactions](https://redis.io/commands/?group=transactions)

- All the commands that are encapsulated
  in the transaction are serialized and executed sequentially. This guarantees that all the commands are executed in a single isolated operation.

### Main Tranactions Commands

- `MULTI`: Marks the start of a transaction block.
- `EXEC`: Executes all previously queued commands in a transaction.
- `DISCARD`: Flushes all previously queued commands in a transaction. [O(N) -> Number of commands in the transaction.]

### Notes

- Redis Doesn't Support Nested Transactions.
- Redis Offers No Rollback, so any logical, semantic or programming mistakes are not undoable.
- If a queued command operates on the wrong datatype, for example executing an INCR on a List datatype, then **Redis skips this command** but continues to execute the subsequent commands in the queue. So in this circumstance, **it does not cause the Transaction to fail**.

### Optimistic Concurrency/Locking

- Optimistic locking or optimistic
  concurrency control is a mechanism that allows you to specify an interest in an
  object and get a notification if that object has changed.

#### Problem It Solves

- Redis is often used to perform
  hundreds of thousands or even millions of writes a second. If the commands are
  queued up before a transaction is executed, what guarantees do you have
  whether a particular key has been changed by another process or
  application?

- Often you need to ensure that when you come to change a value that another
  process has not changed it. Typically because you've already read the value
  before modifying it. Keyspace notifications is the mechanism that can
  be used in Redis to satisfy this need. There are a couple of commands that make
  it simple to implement this.

#### Optimistic Concurrency Commands

- `WATCH key [...keys]`: When `EXEC` is executed, the transaction will fail if any `WATCH`ed keys are modified.
  - `WATCH` Has to be called before `MULTI` ( Before Tranaction Starts )
  - `WATCH` commands
    do not override previous keys being watched. ( Cumulitive )
  - Local to Client
- `UNWATCH`: Remove all watched keys.
  - All keys are automatically `UNWATCH`ed after calling `EXEC`.

## [Object Storage With Hashes](https://redis.io/commands#hash)

- What if instead of serializing a dictionary into Redis as a JSON string and deserializing it everytime we need to read a property from it, we'll need to read
  the entire structure from the database, transport it through the network and
  deserialize it into a language specific structure.
- We can just use Hashes so we can read it directly

> Remember that hashes doesn't support nesting of complex data types.

- Hashes Fields cannot expire individually.

### Solutions To Store Complex Objects

- Flattening Relationships
- Separating the Object into multiple Hashes

> Redis Don't Automatically Handle Relationships between keys, you have to handle them manually according to your needs. </br>
> Take a look at the "Inventory Control Use Case"
