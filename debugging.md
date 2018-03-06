# Debugging

Lets see what are the ways to find out what is happening inside Redis.

### Monitoring commands

When we need to see what is Redis doing, we can connect to it and [monitor](https://redis.io/commands/monitor) all the commands that are executed.

```
$ redis-cli -h myservice.redis.mycompany.com monitor
1515185542.880199 [0 1.1.1.1:34578] "hget" "somekey" "-1566720763"
1515185548.931294 [0 1.1.1.1:47306] "exists" "anotherkey"
```

Then we receive all the commands and we can figure out what is wrong.

Then we can dump the redis logs into a file and do a little analyzes. 

```
$ redis-cli -h myservice.redis.mycompany.com monitor > redis-logs.txt
```

Lets create very simple and stupid script that will give us an idea what is happening in redis. Like how many commands were executed. 

```
import java.text.DateFormat
import java.text.SimpleDateFormat
import java.time.Duration

println "Starting..."

File file = new File("/Users/ondrej/Documents/redis-logs.txt")
println "File exits: ${file.exists()}"

boolean firstLineRead = true
String firstLine
String lastLine

int lineCount = 0
Map<String, Integer> aggregator = [:]
file.eachLine { String it ->
    if (it.contains(']')) {
        if (firstLineRead) {
            String[] split = it.split('\\[')
            firstLine = split[0]
            firstLineRead = false
        }
        String[] split = it.split(']')
        String raw = split[1].trim().replaceAll('"', '')
        String[] commands = raw.split(' ')
        String command = commands[0]
        if (aggregator.containsKey(command)) {
            Integer value = aggregator.get(command)
            aggregator.put(command, value + 1)
        } else {
            aggregator.put(command, 1)
        }
        lineCount++
    }
}

DateFormat df = new SimpleDateFormat("ss.SSSSSSS")

int lastLineCounter = 0
file.eachLine { String it ->
    if (it.contains('[')) {
        if (it.contains('keys')) { // there are too many keys called, lets debug and see more about those
            String[] split = it.split('\\[')
            String date = split[0]
            println df.parse(date).toString() + ' ' + it
        }
        if (lineCount - 1 == lastLineCounter) {
            String[] split = it.split('\\[')
            lastLine = split[0]
        }
        lastLineCounter++
    }
}

println 'Line Count: ' + lineCount
println 'Duration: ' + Duration.between(df.parse(firstLine).toInstant(), df.parse(lastLine).toInstant())
println 'From: ' + df.parse(firstLine).toInstant()
println 'To: ' + df.parse(lastLine).toInstant()
println 'Aggregated: ' + aggregator
```

When we run the script we find that there are too many keys command calls, that can be causing performance issues. 

```
Starting...
File exits: true
Tue Mar 06 16:29:52 PST 2018 1520353791.001869 [0 1.1.1.1:42714] "keys" "domain:*"
Tue Mar 06 16:33:34 PST 2018 1520353792.222612 [0 2.2.2.2:33972] "keys" "domain:*"
Tue Mar 06 16:37:36 PST 2018 1520353793.463126 [0 3.3.3.3:48912] "keys" "domain:*"
Tue Mar 06 16:41:09 PST 2018 1520353794.675874 [0 1.1.1.1:42732] "keys" "domain:*"
Tue Mar 06 16:44:31 PST 2018 1520353795.876654 [0 4.4.4.4:58458] "keys" "domain:*"
Tue Mar 06 16:31:27 PST 2018 1520353797.090809 [0 4.4.4.4:58442] "keys" "domain:*"
Tue Mar 06 16:34:49 PST 2018 1520353798.291417 [0 1.1.1.1:42782] "keys" "domain:*"
Tue Mar 06 16:38:17 PST 2018 1520353799.498612 [0 4.4.4.4:58400] "keys" "domain:*"
Tue Mar 06 16:42:02 PST 2018 1520353800.722885 [0 4.4.4.4:58474] "keys" "domain:*"
Tue Mar 06 16:31:06 PST 2018 1520353802.064315 [0 1.1.1.1:42744] "keys" "domain:*"
Tue Mar 06 16:35:18 PST 2018 1520353803.315992 [0 2.2.2.2:33990] "keys" "domain:*"
Tue Mar 06 16:39:19 PST 2018 1520353804.555265 [0 5.5.5.5:51508] "keys" "domain:*"
Tue Mar 06 16:42:48 PST 2018 1520353805.763993 [0 2.2.2.2:34024] "keys" "domain:*"
Tue Mar 06 16:30:20 PST 2018 1520353807.013636 [0 5.5.5.5:51496] "keys" "domain:*"
Tue Mar 06 16:33:45 PST 2018 1520353808.217226 [0 2.2.2.2:34006] "keys" "domain:*"
Tue Mar 06 16:39:02 PST 2018 1520353809.533146 [0 5.5.5.5:51536] "keys" "domain:*"
Tue Mar 06 16:43:02 PST 2018 1520353810.772674 [0 3.3.3.3:48932] "keys" "domain:*"
Tue Mar 06 16:30:16 PST 2018 1520353812.004307 [0 5.5.5.5:51552] "keys" "domain:*"
Tue Mar 06 16:35:02 PST 2018 1520353813.289321 [0 3.3.3.3:48890] "keys" "domain:*"
Tue Mar 06 16:39:16 PST 2018 1520353814.542788 [0 3.3.3.3:48946] "keys" "domain:*"
Tue Mar 06 16:44:55 PST 2018 1520353875.820252 [0 1.1.1.1:42714] "keys" "domain:*"
Tue Mar 06 16:31:51 PST 2018 1520353877.034528 [0 5.5.5.5:51508] "keys" "domain:*"
Tue Mar 06 16:35:27 PST 2018 1520353878.249376 [0 2.2.2.2:33972] "keys" "domain:*"
Tue Mar 06 16:39:00 PST 2018 1520353879.461128 [0 1.1.1.1:42744] "keys" "domain:*"
Tue Mar 06 16:42:32 PST 2018 1520353880.672742 [0 1.1.1.1:42732] "keys" "domain:*"
Tue Mar 06 16:46:06 PST 2018 1520353881.885008 [0 4.4.4.4:58442] "keys" "domain:*"
Tue Mar 06 16:32:51 PST 2018 1520353883.088522 [0 4.4.4.4:58400] "keys" "domain:*"
Tue Mar 06 16:36:26 PST 2018 1520353884.302706 [0 4.4.4.4:58474] "keys" "domain:*"
Tue Mar 06 16:40:05 PST 2018 1520353885.520515 [0 4.4.4.4:58458] "keys" "domain:*"
Tue Mar 06 16:43:36 PST 2018 1520353886.730684 [0 1.1.1.1:42782] "keys" "domain:*"
Tue Mar 06 16:47:43 PST 2018 1520353887.976394 [0 3.3.3.3:48890] "keys" "domain:*"
Tue Mar 06 16:35:09 PST 2018 1520353889.220188 [0 3.3.3.3:48912] "keys" "domain:*"
Tue Mar 06 16:39:13 PST 2018 1520353890.463113 [0 2.2.2.2:34006] "keys" "domain:*"
Tue Mar 06 16:43:33 PST 2018 1520353891.722755 [0 5.5.5.5:51536] "keys" "domain:*"
Tue Mar 06 16:47:33 PST 2018 1520353892.961386 [0 2.2.2.2:33990] "keys" "domain:*"
Tue Mar 06 16:35:06 PST 2018 1520353894.212115 [0 2.2.2.2:34024] "keys" "domain:*"
Tue Mar 06 16:39:26 PST 2018 1520353895.471673 [0 3.3.3.3:48932] "keys" "domain:*"
Tue Mar 06 16:43:48 PST 2018 1520353896.732099 [0 5.5.5.5:51496] "keys" "domain:*"
Tue Mar 06 16:47:43 PST 2018 1520353897.966262 [0 5.5.5.5:51552] "keys" "domain:*"
Tue Mar 06 16:34:39 PST 2018 1520353899.180006 [0 3.3.3.3:48946] "keys" "domain:*"
Tue Mar 06 16:40:00 PST 2018 1520353960.440709 [0 1.1.1.1:42744] "keys" "domain:*"
Tue Mar 06 16:43:51 PST 2018 1520353961.670995 [0 4.4.4.4:58400] "keys" "domain:*"
Tue Mar 06 16:47:54 PST 2018 1520353962.912673 [0 1.1.1.1:42714] "keys" "domain:*"
Tue Mar 06 16:35:42 PST 2018 1520353964.178923 [0 2.2.2.2:33972] "keys" "domain:*"
Tue Mar 06 16:40:02 PST 2018 1520353965.437455 [0 4.4.4.4:58442] "keys" "domain:*"
Tue Mar 06 16:44:05 PST 2018 1520353966.679877 [0 3.3.3.3:48890] "keys" "domain:*"
Tue Mar 06 16:49:23 PST 2018 1520353967.996752 [0 1.1.1.1:42732] "keys" "domain:*"
Tue Mar 06 16:38:03 PST 2018 1520353969.314398 [0 1.1.1.1:42782] "keys" "domain:*"
Tue Mar 06 16:42:49 PST 2018 1520353970.599438 [0 5.5.5.5:51508] "keys" "domain:*"
Tue Mar 06 16:48:44 PST 2018 1520353971.953736 [0 4.4.4.4:58474] "keys" "domain:*"
Tue Mar 06 16:36:45 PST 2018 1520353973.232722 [0 4.4.4.4:58458] "keys" "domain:*"
Tue Mar 06 16:41:17 PST 2018 1520353974.503515 [0 2.2.2.2:34006] "keys" "domain:*"
Tue Mar 06 16:45:30 PST 2018 1520353975.755307 [0 5.5.5.5:51496] "keys" "domain:*"
Tue Mar 06 16:49:11 PST 2018 1520353976.975560 [0 3.3.3.3:48946] "keys" "domain:*"
Tue Mar 06 16:36:12 PST 2018 1520353978.194286 [0 3.3.3.3:48912] "keys" "domain:*"
Tue Mar 06 16:39:54 PST 2018 1520353979.415281 [0 5.5.5.5:51552] "keys" "domain:*"
Tue Mar 06 16:43:40 PST 2018 1520353980.640471 [0 2.2.2.2:34024] "keys" "domain:*"
Tue Mar 06 16:48:06 PST 2018 1520353981.905844 [0 2.2.2.2:33990] "keys" "domain:*"
Tue Mar 06 16:35:27 PST 2018 1520353983.144219 [0 5.5.5.5:51536] "keys" "domain:*"
Tue Mar 06 16:39:48 PST 2018 1520353984.404687 [0 3.3.3.3:48932] "keys" "domain:*"
Tue Mar 06 16:45:06 PST 2018 1520354045.661126 [0 1.1.1.1:42782] "keys" "domain:*"
Tue Mar 06 16:48:48 PST 2018 1520354046.882851 [0 1.1.1.1:42744] "keys" "domain:*"
Tue Mar 06 16:35:51 PST 2018 1520354048.103749 [0 3.3.3.3:48946] "keys" "domain:*"
Tue Mar 06 16:40:11 PST 2018 1520354049.362866 [0 1.1.1.1:42714] "keys" "domain:*"
Tue Mar 06 16:44:28 PST 2018 1520354050.618632 [0 4.4.4.4:58400] "keys" "domain:*"
Tue Mar 06 16:48:13 PST 2018 1520354051.842677 [0 1.1.1.1:42732] "keys" "domain:*"
Tue Mar 06 16:35:36 PST 2018 1520354053.083703 [0 4.4.4.4:58442] "keys" "domain:*"
Tue Mar 06 16:40:04 PST 2018 1520354054.350870 [0 4.4.4.4:58474] "keys" "domain:*"
Tue Mar 06 16:44:12 PST 2018 1520354055.597820 [0 4.4.4.4:58458] "keys" "domain:*"
Tue Mar 06 16:48:17 PST 2018 1520354056.841069 [0 5.5.5.5:51552] "keys" "domain:*"
Tue Mar 06 16:35:30 PST 2018 1520354058.072738 [0 3.3.3.3:48932] "keys" "domain:*"
Tue Mar 06 16:39:23 PST 2018 1520354059.304708 [0 5.5.5.5:51508] "keys" "domain:*"
Tue Mar 06 16:42:57 PST 2018 1520354060.517318 [0 2.2.2.2:34024] "keys" "domain:*"
Tue Mar 06 16:47:42 PST 2018 1520354061.801883 [0 5.5.5.5:51536] "keys" "domain:*"
Tue Mar 06 16:34:47 PST 2018 1520354063.024850 [0 5.5.5.5:51496] "keys" "domain:*"
Tue Mar 06 16:39:21 PST 2018 1520354064.297313 [0 2.2.2.2:33990] "keys" "domain:*"
Tue Mar 06 16:43:04 PST 2018 1520354065.519649 [0 2.2.2.2:33972] "keys" "domain:*"
Tue Mar 06 16:46:50 PST 2018 1520354066.744361 [0 2.2.2.2:34006] "keys" "domain:*"
Tue Mar 06 16:50:33 PST 2018 1520354067.966104 [0 3.3.3.3:48890] "keys" "domain:*"
Tue Mar 06 16:37:41 PST 2018 1520354069.192375 [0 3.3.3.3:48912] "keys" "domain:*"
Tue Mar 06 16:43:02 PST 2018 1520354130.452396 [0 1.1.1.1:42782] "keys" "domain:*"
Line Count: 609242
Duration: PT11M39.199S
From: 2018-03-07T00:31:23.197Z
To: 2018-03-07T00:43:02.396Z
Aggregated: [hget:188212, exists:186372, zadd:92875, evalsha:46748, zrangebyscore:46748, set:46126, keys:81, zcount:2080]
```

### Status of Redis

We can find out a lot by running [info](https://redis.io/commands/info) command. Things like, how much memory is used or how many clients are connected.

```
$ redis-cli -h myservice.redis.mycompany.com
> info

# Server
redis_version:3.2.10
redis_git_sha1:0
redis_git_dirty:0
redis_build_id:0
redis_mode:standalone
os:Amazon ElastiCache
arch_bits:64
multiplexing_api:epoll
gcc_version:0.0.0
process_id:1
run_id:c0b9184187ee7dde58d08fdb4ea2c3709b9577d0
tcp_port:6379
uptime_in_seconds:1556281
uptime_in_days:18
hz:10
lru_clock:5236090
executable:-
config_file:-

# Clients
connected_clients:17
client_longest_output_list:302
client_biggest_input_buf:48
blocked_clients:0

# Memory
used_memory:136252232
used_memory_human:129.94M
used_memory_rss:149471232
used_memory_rss_human:142.55M
used_memory_peak:1384452976
used_memory_peak_human:1.29G
used_memory_lua:40960
used_memory_lua_human:40.00K
maxmemory:4875878400
maxmemory_human:4.54G
maxmemory_policy:volatile-lru
mem_fragmentation_ratio:1.10
mem_allocator:jemalloc-4.0.3

# Persistence
loading:0
rdb_changes_since_last_save:6771981
rdb_bgsave_in_progress:0
rdb_last_save_time:1513629249
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:78146
total_commands_processed:58444357
instantaneous_ops_per_sec:155
total_net_input_bytes:2481357168
total_net_output_bytes:97473675339
instantaneous_input_kbps:9.26
instantaneous_output_kbps:33.78
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:1000064
evicted_keys:0
keyspace_hits:19626326
keyspace_misses:108
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:335.99
used_cpu_user:156554.97
used_cpu_sys_children:0.00
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=251842,expires=251828,avg_ttl=42457658
(12.58s)
```

### How many keys is there

[dbsize](https://redis.io/commands/dbsize) is used to get how many keys our Redis has.

```
$ redis-cli -h myservice.redis.mycompany.com
dbsize
(integer) 257856
(5.03s)
```

### Find slow commands

We can find commands that are slow. The [slowlog](https://redis.io/commands/slowlog) is aggregated in memory and then returned to client.

The most exciting number is `3) (integer) 29747`, which says number milliseconds the command took to execute. `1)` is ID and `2)` is timestamp.

```
myservice.redis.mycompany.com:6379> slowlog get 10
 1) 1) (integer) 3143410
    2) (integer) 1515186728
    3) (integer) 29747
    4) 1) "keys"
       2) "somekey1"
 2) 1) (integer) 3143409
    2) (integer) 1515186728
    3) (integer) 29808
    4) 1) "keys"
       2) "somekey2"
 3) 1) (integer) 3143408
    2) (integer) 1515186728
    3) (integer) 30949
    4) 1) "keys"
       2) "somekey3"
 4) 1) (integer) 3143407
    2) (integer) 1515186728
    3) (integer) 30072
    4) 1) "keys"
       2) "somekey4"
 ...
(20.18s)
```

We can see that we got problem with using [keys](https://redis.io/commands/keys) command too much. There is a warning that `keys` command shouldn't be used with care in production.

