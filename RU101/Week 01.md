# RU101 Week 1

[Documentation for Key commands at redis.io](https://redis.io/commands#generic)  
[Wikipedia article on Glob style wildcards](<https://en.wikipedia.org/wiki/Glob_(programming)>)

## Keys

- used for primary access to data values
- Unique
- Binary Safe: "Foo", 42, 3.1215. 0xff, any sequence of bytes
- Up to 512MB in size, super long keys not recomanded

### Key Spaces

- no classic databasesc but uses Logical databases
- Logical database
  - single flat key space
  - no automatic separation of key names
  - keep it simple, no namespacing complexity
  - identified by a zero-based index
  - default db is: `Database zero`
  - best suited for different key spaces for the same application
  - Restrictions
    - Redis cluster supports only 0
    - many tools asume you use only db0

### Key names structure

    - can use a separator like `:`
        - `user:id:followes` - `user:100:followers`
    - make sure is consistent between teams
    - case sensitive

### Manipulate keys

- `SET key value [EX seconds] [PX miliseconds] [NX|XX]`
  - set TTL
    - EX - expire after seconds
    - PX - expire after miliseconds
  - check for existance
    - NX - not exists
      - set will not work if the key already exists
    - XX - already exists
      - set will work only if the key already exists
  - `SET` Returns Ok if the key was set and (nil) if not.

- `GET key`
  - returns the value or nil

- `KEYS pattern`
  - blocks everything
  - ! Don't use in production
  - usefull for dev
  - `KEYS user:1*`

- `SCAN slot [MATCH pattern] [COUNT count]`

  - iterates using a cursor
  - returns a slot reference
  - safe for production, blocks a small chunk
  - may return 0 or more keys per call
  - use slot to go further
  - bigger count value might block for a longer time
  - returns cursor 0 whan no more keys to iterate
  - Examples:
    - `scan 0 MATCH customer:1*`
      - This is used to have a less command blocking time & Returns 2 Lists:
        - the position of the next slot position to scan and "0" if there's no more.
        - the first matched key value.

    - `scan 0 MATCH customer:1* COUNT 10000`
      - This will make the commad block for longer.

- `DEL key [key ...]`
  - remove the key and the memory associated with the key
  - blocking operation
- `UNLINK key [key ...]`
  - unlink the key
  - memory reclaimed by an async process
  - non-blocking operation
  - returns the number of unlinked keys.
- Keys don't need to exist before is manipulated
- `EXISTS  key [key ...]`
  - 1 exists
  - 0 doesn't exist
  - it is a bad practice to use this command before a `SET` command as the key could be inserted between them by another connection, using the NX/XX attributes of the `SET` command is recommended.

### Keys expiration

- Expiration times set in: miliseconds, seconds or UNIX timestamp
- Keys will eventually be deleted after the expiration time passes
- TTL - time to live
  - `EXPIRE key seconds`
  - `EXPIREAT key timestamp`
  - `PEXPIRE key miliseconds`
  - `PEXPIREAT key miliseconds-timestamp`
- inspect with
  - `TTL key`
  - `PTTL key`
- remove expiration
  - `PERSIST key`
  - TTL will return -1
- can set with `EX` or `PX` params in `SET`
- TTL can be -1 if the key doesn't exist or if it's Persisted (no expiry). // @TODO: Check if this is correct

## Strings

- Binary sequence of bytes
- can store anything: string, integer, binary, comma-separates, json, larger objects like images or videos
- most common use cases are caching [API response, sessions storage, HTML pages]
- great for implementing counters.

- Manipulate with `SET`, `GET`
- clasic usecase:
  - cache database response as JSON to offload the database
- Manipulate as a number, but stored still as string
- `INCR`, `INCRBY`, `DECR`, `DECRBY`
  - increment - `INCR key`
  - if not existed will be created as 0 then increment with 1
  - `INCRBY key -1` can also decrement
  - will throw error if not int
- `INCRBYFLOAT key`
- `TYPE key` - string
- `OBJECT ENCODING key` - eg: int, embstr

- supports Polymorphism - can change data type
- no schema enforcing
