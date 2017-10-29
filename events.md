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



