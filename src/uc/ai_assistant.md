This intelligent AI assistant is designed to support real-world, multi-session use with Redis as its memory core.

What you get:
    - **Smart Memory**: Ephemeral context that expires automatically, long-term facts retained forever
    - **Semantic Search**: Recall relevant info by meaning using vector search
    - **Zero Maintenance**: Auto-expiring short-term memory without a need to track timestamps manually
    - **Multi-User**: Isolated memory per user
    - **Learning**: Assistant can "understand" each user better the more it's used

**Note**: Requires [Redis 8](https://hub.docker.com/_/redis/tags) for `HSETEX`, which adds per-field TTL for hashes, which is ideal for managing short-term memory with precision.

### Architecture Overview
| Layer | Description |
| ---------- | ---------- |
| `Working Memory`| `Short-term chat context (ephemeral)` |
| `Knowledge Base` | `Persistent facts, user preferences` |
| `Vector Search` | `Unified semantic recall across both layers` |

### Working Memory (Ephemeral)
Stores recent user messages with TTL based on importance. Automatically expires to prevent bloat.
This uses `HSETEX`, a Redis 8 command that adds field-level expiration to hashes. It allows storing all temporary messages in a single key while managing TTLs per message, simplifying short-term memory management without needing multiple keys.

```redis:[run_confirmation=true] Recent Conversations with TTL Based on Importance.
// Quick exchanges (5 min)
HSETEX user:alice:session:001 EX 300 FIELDS 1 msg:001 "What's the weather?"
// Session context (30 min)  
HSETEX user:alice:session:001 EX 1800 FIELDS 1 msg:002 "I need a dentist appointment"
// Important decisions (2 hours)
HSETEX user:alice:session:001 EX 7200 FIELDS 1 msg:003 "Book it for Tuesday 2 PM"
```

### Knowledge Base (Persistent)
Long-term memory: stores important facts, user preferences, and context across sessions. These never expire.
`embedding` is a binary-encoded `FLOAT32[]` used for vector similarity that can be generated using sentence-transformers or similar libraries. Demo uses 8-dim vectors; production models typically use 128–1536 dimensions.

```redis:[run_confirmation=true] Important User Information That Never Expires.
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
Unify working + knowledge memory into a vector index for semantically meaningful search.

```redis:[run_confirmation=true] Create a Vector Index
FT.CREATE idx:memory
    ON HASH
    PREFIX 2 vmemory: user:
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

### Add Indexed Memory
Populate the vector index with memory items from both ephemeral and persistent layers.

```redis:[run_confirmation=true] Add entries for the chatbot
// Working memory (expires from Redis, stays in search until rebuild)
HSET vmemory:alice:001 user_id "alice" memory_type "working" content "Book dentist for Tuesday 2 PM" importance 8 timestamp 1717935310 embedding "\x3f\x80\x00\x00\x40\x00\x00\x00\x40\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00\x40\x20\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00"

// Knowledge base (persistent)
HSET vmemory:alice:kb:001 user_id "alice" memory_type "knowledge" content "allergic to shellfish" importance 10 timestamp 1717935000 embedding "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
```

### Full Memory Search
Search across all memories to recall relevant data.

```redis:[run_confirmation=false] Find Top 5 Related Messages By Meaning
FT.SEARCH idx:memory 
    "(@user_id:{alice}) => [KNN 5 @embedding $vec AS score]" 
    PARAMS 2 vec "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
    RETURN 4 content memory_type importance score
    SORTBY score ASC
    DIALECT 2
```

### Session-Only Search
Limit results to current conversation context (ephemeral memory).

```redis:[run_confirmation=false] Session-Only Search
FT.SEARCH idx:memory 
    "(@user_id:{alice} @memory_type:{working}) => [KNN 5 @embedding $vec AS score]" 
    PARAMS 2 vec "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"
    RETURN 4 content score timestamp memory_type
    SORTBY score ASC
    DIALECT 2
```

### Knowledge-Only Search
Focus only on persistent memory: facts, preferences, decisions.

```redis:[run_confirmation=false] Knowledge-Only Search
FT.SEARCH idx:memory 
    "(@user_id:{alice} @memory_type:{knowledge}) => [KNN 8 @embedding $vec AS score]" 
    PARAMS 2 vec "\x40\x40\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"
    RETURN 4 content importance score timestamp
    SORTBY score ASC
    DIALECT 2
```

### Monitoring Memory State
Use these queries to inspect what’s stored in memory.

```redis:[run_confirmation=false] Check Memory State
// Check active session memory
HGETALL user:alice:knowledge:pref:001  // Example for one preference item

// View user knowledge
HGETALL user:alice:knowledge:pref:001

// Search user's memories
FT.SEARCH idx:memory
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