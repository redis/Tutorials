The JSON capability provides JavaScript Object Notation (JSON) support for Redis, which allows Redis to function as a document database.
It lets you store, update, and retrieve JSON values in a Redis database, similar to any other Redis data type. Redis JSON also works seamlessly with Search and Query to let you index and query JSON documents.

Primary features include:

- Full support for the JSON standard.
- [JSONPath](https://goessner.net/articles/JsonPath/) syntax for selecting/updating elements inside documents.
- Documents are stored as binary data in a tree structure, allowing fast access to sub-elements.
- Typed atomic operations for all JSON value types.

### Prerequisites

[Redis 8](https://hub.docker.com/layers/library/redis/8.0.3/images/sha256-426e6823fb1778e8c49f327f9e5af00e505a7fca726ffe11b7930eb1d99ef5fd) or higher \
OR \
[RedisJSON](https://github.com/RedisJSON/RedisJSON/) >=2.6.8 \
OR \
A free Redis instance on [Redis Cloud](https://redis.io/try-free/?utm_source=redisinsight&utm_medium=app&utm_campaign=json_tutorial).
