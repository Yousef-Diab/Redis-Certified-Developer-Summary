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
- classic usecase:
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
- so if we try to read a number using `GET key` we will get it between quotes indicating a string but if we read it using the `OBJECT ENCODING key` we will get "int", the numbers manipulation commands will decide if the value is suitable to perform an action on depending on the encoding.

- so Redis supports Polymorphism - can change data type
- no schema enforcing

## Hashes

[Documentation](https://redis.io/commands/?group=hash)

- A hash is a mini key value store within a key.
- Hashes are used to store object like structures.
- they're mutable.
- extremely memory effecient.
- not recursive, so fields values are only strings, not complex data strcutures.

### Typical Use Cases

- API Rate Limiting
- Session Storage

### Reading, Adding, Updating and Removing hash field value pairs

- GET
  - `HGET key field`: returns the value of the requested field or (nil).
  - `HGETALL key`: Get all key value pairs of hash.
  - `HMGET key [...fields]`: returns all values of the requested fields in order.

- SET
  - `HSET key fieldName fieldValue`: Create or set the value of a field in a hash.
  - `HSET key fieldName [...fields]`: Create or set the values of multiple fields in a hash.
  - `HSETNX key`: SET if it doesn't exist.

- REMOVE
  - `HDEL key fieldName`: Delete Field.
  - `DEL key`: removes hash and related fields.

- `[HINCRBY|HINCRBYFLOAT] key fieldName incrByValue`: Increment number fields.

- Hash manipulation commands are done in Constant Time O(1) while `HGETALL` is done in O(n) where n is the number of field value pairs in a hash

- An alternative to `HGETALL` for hashes with a lot of fields is `HSCAN` which is used to select required fields only

- Hashes are not very suitable for objects with nested properties, instead, redis JSON is used to store those objects as JSON

### Other Commands

- `HEXISTS key`: returns 1 if hash exists and 0 otherwise
- `HKEYS key`: get all field names of requested hash key.
- `HVALS key`: get all field values of requested hash key.

### Notes

- All expiration commands we used on keys previously can be used on hashes directly.
