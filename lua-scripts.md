# Lua Scripts

Lua scripts are used to perform code fragments in Redis.

### Performance Issues

Lets say we want to implement this logic for a game. We want to get random user details from redis. We have set with user indexes where key is index and value is time they were selected last time. Then we have set of users where key is index an value is JSON with values. 

We want to get random user indexes first, lets say 100. Then use this list of users and get random user index. Then we use this index to get value from set that contains indexes and JSON details about user. The last step, after all is successful, we want to update the timestamp. 

```
  async getUser(user) {
    const userIndex = await this.userRedisService.getUserIndex(user)

    if (!userIndex) {
      throw new Q.Errors.NoUserAvailableError('Not able to find user index for user', { user })
    }

    const user = await this.userRedisService.findByIndex(userIndex)
    if (!user) {
      throw new Q.Errors.NoUserAvailableError('Not able to find user for user', { user })
    }
    user.userIndex = userIndex
    if (user && user.unlimited) {
      return user
    }

    await this.userRedisService.updateTimestamp(user, userIndex)

    return user
  }
```

This will cause at least 3 calls to Redis. When we execute hundreds or thousands of concurrent requests, we will see performance decrease here. Each call can take about 20-60ms when we call this with 100 concurrent requests. We need to move this into one atomic step using Lua script. 

We should use [EVAL](https://redis.io/commands/eval) command for development of the command. I like to put the Lua script into a bash file so I can edit the script and run it from command line easily. 

```
#!/bin/bash
redis-cli eval "local user = KEYS[1] \
local timestamp =  KEYS[2] \
local from = KEYS[3] \
local to = KEYS[4] \
local penalty = KEYS[5] \
local indexes = redis.call('zrangebyscore', user, '-inf', timestamp, 'limit', from, to) \
local tableLength = table.getn(indexes) \
if tableLength == 0 then \
	return 'NO_USERS_AVAILABLE' \
end \
local index = indexes[math.random(tableLength)] \
local user = redis.call('hget', 'users', index) \
redis.call('zadd', user, timestamp + penalty, index) \
return user" 5 user:game1 $(date +%s000) 0 100 300000
```

When we are done with the script, we need to load it to Redis so it can be compiled only once \(not every time we call the script\). We use [SCRIPT LOAD](https://redis.io/commands/script-load) for this.  

```
➜ redis-cli SCRIPT LOAD "local user = KEYS[1] \
local timestamp =  KEYS[2] \
local from = KEYS[3] \
local to = KEYS[4] \
local penalty = KEYS[5] \
local indexes = redis.call('zrangebyscore', user, '-inf', timestamp, 'limit', from, to) \
local tableLength = table.getn(indexes) \
if tableLength == 0 then \
	return 'NO_USERS_AVAILABLE' \
end \
local index = indexes[math.random(tableLength)] \
local user = redis.call('hget', 'users', index) \
redis.call('zadd', user, timestamp + penalty, index) \
return user"
"some-hash"
```

Then we call the script using its SHA using [EVALSHA](https://redis.io/commands/evalsha). 

```
redis-cli evalsha some-hash 5 user:game1 $(date +%s000) 0 100 300000
```



