https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/docker/
# Redis database
run the redis server without persistence:  
```
podman run --rm -d --name redis -p 6379:6379 --network foo docker.io/redis/redis-stack-server:7.2.0-v10
```
To enable persistence, add `--save 60 1 --loglevel warning` to the previous command. It will save a snapshot on disk at `/data` every 60 seconds (only if at least 1 operation happened since the last snapshot).

## redis CLI
### connect
connecting via redis-cli:  
```
podman run --rm -it --network foo docker.io/redis/redis-stack-server:7.2.0-v10 redis-cli -h redis
```
### caching
add a key/value string pair:  
```
SET bike:1 "Process 134"
```

get a value:  
```
GET bike:1
```

add a dictionnary (i.e. `Hashes` in Redis terminology):
```
HSET user:1 name John last-name Doe
```

get a dictionnary:
```
HGETALL user:1
```

get a dictionnary value:
```
HGET user:1 name
```

add a JSON array:  
```
JSON.SET animal:0 $ '[  { "cat": 42 } ]'
```

get a JSON array:
```
JSON.GET animal:0 $
```

get an item from a JSON array:
```
JSON.GET animal:0 $[0].cat
```

scan stored values:  
https://redis.io/docs/latest/commands/scan/  
```
SCAN 0 MATCH "*" COUNT 100
```
The first value is the cursor that must be set to 0 when starting iteratin over values. If there are more than the requested values then the Redis server response with a non-zero cursor that should be used in order to iterate over the next page of values.
#### atomic operations
Increment an integer value (integer are stored as string):
```
SET counter 0
INCR counter
```
The INCR operation is atomic meaning if 2 clients are performing it concurrently then the counter is correctly incremented.
All types comes with atomic operations to allow clients to rely on Redis instead of a SQL database.

## persistence
### RDB (Redis database or snapshotting)
If enabled, it saves a snapshot of the data on disk. The interval and number of writes operations between 2 snapshots is configurable.
### AOF (Append only file)
Persists every write operation received by the server. These operations can then be replayed again at server startup, reconstructing the original dataset. The policy to persist write operations is configurable. Default is to persist every second but it can also persist at every write operation (very slow).

### RDB & AOF
Using both persistence strategy will minimize lost of data in case of a disaster while allowing faster restart.

# pub/sub
Publisher/subscriber publishes/subscribes to messages to/from a given channel.  
To publish a message on a `user` channel:  
```
PUBLISH user '{name: "John"}'
```
To subscribe to messages from `user` channel:  
```
SUBSCRIBE user
```
Subscribers receive the messages in the order that the messages are published.  
The delivery guarentee of Redis Pub/Sub is 'at most once' meaning if the subscriber is down when the message is sent then it won't receive the message.  

# Redis streams
Messages in streams are persisted, and support both at-most-once as well as at-least-once delivery semantics.

# misc
TTL when saving data
FAQ:
https://redis.io/docs/latest/develop/get-started/faq/#background-saving-fails-with-a-fork-error-on-linux