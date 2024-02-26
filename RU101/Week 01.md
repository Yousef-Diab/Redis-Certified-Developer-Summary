# RU101 Week 1

[Documentation for Key commands at redis.io](https://redis.io/commands#generic)  
[Wikipedia article on Glob style wildcards](<https://en.wikipedia.org/wiki/Glob_(programming)>)

## Keys ( Any Kind of Keys, Lists, Hashes, Sets, Sorted Sets, etc.. )

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

- `DEL key [...keys]`
  - remove the key and the memory associated with the key
  - blocking operation
- `UNLINK key [...keys]`
  - unlink the key
  - memory reclaimed by an async process
  - non-blocking operation
  - returns the number of unlinked keys.
- Keys don't need to exist before is manipulated
- `EXISTS key [...keys]`
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

### Hashes Use Cases

- API Rate Limiting
- Session Storage

### Hashes Commands

- GET
  - `HGET key field`: returns the value of the requested field or (nil).
  - `HGETALL key`: Get all key value pairs of hash.
  - `HMGET key field [...fields]`: returns all values of the requested fields in order.

- SET
  - `HSET key fieldName fieldValue`: Create or set the value of a field in a hash.
  - `HMSET key fieldName fieldValue [...keyValuePairs]`: Create or set the values of multiple fields in a hash.
  - `HSETNX key`: SET if it doesn't exist.

- REMOVE
  - `HDEL key fieldName`: Delete Field.
  - `DEL key`: removes hash and related fields.

- `[HINCRBY|HINCRBYFLOAT] key fieldName incrByValue`: Increment number fields.

- Hash manipulation commands are done in Constant Time O(1) while `HGETALL` is done in O(n) where n is the number of field value pairs in a hash

- An alternative to `HGETALL` for hashes with a lot of fields is `HSCAN` which is used to select required fields only

- Hashes are not very suitable for objects with nested properties, instead, redis JSON is used to store those objects as JSON

### Other Hashes Commands

- `HEXISTS key`: returns 1 if hash exists and 0 otherwise
- `HKEYS key`: get all field names of requested hash key.
- `HVALS key`: get all field values of requested hash key.

### Hashes Notes

- All expiration commands we used on keys previously can be used on hashes directly.

## Lists

- Just like normal programming languages lists.
- can be used to implement stacks & queues.
- redis implements them as doubly linked lists.
- Doesn't store complex data structures (no support for nesting).

### Lists Use cases

- Spotify songs queue
- Inter process communication ( Producer Consumer Pattern ) ( Queue )

### List Commands

- `[R/L]PUSH key value [...values]`: pushes a value to the right/left of the list. - returns the new size of the list.
- `[R/L]POP key`: pops a value to the right/left of the list. - returns the popped value.
- `LRANGE key start stop`: returns the values in a list between the start and stop indecies (0-indexed inclusive), note that a negative value can be used with stop to indicate a starting index from the end of the list.
- `LLEN key`: returns the size of the list.
- `LINDEX key index`: returns the element at the specified index.
- `LSET key index value`: sets the value of a specified index in list.
- `LREM key count value`: removes {count} occurences of value in list starting from the left.
- `LTRIM key start stop`: trims list from start to stop indecies starting from the left.

### Lists Notes

- `push/pop/len` commands are executed in O(1) Time and are independent of the list size.
- `LRANGE`'s time complexity is O(s+n) where n is (stop - start) and s is the distance of start from the head of the list.
- Queue Implementation can be done using the `RPUSH` and `LPOP` Commands.
- Stack Implementation can be done using the `RPUSH` and `RPOP` Commands.

## Sets

- a set is an unordered collection that has no duplicates.
- O(1) Lookup
- Supports standard sets operations i.e Intersection, difference, union
- Doesn't store complex data structures (no support for nesting).

### Sets use cases

- Who's online game widget
- Unique visitors of a URL, Did this IP address pass by me an hour ago?
- is this user online?
- has this user been blacklisted?
- Tag Cloud (Linking tags to certain names).

### Sets Commands

- `SADD key member [...members]`: returns 1 if the member is not duplicate and was added to the set.
- `SCARD key`: returns the number of records inside the set.
- `SISMEMBER key member`: returns 1 if the member exists inside the list.
- `SMEMBERS key`: returns all elements in a set.
- `SSCAN key cursor [MATCH pattern] [COUNT count]`.
- `SREM key member [...members]`.
- `SPOP key [count]`: removes a random element(s) from the set and returns them.

- `SINTER key [...keys]`: returns a list containing the intersection of the given sets.
- `SDIFF key [...keys]`: returns a list containing the difference of the given sets.
- `SUNION key [...keys]`: returns a list containing the distinct common values of the given sets.

#### `[SUNIONSTORE|SDIFFSTORE|SINTERSTORE] destinationKey key [...keys]`: can be used to store the result in a new set specified in the destinationKey, if it already exists, it's overwritten  

## Sorted Sets

- an ordered collection of unique members.
- members added to a set should also have a specified floating point score which is the sorting criteria.
- doesn't support nesting.

### Sorted Sets use cases

- Low latency leaderboard
- priority queues

### Sorted Sets commands

- `ZADD key score member [...scoreMemberPairs] [NX|XX] [CH] [INCR]`
  - XX: Only update elements that already exist. Don't add new elements.
  - NX: Only add new elements. Don't update already existing elements.
  - LT: Only update existing elements if the new score is less than the current score. This flag doesn't prevent adding new elements.
  - GT: Only update existing elements if the new score is greater than the current score. This flag doesn't prevent adding new elements.
  - CH: Modify the return value from the number of new elements added, to the total number of elements changed (CH is an abbreviation of changed). Changed elements are new elements added and elements already existing for which the score was updated. So elements specified in the command line having the same score as they had in the past are not counted. Note: normally the return value of ZADD only counts the number of new elements added.
  - INCR: When this option is specified ZADD acts like ZINCRBY. Only one score-element pair can be specified in this mode.
- `ZINCRBY key incrementValue member`
- `ZRANGE key start stop [WITHSCORES]`: gets the first (stop - start + 1) members in an ascendingly sorted list from lowest to highest score, the WITHSCORES flag returns the associated scores next to the members names.
- `ZREVRANGE key start stop [WITHSCORES]`: same as `ZRANGE` but from highest to lowest.
- `ZRANK key member [WITHSCORES]`: Returns the rank of member in the sorted set stored at key, with the scores ordered from low to high. The rank (or index) is 0-based, which means that the member with the lowest score has rank 0.
- `ZREVRANK key member [WITHSCORES]`: same as `ZRANK` but with scores orders from high to low.
- `ZSCORE key member`: Returns the score of member in the sorted set at key.
- `ZCOUNT key min max`: Returns the number of elements in the sorted set at key with a score between min and max.
- `ZREM key member [...members]`: Removes the specified members from the sorted set stored at key. Non existing members are ignored.

### Sorted Sets Notes

- If 2 members are equal in scores then they're sorted according to lexicographic order.
- Other variations of `ZRANGE` (which retrieves members by position) are `ZRANGEBYSCORE` and `ZRANGEBYLEX` which are self explanatory.
- the above variations also exist on the command `ZREM`
- support for `ZDIFF` was introduced in Redis 6.2.
- Sorted sets has union and intersection operations but cannot be returned directly and should be stored into a destination set. ( `ZINTERSTORE`, `ZUNIONSTORE` )

#### Priority Queue Implementation using Sorted Sets

- top elements can be retreived using `ZRANGE key 0 0`
- then it can be removed using `ZREM key`
- above operations can be combined into a transaction for safer usage.
