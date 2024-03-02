# RU101 Week 4

## Bitmaps/Bitarrays/Bitfields

### Bitmaps/Bitarrays

- Data type: string.
- An array of bits stored as a string which Redis allows to be handled with constant time complexity.
- Bitmaps also support bitwise operations.

#### Bitmaps Commands

- `SETBIT key offset value`
  - offset represents the position of the bit from the beginning of the map, so if we have a 2d game map, we can maybe set the offset as `Y * MAX_WIDTH + X`.
  - value is `0` or `1`.
  - if we use `SETBIT players:43:map:3 9 1` then bit #9 will be 1 and all bits from 0 through 8 will be initialized as 0.
- `GETBIT key offset`
- `BITCOUNT key [start] [end]`: returns number of bits which values are 1.
  - optional params `start` and `end` are 0-indexed byte-offsets, negative values are accepted.
    - e.g after executing the following:
      - BITFIELD ba4 set u1 7 1 set u1 15 1 set u1 23 1 -> `00000001 00000001 00000001`
      - `BITCOUNT ba4 1 2` returns 2 as it will count the second and third bytes only
- `BITOP [AND|OR|XOR|NOT] destKey key [...key]`
  - e.g:
    - we will execute the following
      - `BITFIELD ba1 set u1 6 1` OR `SETBIT ba1 6 1`. -> `0000 0010`
      - `BITFIELD ba2 set u1 7 1` OR `SETBIT ba2 7 1`. -> `0000 0001`
      - `BITOP or ba3 ba1 ba2`                         -> `0000 0011`
- `BITPOS key bit [start] [end]`
  - returns the position of the first bit equal to `bit`

### Bitfields

#### Bitfield Commands

- `BITFIELD key SET type offset value [...SET type offset value]`: e.g `BITFIELD mykey set u8 0 42` which sets the value 42 using an 8-bit unsigned field.
  - `type` param consists of 2 parts, first is `i` or `u` which means signed and unsigned and next to it is the `size` which is any int.
  - `offset` can be specified either using number of bits or using `#NUMBER` to specifit the index of the offset e.g `BITFIELD mykey set u8 #1 5` will skip size(8) * #number(1) bits before setting the value 5 so the representation will be as `00000000 00000101`
- `BITFIELD key GET type offset [...GET type offset]`
- using `GET key` on a BITFIELD key will return a string containing the Hexa value of the fields inside e.g `get mykey` returns "\x00\x05" which consists of 2 8-bit fields the first containing 0 and the second containing 5.
- `BITFIELD key INCRBY type offset value [...INCRBY type offset value]`

### Bitfields/Bitarrays/Bitmaps notes

- Bitfields commands are a superset of bitarrays commands as the bitarrays commands are like using bitfields commands using u1 type.

### Bitfields/Bitarrays/Bitmaps Use cases

- Keeping track of visited tiles in a game's map using a 2d bitmap where 1 indicates a visited tile.
- store premissions e.g for files.
- representing histograms.
- seats reservation system.

### Bitfields/Bitarrays/Bitmaps Concurrency Issues

- If we use the `WATCH` command for the bitmap key then one of 2 transactions modifying different bits inside it won't work even tho they're not actually colliding.
- to create a custom lock for a specific bit, we can create a temporary key using `SET key NX PX 5000` and if setting it succeeds it means it is not locked and we can continue with the operation, then the key will expire automatically and the bit will be ready to be read by other clients again.

## [Publish / Subscribe](https://redis.io/topics/pubsub)

- The publish and subscribe
functionality in Redis allows for simple message buses to be created. This allows
Redis to act as a broker for multiple clients providing a simple way to post
and consume messages and events.
- Messages sent can be any arbitary binary string like the Redis message data type.

### [Publish/Subcribe Commands](https://redis.io/commands#pubsub)

- Publish/Subscribe is divided into two kinds of syndications:

1. Simple Syndication
2. Patterned Syndication

- Simple
  - `PUBLISH channel message`
    - subscribed clients will recieve a message of 3 parts, first is "message", second is the name of the channel recieving a message from, third is the message's content.
    - if a client is subscribed using `PSUBSCRIBE` then they'll recieve an extra response as the second part which will be the pattern they used to sub with.
  - `SUBCRIBE channel [...channel]`
    - response is of 3 parts, first is "subscribe", second is subscribed channel name and the 3rd of the response is the number of clients after subscribtion.
  - `UNSUBSCRIBE [channel [...channel]]`

- Patterned: used to only recieve messages from channels matching a certain pattern
  - patterns examples:
    - `?` Single character wildcard
    - `*` Multi character wildcard
    - `[...]` Single character alternate
    - `^` Prefix chars
  - `PSUBCRIBE pattern [...pattern]`
  - `PUNSUBSCRIBE [pattern [...pattern]]`

- Admin
  - `PUBSUB CHANNELS [pattern]`
  - `PUBSUB NUMSUB [...channels]`
  - `PUBSUB NUMPAT`

### Publish/Subcribe Notes

- For a single Redis node, the ordering of messages in a channel is guaranteed. ( Order Guaranteed )
- Clients only recieve messages the channel recieved **AFTER** they subscribed to the channel. ( Delivery Not Guaranteed )
- Clients can sub to multiple channels.
- The number of clients acts as a
multiplier for the amount of data that has to be transmitted over the network.
With a one megabyte payload, 100 clients will cause 100 megabytes of data to be
transferred over the network, so this needs some careful planning and consideration.

### Publish/Subcribe Use cases

- Twitch Chat
- Interprocess Communication
- Fan out
