# Designing Database

If there is no key, there is nothing in the database. There are no indexes, no queries, just keys and its values. Keys exist in a database. You can switch database using `SELECT index` command.

## Manipulating Keys

Check if key exists.

```
EXISTS key
```

Renames a key or rename if key does not exist.

```
RENAME key newkey
RENAMENX key newkey
```

Print all keys for given pattern. Don't use KEYS too much in production, it can slow down Redis a lot.

```
KEYS pattern
```

Remove all keys from database.

```
FLUSHDB
```

## Conventions for key naming

Composite keys that tell a lot are good. For example:

```
SET user:id:123 '{name: "Ondrej"}'
GET user:id:123
```

## Modeling

We need to come up with the way to identify a value. Lets say we use email to identify a user. Then we need to come up with relations, a user could have an account. It all needs to be associated with keys.

## Sensitive data

Things like password should be encrypted.

```
HMSET myhash field1 "Hello" field2 "World"
HMGET myhash field1
```

### Generate Unique ID

There might be a lot of IDs when working with multiple entities. Here is the way to generate a unique ID in Redis.

```
INCR key
```

### Referencing Set

Add items into a set and associate that with an user. Our user has ID 42.

```
SADD user:42:shoppingList milk butter bread
SMEMBERS user:42:shoppingList
```

### Search in Keys

When designing keys, remember we can search in them using [patterns](https://redis.io/commands/keys).

```
MSET 1key 1 2key 2 3key 3
KEYS *key
```

### Atomicity

Use multi commands to ensure atomic operations.

### Index Tables

When you know you would have to duplicate data in many Redis structure, it is good idea to create one index table that contains keys with its values. Then other structures contain only a key into this table, not the whole value.

The result is low memory consumption.

