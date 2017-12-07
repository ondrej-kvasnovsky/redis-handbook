# Events

Publish / subscribe - decouples sender and receiver.

```
SUBSCRIBE channel
UNSUBSCRIBE channel
PUBLISH channel message
```

Subscribe to multiple channels using `P` pattern. That said, we can subscribe to all channels, like this `PSUBSCRIBE *`.

```
PSUBSCRIBE channel
PUNSUBSCRIBE channel
```

### Keyspace notifications

[Keyspace notifications](https://redis.io/topics/notifications) allow us to get notified when: 

* All the commands affecting a given key.
* All the keys receiving an LPUSH operation
* All the keys expiring in the database 0.

> These notifications are not reliable, once your servers are down and Redis sends the notifications, they are lost forever.



