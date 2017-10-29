# Basics

* All data is accessible via String keys
* Commands are atomic

## Basic Commands

Sets value.

```
SET key value
```

Returns value.

```
GET key
```

Removes a value.

```
DEL key
```

Returns value and then updates the key with a new value.

```
GETSET key value
```

Set value if it does not exist.

```
SETNX key value
```

There are other commands that work for multiple values. They have `M` prefix. Just `DEL` works for multiple by default.

```
MSET
MGET
MSETNX
DEL
```

## Expiration Commands

Expires key after given number of seconds.

```
EXPIRE key seconds
```

Expire at given time.

```
EXPIREAT key timestamp
```

Set value with a timestamp in a single statement.

```
SETEX key seconds value
```

Get expiration date.

```
TTL key
```

Remove expiration time and keep the key persistent.

```
PERSIST key
```

## Other Commands

Increase value without retrieving it first. Good for counters.

```
INCR operation
```

Get part of a large string.

```
GETRANGE operations
SETRANGE operations
```



