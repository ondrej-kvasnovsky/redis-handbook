# Debugging

When we need to see what is Redis doing, we can connect to it and [monitor](https://redis.io/commands/monitor) all the commands that are executed. 

```
redis-cli -h myservice.redis.mycompany.com monitor
```

Then we receive all the commands and we can figure out what is wrong. 

