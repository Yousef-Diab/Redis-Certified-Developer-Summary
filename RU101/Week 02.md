# RU101 Week 2

## Cardinality & Capped Collections

- Capped Collections is when we want to remove a part of a collection (lists/sets/sorted sets).

### Capping Lists

- `LTRIM key start stop`: keeps the specified elements between start and stop inclusive modifies the list directly unlike `LRANGE`

### Capping Sorted Sets

- `ZREMRANGEBYRANK key start stop`: removes the specified elements between start and stop inclusive.

## Facted Search

## Performance & Big O Notation

### Redis executes commands in a single thread for all clients

### also When a command is executed it's guaranteed to be atomic. So it either fully completes or in a failure case the state of the system is returned back to the state before the command was executed

- Lookup BIG O Notations for each command we used before [here](https://redis.io/commands/)

- O(1) is significant because what it's saying is that regardless of the
cardinality the structure being navigated, then the time complexity is
constant. It does not mean that the wall clock execution for two executions of the
same command is the same.
