# Data types

The default is map - key value store. Then there is Set, that can be used to implement a dictionary. Set is to be used for fast membership checks and multi-set arithmetics \(O\(n\) performance\).

Key-value Map

## Set

Key is name of a [set](https://redis.io/commands#set). All set commands have `S` prefix.

Add an item into set.

```
SADD key member
```

Returns all items from a set.

```
SMEMBERS key
```

Returns 1 if member is contained in a set, 0 otherwise.

```
SISMEMBER key member
```

Get the cardinality \(size\) of set.

```
SCARD key
```

We can do set arithmetics, like union, intersection and difference.

```
SUNION
SINTER
SDIF
```

## Sorted Set

[Set sorted](https://redis.io/commands#sorted_set) by a score, descending.

```
ZADD key [option] score member
ZREM key member
ZCARD key
ZSCORE key member
ZRANK key member
ZCOUNT key min max
ZRANGEBYSCORE key 0 1000 LIMIT 0 1
```

## List

TODO:

