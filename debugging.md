# Debugging

Lets see what are the ways to find out what is happening inside Redis. 

### Monitoring commands

When we need to see what is Redis doing, we can connect to it and [monitor](https://redis.io/commands/monitor) all the commands that are executed.

```
redis-cli -h myservice.redis.mycompany.com monitor
```

Then we receive all the commands and we can figure out what is wrong.

### Status of Redis

We can find out a lot by running [info](https://redis.io/commands/info) command. Things like, how much memory is used or how many clients are connected.

```
$ redis-cli -h myservice.redis.mycompany.com
info
```



