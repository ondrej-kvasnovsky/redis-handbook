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

