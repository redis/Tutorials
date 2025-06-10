Build a production-ready AI assistant that remembers conversations semantically and auto-cleans using Redis 8’s new features.

This smart AI assistant offers:
    - **Ephemeral Memory**: Messages expire automatically at different intervals
    - **Semantic Recall**: Search past chats by meaning, not just keywords
    - **Zero Maintenance**: No cleanup scripts or cron jobs needed
    - **Multi-User Support**: Memory isolated by user and session
    - **Hybrid Search**: Combine text and vector search for better results

**Note**: [Redis 8](https://hub.docker.com/_/redis/tags) is required for this tutorial Redis 8 since it introduces `HSETEX`, which lets you set TTL per hash field, ideal for ephemeral chat memory without complex expiration logic.

### Hierarchical Memory Structure
Store chat sessions as Redis hashes. Each message is a field with its own TTL.

```redis:[run_confirmation=true] Upload Session Data
// User-scoped sessions for isolation
// Pattern: user:{user_id}:session:{session_id}
HSETEX user:alice:session:morning msg:1717935301 3600 "Good morning! What's my schedule today?"
HSETEX user:alice:session:morning msg:1717935361 1800 "Remind me about the team meeting at 2 PM"
HSETEX user:alice:session:morning msg:1717935420 900  "What's the weather forecast?"
HSETEX user:alice:session:morning msg:1717935480 300  "Thanks, that's all for now"

// Different user, same session pattern
HSETEX user:bob:session:work msg:1717935500 7200 "I need to prepare for the client presentation"
HSETEX user:bob:session:work msg:1717935560 3600 "What are the key points I should cover?"

```

### Memory Tiers for Different Lifetimes
Control how long messages last depending on their importance.

```redis:[run_confirmation=true] Memory Tiers Strategy
// Short-term (5 minutes) - Immediate context
HSETEX user:alice:session:current msg:1717935301 300 "Current conversation context"

// Medium-term (30 minutes) - Session memory  
HSETEX user:alice:session:current msg:1717935302 1800 "Important session details"

// Long-term (2 hours) - Cross-session context
HSETEX user:alice:session:current msg:1717935303 7200 "Key user preferences and facts"
```

### Check Current Session Memory
No manual cleanup needed; expired messages vanish automatically.

```redis:[run_confirmation=true] Monitor Session State Over Time
// After a few minutes, run this command to see what's left.
HGETALL user:alice:session:morning
```

### Vector Search Setup for Semantic Recall
Create an index to store messages as vectors for semantic search.

```redis:[run_confirmation=true] Create a Vector Index
FT.CREATE idx:ai_memory 
    ON HASH 
    PREFIX 1 memory: 
    SCHEMA 
        user_id TAG SORTABLE
        session_id TAG SORTABLE  
        message TEXT WEIGHT 2.0 PHONETIC dm:en
        context TEXT WEIGHT 1.0
        timestamp NUMERIC SORTABLE
        embedding VECTOR HNSW 6 
            TYPE FLOAT32 
            DIM 8 // DIM = embedding size, DIM 8 is just for demo purposes. In real use, embeddings are usually 128–1536 dimensions.
            DISTANCE_METRIC COSINE // COSINE = measures semantic closeness
            INITIAL_CAP 10000 
            M 16 
            EF_CONSTRUCTION 200
```

Add sample vectorized messages (embedding dims are demo-sized):

```redis:[run_confirmation=true] Add entries for the chatbot
HSET memory:alice:1 user_id "alice" session_id "morning" message "I have a dentist appointment at 3 PM today" context "healthcare scheduling appointment" timestamp 1717935301 embedding "\x00\x00\x80?\x00\x00\x00@\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x00@"
HSET memory:alice:2 user_id "alice" session_id "morning" message "Remind me to water the plants in my office" context "task reminder plants office" timestamp 1717935361 embedding "\x00\x00\x80@\x00\x00\x80@\x00\x00\x80@\x00\x00\x80?\x00\x00\x80?\x00\x00@@"
HSET memory:alice:3 user_id "alice" session_id "work" message "Schedule a meeting with the engineering team" context "work scheduling meeting team" timestamp 1717935420 embedding "\x00\x00@@\x00\x00\x00@\x00\x00\x00@\x00\x00\x00@\x00\x00\x80?\x00\x00\x80?"
HSET memory:bob:1 user_id "bob" session_id "work" message "I need to review the quarterly sales report" context "business analysis quarterly report" timestamp 1717935480 embedding "\x00\x00@@\x00\x00\x00@\x00\x00\x80?\x00\x00\x80@\x00\x00\x00@\x00\x00\x00@"
```

### Let Chatbot Think – Semantic Search with Vectors
When a user says something new, find all related past conversations across your entire system based on semantic meaning.

```redis:[run_confirmation=false] Find Top 5 Related Messages By Meaning
FT.SEARCH idx:ai_memory 
    "*=>[KNN 5 @embedding $query_vec AS vector_score]" 
    PARAMS 2 query_vec "\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x80?\x00\x00@@\x00\x00\x00@"
    RETURN 6 user_id message context vector_score timestamp
    SORTBY vector_score ASC
    DIALECT 2
```

Now your assistant “remembers” things it’s heard before - by meaning.

### User-Scoped Semantic Search
Your AI should only recall memories from the specific user it's talking to, not leak information between users.

```redis:[run_confirmation=false] Find Similar Memories For Specific User Only
FT.SEARCH idx:ai_memory 
    "(@user_id:{alice}) => [KNN 3 @embedding $query_vec AS vector_score]" 
    PARAMS 2 query_vec "\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x80?\x00\x00@@\x00\x00\x00@"
    RETURN 6 user_id message context vector_score session_id
    SORTBY vector_score ASC
    DIALECT 2
```

### Time-Bounded Semantic Search
When users ask about "recent" things, limit your search to a specific time window while still using semantic matching.

```redis:[run_confirmation=false] Find recent similar memories (last 24 hours)
FT.SEARCH idx:ai_memory 
    "(@timestamp:[1717849200 +inf]) => [KNN 3 @embedding $query_vec AS vector_score]" 
    PARAMS 2 query_vec "\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x80?\x00\x00@@\x00\x00\x00@"
    RETURN 6 message timestamp vector_score
    SORTBY timestamp DESC
    DIALECT 2
```

### Session-Specific Recall
When users refer to something from "earlier in our conversation," search only within the current session context.

```
FT.SEARCH idx:ai_memory 
    "(@user_id:{alice} @session_id:{morning}) => [KNN 10 @embedding $query_vec AS vector_score]" 
    PARAMS 2 query_vec "\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x80?\x00\x00@@\x00\x00\x00@"
    RETURN 6 message context vector_score timestamp
    SORTBY vector_score ASC
    DIALECT 2
```

### Want to check what's still in memory?

Only unexpired fields remain.

```redis:[run_confirmation=false] Check Sessions
HGETALL memory:alice:1
HGETALL memory:alice:2
HGETALL memory:alice:3
HGETALL memory:bob:1
HGETALL memory:bob:2
```

### Next Steps
Now that your assistant has memory and meaning, you can:
    - Combine Redis Vector Search with LLMs (RAG)
    - Use OpenAI or sentence-transformers for embeddings
    - Add fallback persistent storage with Redis Flex
    - Manage users with ACLs, quotas, and keyspace notifications