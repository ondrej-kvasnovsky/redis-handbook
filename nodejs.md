

# NodeJS

Create client

```
const redis = require('redis')
Promise.promisifyAll(redis.RedisClient.prototype)
Promise.promisifyAll(redis.Multi.prototype)

// redis client for sessions
module.exports = async function(config) {
  let redisConfig = config.get('redis', {})
  Q.log.debug('Connecting to redis')
  let client = redis.createClient(redisConfig)
  client.on('error', err => Q.log.error({ err }, 'REDIS ERROR'))
  await new Promise(function(resolve, reject) {
    client.on('ready', function() {
      Q.log.debug('Connected to Redis')
      resolve(client)
    })
  })
  return client
}

```

Replace KEYS with SCAN. 

    async findAllKeys() {
        return this.redis.keysAsync(`${DOMAIN_SET_PREFIX}*`)
    }

The replacement. 

```
  async findAllKeys() {
    const keys = []
    const batchSize = 5000
    let cursor = null
    while (cursor !== '0') {
      if (!cursor) {
        cursor = '0'
      }
      const result = await this.redis.scanAsync(cursor, 'MATCH', 'somePrefix:', 'COUNT', batchSize)
      cursor = result[0]
      const foundKeys = result[1]
      keys.push(...foundKeys)
    }
    return keys
  }
```



