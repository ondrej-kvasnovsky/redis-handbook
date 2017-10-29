# Installation and Setup

The easiest way to install Redis is to use Brew. First, install Brew.

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Then we can install Redis.

```
brew install brew
```

Check whether the Redis is running on you system as a service.

```
brew services list
```

If not, start it up like this.

```
brew services start redis
```

Once Redis is started, it is available on port 6379. Default configuration is used and it is located in this file /usr/local/etc/redis.conf.

## Connect

First check if Redis is running. If we send PING, we should receive PONG from Redis server.

```
redis-cli -h localhost -p 6379 PING
```



