# Redis Cheatsheet — Quick Learning & Revision

Goal: concise Redis patterns, commands, .NET usage and best practices.

## Setup & client
- Start: docker run -p 6379:6379 redis
- .NET client: StackExchange.Redis

## Basic commands
- Strings: GET, SET, INCR, SETEX
- Hashes: HSET, HGET, HGETALL
- Lists: LPUSH, RPUSH, LPOP, LRANGE
- Sets: SADD, SMEMBERS
- Sorted Sets: ZADD, ZRANGE
- Pub/Sub: PUBLISH, SUBSCRIBE
- Streams: XADD, XREAD

## Caching pattern (cache-aside)
```csharp
var db = conn.GetDatabase();
var value = await db.StringGetAsync(key);
if (value.IsNull) {
  var data = await LoadFromDb();
  await db.StringSetAsync(key, JsonSerializer.Serialize(data), TimeSpan.FromMinutes(10));
}
```

## Locks & atomic ops
- Simple lock: SET key value NX PX <ms>
- Consider RedLock for distributed locks (understand caveats).

## Pub/Sub & Streams
- Pub/Sub: transient notifications (no persistence).
- Streams: durable message stream with consumer groups.

## Persistence & scaling
- Persistence: RDB and/or AOF. Use replicas for reads, Cluster for sharding.

## Common patterns
- Rate limiting (counter with expiry), distributed cache, session store, leaderboard with sorted sets.

## Real-world example — rate limit
```csharp
var key = $"rate:{userId}";
var count = await db.StringIncrementAsync(key);
if (count == 1) await db.KeyExpireAsync(key, TimeSpan.FromMinutes(1));
if (count > 60) return TooManyRequests();
```

## Advanced Redis topics

### Eviction policies & memory management
- maxmemory-policy options: volatile-lru, allkeys-lru, volatile-ttl, noeviction
- Monitor memory with INFO MEMORY and tune maxmemory to avoid OOM.

### Persistence: RDB vs AOF
- RDB: point-in-time snapshots, fast restart.
- AOF: append-only log with better durability; rewrite needed to compact logs.
- Use both for balance (RDB for snapshots + AOF for durability).

### Lua scripting (atomic operations)
```lua
-- check-and-set example
local current = redis.call('GET', KEYS[1])
if current == ARGV[1] then
  redis.call('SET', KEYS[1], ARGV[2])
  return 1
end
return 0
```
- Execute via ScriptEvaluate in .NET for atomic sequences.

### Redis Modules & advanced features
- RedisJSON for structured storage, RediSearch for indexing/querying, RedisGears for functions.
- Use Streams and consumer groups for durable message workflows.

### Memory & key design tips
- Keep keys small, use namespacing (app:entity:id), set TTL for volatile caches.
- Avoid huge hash values; split large payloads if needed and monitor fragmentation.

### High availability
- Use Redis Sentinel for HA or Redis Cluster for sharding; configure replicas and promotion policies.
