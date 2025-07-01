This intelligent AI assistant is designed to support real-world, multi-session use with Redis as its memory core.

What you get:
    - **Smart Memory**: Ephemeral context that expires automatically, long-term facts retained forever
    - **Semantic Search**: Recall relevant info by meaning using vector search
    - **Zero Maintenance**: Auto-expiring short-term memory without a need to track timestamps manually
    - **Multi-User**: Isolated memory per user
    - **Learning**: Assistant learns user preferences and context over time.

**Note**: Requires [Redis 8](https://hub.docker.com/_/redis/tags) for `HSETEX`, which adds per-field TTL for hashes, which is ideal for managing short-term memory with precision.

### Architecture Overview
| Layer | Description |
| ---------- | ---------- |
| `Working Memory`| `Short-term chat context (ephemeral)` |
| `Knowledge Base` | `Long-term memory: persistent facts and preferences` |
| `Vector Search` | `Unified semantic recall across both layers` |

### Working Memory (Ephemeral)
Stores recent user and assistant messages with per-message TTL. Uses:
    - `HSETEX` to add field-level expiration to hashes to store all temporary messages in a single key while managing TTLs per message, simplifying short-term memory management without needing multiple keys.
    - `ZADD` for message ordering (based on timestamp)

```redis:[run_confirmation=true] Recent conversations with TTL based on importance
// Step 1: Add message with TTL (5 mins)
HSETEX user:alice:session:001 EX 300 FIELDS 1 msg:001 "What's the weather?"
// Step 2: Track order with timestamp (epoch seconds)
ZADD user:alice:session:001:zorder 1717935001 msg:001
// Another message (30 min TTL)
HSETEX user:alice:session:001 EX 1800 FIELDS 1 msg:002 "I need a dentist appointment"
ZADD user:alice:session:001:zorder 1717935030 msg:002
// Assistant reply (2 hour TTL)
HSETEX user:alice:session:001 EX 7200 FIELDS 1 msg:003 "Booked for Tuesday 2 PM"
ZADD user:alice:session:001:zorder 1717935090 msg:003
```

```redis:[run_confirmation=true] Reading the session in order
ZRANGEBYSCORE user:alice:session:001:zorder -inf +inf
// Then for each msg ID:
HGET user:alice:session:001 msg:001
HGET user:alice:session:001 msg:002
HGET user:alice:session:001 msg:003
```

```redis:[run_confirmation=true] ZSET cleanup (recommended in background):
// For each msg:XXX in the ZSET, check if it still exists
HEXISTS user:alice:session:001 msg:003
// If not, clean up:
ZREM user:alice:session:001:zorder msg:003
```

### Knowledge Base (Persistent)
Long-term memory: stores important facts, user preferences, and context across sessions. These never expire.
`embedding` is a binary-encoded `FLOAT32[]` used for vector similarity that can be generated using sentence-transformers or similar libraries. Demo uses 8-dim vectors; production models typically use 128–1536 dimensions.

```redis:[run_confirmation=true] Important user information that never expires
// User preferences - need vector fields for search
HSET user:alice:knowledge:pref:001 user_id "alice" memory_type "knowledge" content "prefers mornings before 10 AM" importance 9 timestamp 1717935000 embedding "\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x3f\x40\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00"
HSET user:alice:knowledge:pref:002 user_id "alice" memory_type "knowledge" content "likes detailed explanations" importance 8 timestamp 1717935000 embedding "\x3f\x40\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
// Personal facts
HSET user:alice:knowledge:personal:001 user_id "alice" memory_type "knowledge" content "allergic to shellfish" importance 10 timestamp 1717935000 embedding "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
HSET user:alice:knowledge:personal:002 user_id "alice" memory_type "knowledge" content "golden retriever named Max" importance 7 timestamp 1717935000 embedding "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"
// Work context
HSET user:alice:knowledge:work:001 user_id "alice" memory_type "knowledge" content "Senior PM at TechCorp" importance 8 timestamp 1717935000 embedding "\x40\x40\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"
HSET user:alice:knowledge:work:002 user_id "alice" memory_type "knowledge" content "leading Project Apollo" importance 9 timestamp 1717935000 embedding "\x40\x60\x00\x00\x40\x80\x00\x00\x3f\x40\x00\x00\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x3f\x00\x00\x00"
```

### Vector Search: Semantic Memory Recall
Indexing persistent memory (knowledge base) for semantically meaningful search.

```redis:[run_confirmation=true] Create a vector index
FT.CREATE idx:kbmemory
    ON HASH
    PREFIX 1 user:
    SCHEMA
        user_id TAG
        memory_type TAG
        content TEXT
        importance NUMERIC
        timestamp NUMERIC
        embedding VECTOR HNSW 6
            TYPE FLOAT32
            DIM 8 // DIM = embedding size; 8 used here for simplicity — in production, use 128 to 1536
            DISTANCE_METRIC COSINE // COSINE = measures semantic closeness
```

### Search for most relevant memory entries
Find the top 5 most semantically relevant knowledge memory entries for user "alice" by performing a vector similarity search on the embedding field.

```redis:[run_confirmation=false] Find top 5 related messages by meaning
FT.SEARCH idx:kbmemory 
    "(@user_id:{alice}) => [KNN 5 @embedding $vec AS score]" 
    PARAMS 2 vec "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
    RETURN 4 content memory_type importance score
    SORTBY score ASC
    DIALECT 2
```

### Search for High-Importance Knowledge Items Only
This query finds the top 3 most relevant knowledge memories for user "alice" that have an importance score above 7.

```redis:[run_confirmation=false] Knowledge-only search
FT.SEARCH idx:kbmemory
  "(@user_id:{alice} @memory_type:{knowledge} @importance:[7 +inf]) => [KNN 3 @embedding $vec AS score]"
  PARAMS 2 vec "\x40\x40\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"
  RETURN 4 content importance score timestamp
  SORTBY score ASC
  DIALECT 2
```

### Search Working Memory Excluding Messages Older Than a Certain Timestamp
Retrieve the most similar knowledge memories for user "alice" that were created after a given timestamp (e.g., 1717935000), ensuring you only get recent context:
```redis:[run_confirmation=false] Session-only search
FT.SEARCH idx:kbmemory
  "(@user_id:{alice} @memory_type:{knowledge} @timestamp:[1717935000 +inf]) => [KNN 5 @embedding $vec AS score]"
  PARAMS 2 vec "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"
  RETURN 4 content timestamp memory_type score
  SORTBY score ASC
  DIALECT 2
```

### Monitoring Memory State
Use these queries to inspect what’s stored in memory.

```redis:[run_confirmation=false] Check memory state
// Check active session memory
HGETALL user:alice:knowledge:pref:001  // Example for one preference item

// View user knowledge
HGETALL user:alice:knowledge:pref:001

// Search user's memories
FT.SEARCH idx:kbmemory
    "@user_id:{alice}"
    RETURN 3 content memory_type importance
```

### Data Cleanup

For privacy compliance, delete all user-related keys.

```redis:[run_confirmation=true] Complete user removal
DEL user:alice:knowledge:pref:001
DEL user:alice:knowledge:pref:002
DEL user:alice:knowledge:personal:001
DEL user:alice:knowledge:personal:002
DEL user:alice:knowledge:work:001
DEL user:alice:knowledge:work:002
DEL vmemory:alice:001
DEL vmemory:alice:kb:001
```

### Next Steps
Now that your assistant has memory and meaning, you can:
    - Combine with RAG Pipelines
    - Use sentence-transformers to generate high-dimensional vectors
    - Add [Redis Flex](https://redis.io/solutions/flex/?utm_source=redisinsight&utm_medium=app&utm_campaign=tutorials) for fallback persistence
    - Use Redis ACLs to isolate users, enforce quotas, and monitor usage
