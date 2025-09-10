The following JSON data model will be used in this tutorial. The data represents user preferences that a hypothetical bike shop might use to target ads to its customers.

```
{
  "user": "morti",
  "descr": "Morti is into kid's and road bikes.",
  "labels": "kids, road",
  "vector_embedding": [
    0.1,
    0.9,
    0.7
  ]
}
```

The `vector_embedding` element is a vector, a list of numbers, that represents preferences for certain kinds of bikes.
Each three-element list represents a preference score for:

- mountain bikes
- kid's bikes
- road bikes

**Note:**
> This is just a trivial example that was created as an illustration. Vector embeddings for real projects can be huge. For a more realistic example, see this [vector search quick start guide](https://redis.io/docs/get-started/vector-database/).

To conduct vector searches, you must first create an index and then load your data.

```redis Create an index
FT.CREATE idx:user_prefs                      // index name
  ON JSON                                     // the type of data to be indexed
    PREFIX 1 user:                            // identifies the keys to be indexed
  SCHEMA 
    $.descr TEXT                              // Allows full-text search queries
    $.labels TAG SEPARATOR ","                // Allows exact-match queries, such as categories or primary keys
    $.vector_embedding as vector VECTOR HNSW  // Hierarchical navigable small worl vector
      6                                       // Six parameters
      TYPE FLOAT32                            // Each vector element is a 32-bit, floating point number
      DIM 3                                   // Each vector has three dimensions
      DISTANCE_METRIC COSINE                  // See the docs for a description
```

This command will create an index called `idx:user_prefs` on all JSON data with keys prefixed by `user:`.
The index schema has the following attributes:

- `$.descr TEXT` - the key's `descr` field, indexed as `TEXT`
- `$.labels TAG SEPARATOR ","` - the key's `labels` field, indexed as `TAG` using comma as a separator
- a vector attribute

This vector index attribute is broken down as follows:

- `$.vector_embedding as vector` - `vector` is an alias for `$.vector_embedding`
- `VECTOR` is the type of attribute
- `HSNW` and its six arguments defined the algorithm that will be used, Hierarchical Navigable Small World.
    - The vector `TYPE` is `FLOAT32`; each element of the given vector is a 32-bit, floating point number.
    - `DIM 3` means that each vector is three dimensional.
    - The `DISTANCE_METRIC` is defined as `COSINE`. Other possible values are `IP` and `L2`.

See the [vector reference](https://redis.io/docs/latest/develop/ai/search-and-query/vectors/?utm_source=redisinsight&utm_medium=main&utm_campaign=tutorials) for more detailed information about each available option.

```redis Load some data
JSON.SET user:1 $ '{"user": "samuel", "descr": "Samuel likes mountain and kid\'s bikes.", "labels": "mountain, kids", "vector_embedding": [0.9, 0.7, 0.2]}'
JSON.SET user:2 $ '{"user": "david", "descr": "David likes mountain and kid\'s bikes.", "labels": "mountain, kids", "vector_embedding": [0.7, 0.9, 0.1]}'
JSON.SET user:3 $ '{"user": "pieter", "descr": "Pieter likes kid\'s bikes.", "labels": "kids", "vector_embedding": [0.3, 0.9, 0.2]}'
JSON.SET user:4 $ '{"user": "morti", "descr": "Morti is into kid\'s and road bikes.", "labels": "kids, road", "vector_embedding": [0.1, 0.9, 0.7]}'
```

Suppose you want to search for users with preferences similar to the vector `[0.9, 0.7, 0.2]`. Before searching, you'll need to convert the given vector, the JSON array `[0.9, 0.7, 0.2]`, into a flat sequence of bytes. Python's NumPy library can do this:

```python
import numpy as np

vector_embedding = [0.9, 0.7, 0.2]
input_vector = np.array(data['vector_embedding']).astype(np.float32).tobytes()
print(input_vector)
b'fff?333?\xcd\xccL>'
```

Now you can perform your search using the byte string you obtained from Python as the third to last parameter.

```redis Search
FT.SEARCH idx:user_prefs "(*)=>[KNN 2 @vector $input_vector]" PARAMS 2 input_vector "fff?333?\xcd\xccL>" DIALECT 2
```
