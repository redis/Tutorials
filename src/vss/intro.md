## Vector search

Data is often unstructured, which means that it isn't described by a well-defined schema.

Examples of unstructured data include:
- Text passages
- Images
- Videos
- Music titles

One approach to dealing with unstructured data is to vectorize it. Vectorizing means mapping unstructured data to a flat sequence of numbers.

Vectors represent the data embedded in an N-dimensional space.

Machine learning models have facilitated the rise of embeddings as a widely embraced method for generating dense, low-dimensional vector representations.

Given a suitable machine learning model, the generated embeddings can encapsulate complex patterns and semantic meanings inherent in data.

You can use Redis 8 as a vector database, which allows you to:

- Store vectors and the associated metadata within hashes or JSON documents
- Retrieve vectors
- Perform vector searches


### Prerequisites

[Redis 8](https://hub.docker.com/layers/library/redis/8.0.3/images/sha256-426e6823fb1778e8c49f327f9e5af00e505a7fca726ffe11b7930eb1d99ef5fd) or higher \
OR \
[Redis Query Engine](https://redis.io/docs/latest/develop/ai/search-and-query/?utm_source=redisinsight&utm_medium=app&utm_campaign=vector_search_tutorial) >=2.8.11 \
OR \
A free Redis instance on [Redis Cloud](https://redis.io/try-free/?utm_source=redisinsight&utm_medium=app&utm_campaign=vector_search_tutorial).
