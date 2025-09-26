Lists, sets, and sorted sets are great for many use cases, but the hash data type is on a higher level. Redis hashes are maps (sequences of field-value pairs) and are used to represent objects in Redis. Consider the following example that shows how to create a hash key using the `HSET` command.

Redis hashes are record types that are structured as name-value pairs. Consider the following example that shows how to create a hash key using the `HSET` command.

```redis:[run_confirmation=true] Create a hash
HSET bike:1 model Deimos brand Ergonom type "Enduro bikes" price 4972
```

`HSET` returns the number of added name-value pairs.

To retrieve the stored data, use the `HGETALL` command.

```redis HGETALL usage
HGETALL bike:1 // returns all the name-value pairs associated with the key
```

If you only want the values of a subset of the fields, use the `HGET` command.

```redis HGET usage
HGET bike:1 price
```

You can update fields in a hash using `HSET` by specifying a subset of its name-value pairs.

```redis:[run_confirmation=true] Update an existing hash
HSET bike:1 model "Kraken" price 3000
```

Use the `HDEL` command to delete one or more fields from a hash. Once all fields are removed, the hash key itself will be deleted.

```redis:[run_confirmation=true] Delete hash fields and keys
HDEL bike:1 model
HGETALL bike:1
HDEL bike:1 brand type price
HGETALL bike:1
EXISTS bike:1
```

Integer values in hash keys can be incremented or decremented in the same way as simple string keys using the `HINCRBY` command.
The increment value must be a positive or negative integer.

```redis Hash INCRBY usage
HINCRBY bike:1 price 100
HINCRBY bike:1 price -100
```

Available in [Redis 8](https://hub.docker.com/layers/library/redis/8.0.3/images/sha256-426e6823fb1778e8c49f327f9e5af00e505a7fca726ffe11b7930eb1d99ef5fd) is the ability set the expiration time or the remaining time-to-live (TTL) for individual hash fields. This is called hash field expiration (HFE). HFE works just like [key expiration](https://redis.io/docs/latest/develop/use/keyspace/?utm_source=redisinsight&utm_medium=main&utm_campaign=tutorials#key-expiration) and includes the following commands:

- `hexpire` - set an expiration (time-to-live or TTL) in seconds on a hash key's field(s).
- `hexpireat` - set a TTL as an absolute Unix timestamp (seconds since Unix epoch) on a hash key's field(s).
- `hexpiretime` - returns the absolute Unix timestamp (seconds since Unix epoch) at which the hash key's field(s) will expire.
- `hpersist` - remove expirations from a hash key's field(s).
- `hpexpire` - set an expiration (time-to-live or TTL) in milliseconds on a hash key's field(s).
- `hpexpireat` - set a TTL as an absolute Unix timestamp (milliseconds since Unix epoch) on a hash key's field(s).
- `hpexpiretime` - returns the absolute Unix timestamp (milliseconds since Unix epoch) at which the the hash key's field(s) will expire.
- `hpttl` - returns the remaining TTL in milliseconds of a hash key's field(s).
- `httl` - returns the remaining TTL in seconds of a hash key's field(s).

```redis HFE example 1
HSET hash field1 "foo" field2 "bar" field3 "baz"
HEXPIRE hash 10 FIELDS 1 field2
HTTL hash FIELDS 1 field2
```

```redis HFE example 1 cont'd
HTTL hash FIELDS 1 field2
HGETALL hash
```

```redis HFE example 2
HSET hash field1 "foo" field2 "bar" field3 "baz"
HEXPIRE hash 10 FIELDS 2 field1 field3
HTTL hash FIELDS 2 field1 field3
HPERSIST hash FIELDS 1 field3
HTTL hash FIELDS 2 field1 field3
```

```redis HFE example 2 cont'd
HGETALL hash
```
### Resources

- A tutorial on building an [AI assistant](redisinsight:_?tutorialId=ai_assistant) using hash field expiration.
- Hash type [reference page](https://redis.io/docs/latest/develop/data-types/hashes/?utm_source=redisinsight&utm_medium=main&utm_campaign=tutorials).
- Entire set of [Redis hash commands](https://redis.io/commands/?group=hash?utm_source=redisinsight&utm_medium=main&utm_campaign=tutorials).
- Check out [Get started with Redis](https://university.redis.io/learningpath/14q8m6gilfwltm?utm_source=redisinsight&utm_medium=main&utm_campaign=tutorials) learning path on Redis University for an introduction to working with all core data structures in Redis.
