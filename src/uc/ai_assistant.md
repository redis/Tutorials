This tutorial demonstrates how to build an AI assistant's memory system with Redis as its memory core.

**Note**: Requires [Redis 8](https://hub.docker.com/_/redis/tags) for `HSETEX`, which adds per-field TTL for hashes, which is ideal for managing short-term memory with precision.

### Architecture Overview
| Layer | Description | Data type |
| ---------- | ---------- | ---------- |
| `Session History`| `Recent conversation context` | List |
| `Rate Limiting` | `Per-user request throttling` | Hash |
| `Knowledge Base` | `Long-term facts and preferences` | Hash |

### Session History
AI assistants need context from previous messages to provide coherent responses. Without conversation history, each interaction would be isolated and the AI couldn't reference what was discussed earlier. We can store conversation history using Redis lists - simple, ordered, and efficient for chat scenarios.

```redis:[run_confirmation=true] Store conversation history
// Add user message to session
LPUSH user:alice:history:session_001 '{"type": "human", "content": "What's the weather like?", "timestamp": 1717935001}'

// Add AI response
LPUSH user:alice:history:session_001 '{"type": "ai", "content": "It's sunny with 75°F temperature.", "timestamp": 1717935002}'

// Add another user message
LPUSH user:alice:history:session_001 '{"type": "human", "content": "Should I bring an umbrella?", "timestamp": 1717935003}'

// Add AI response
LPUSH user:alice:history:session_001 '{"type": "ai", "content": "No umbrella needed today!", "timestamp": 1717935004}'
```
### Reading Conversation History
Now we can retrieve conversation history to provide context to the AI.

```redis:[run_confirmation=true] Read conversation history
// Get last 5 messages (most recent first)
LRANGE user:alice:history:session_001 0 4

// Get all messages in session
LRANGE user:alice:history:session_001 0 -1

// Get specific message by index
LINDEX user:alice:history:session_001 0

// Check how many messages in session
LLEN user:alice:history:session_001
```
### Session Expiration
Without expiration, conversation history would accumulate indefinitely, consuming memory and potentially exposing sensitive information. TTL ensures privacy and efficient memory usage.

```redis:[run_confirmation=true] Session expiration
// Set session to expire in 24 hours
EXPIRE user:alice:history:session_001 86400

// Set session to expire in 1 hour
EXPIRE user:alice:history:session_001 3600

// Check remaining TTL
TTL user:alice:history:session_001

// Remove expiration (make persistent)
PERSIST user:alice:history:session_001
```

### Rate Limiting
Rate limiting prevents abuse and ensures fair resource usage. Without it, users could overwhelm the system with requests, degrading performance for everyone.

```redis:[run_confirmation=true] Initialize Rate Limiting
// First request - set counter with 1-minute TTL
HSETEX user:alice:rate_limit EX 60 FIELDS 1 requests_per_minute 1

// Check current request count
HGET user:alice:rate_limit requests_per_minute
```

The `HINCR` command allows you to atomically increment the counter, preventing race conditions in high-concurrency scenarios.

```redis:[run_confirmation=true] Increment Requests
// Increment request counter
HINCR user:alice:rate_limit requests_per_minute

// Check if field exists and get count
HEXISTS user:alice:rate_limit requests_per_minute
HGET user:alice:rate_limit requests_per_minute

// Check TTL on the hash
TTL user:alice:rate_limit
```

Different time windows serve different purposes - per-minute prevents burst attacks, per-hour prevents sustained abuse, per-day enforces usage quotas.
```redis:[run_confirmation=true] Rate Limiting with Different Time Windows
// Set multiple rate limits with different TTLs
HSETEX user:alice:rate_limit EX 60 FIELDS 2 requests_per_minute 1 requests_per_hour 1

// Daily rate limit (24 hours)
HSETEX user:alice:rate_limit EX 86400 FIELDS 1 requests_per_day 1

// Check all rate limits
HGETALL user:alice:rate_limit
```

### Knowledge Base (Persistent Memory)
AI assistants become more helpful when they remember user preferences, important facts, and context across sessions. This creates a personalized experience that improves over time.
Remembering user preferences (meeting times, communication style) enables the AI to provide more relevant and personalized responses without asking the same questions repeatedly.

```redis:[run_confirmation=true] Store User Preferences
// Always secure sensitive data using encryption at rest, access control (Redis ACLs), and comply with data protection laws (e.g., GDPR).
// Store morning preference
HSET user:alice:knowledge:pref:001 user_id "alice" content "prefers morning appointments before 10 AM" importance 9 timestamp 1717935000 embedding "\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x3f\x40\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00"

// Storing communication preference
HSET user:alice:knowledge:pref:002 user_id "alice" content "likes detailed explanations with examples" importance 8 timestamp 1717935000 embedding "\x3f\x40\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"

// Store work schedule preference
HSET user:alice:knowledge:pref:003 user_id "alice" content "works remotely on Fridays" importance 7 timestamp 1717935000 embedding "\x40\x80\x00\x00\x3f\x00\x00\x00\x40\x40\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"

// Store health information
HSET user:alice:knowledge:personal:001 user_id "alice" content "allergic to shellfish and nuts" importance 10 timestamp 1717935000 embedding "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"

// Store pet information
HSET user:alice:knowledge:personal:002 user_id "alice" content "has a golden retriever named Max, 3 years old" importance 7 timestamp 1717935000 embedding "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"

// Store family information
HSET user:alice:knowledge:personal:003 user_id "alice" content "married to Bob, two kids Sarah (8) and Tom (5)" importance 9 timestamp 1717935000 embedding "\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x3f\x00\x00\x00"
```

### Vector Search: Semantic Memory Recall
Traditional keyword search misses semantic meaning. When a user asks about "scheduling meetings," vector search can find relevant information about "prefers morning appointments" even though the keywords don't match exactly.

Indexing persistent memory (knowledge base) for semantically meaningful search.

```redis:[run_confirmation=true] Create a Vector Index
// Create index for semantic search
FT.CREATE idx:knowledge
    ON HASH
    PREFIX 1 user:
    SCHEMA
        user_id TAG
        content TEXT
        importance NUMERIC
        timestamp NUMERIC
        embedding VECTOR HNSW 6
            TYPE FLOAT32
            DIM 8 // DIM = embedding size, DIM 8 is just for demo purposes. In real use, embeddings are usually 128–1536 dimensions.
            DISTANCE_METRIC COSINE
```

### Search for most relevant memory entries

```redis:[run_confirmation=false] Find Top 5 Most Relevant Knowledge Items
FT.SEARCH idx:knowledge 
    "(@user_id:{alice}) => [KNN 5 @embedding $vec AS score]" 
    PARAMS 2 vec "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
    RETURN 4 content importance score timestamp
    SORTBY score ASC
    DIALECT 2
```

```redis:[run_confirmation=false] Search High-Importance Items Only
FT.SEARCH idx:knowledge
    "(@user_id:{alice} @importance:[8 +inf]) => [KNN 3 @embedding $vec AS score]"
    PARAMS 2 vec "\x40\x40\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"
    RETURN 4 content importance score timestamp
    SORTBY score ASC
    DIALECT 2
```
```redis:[run_confirmation=false] Search Recent Knowledge Only
FT.SEARCH idx:knowledge
    "(@user_id:{alice} @timestamp:[1717935000 +inf]) => [KNN 5 @embedding $vec AS score]"
    PARAMS 2 vec "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"
    RETURN 4 content timestamp score
    SORTBY score ASC
    DIALECT 2
```

### Memory State Monitoring
Understanding what's stored in memory helps debug issues, optimize performance, and ensure data quality. It's also essential for user privacy compliance.
```redis:[run_confirmation=false] Monitor user sessions
// Scan 10,000 keys to find user sessions
SCAN 0 MATCH user:*:history:* COUNT 10000

// Get session statistics
LLEN user:alice:history:session_001
TTL user:alice:history:session_001
```
### Data Cleanup
```redis:[run_confirmation=true] Delete user data
// Remove all user data (GDPR compliance)
DEL user:alice:history:session_001
DEL user:alice:history:session_002
DEL user:alice:rate_limit
DEL user:alice:knowledge:pref:001
DEL user:alice:knowledge:pref:002
DEL user:alice:knowledge:pref:003
DEL user:alice:knowledge:personal:001
DEL user:alice:knowledge:personal:002
DEL user:alice:knowledge:personal:003
DEL user:alice:knowledge:work:001
DEL user:alice:knowledge:work:002
DEL user:alice:knowledge:work:003
```

### Next Steps
Now that your assistant has memory and meaning, you can:
    - Combine with RAG Pipelines
    - Use sentence-transformers to generate high-dimensional vectors
    - Add [Redis Flex](https://redis.io/solutions/flex/?utm_source=redisinsight&utm_medium=app&utm_campaign=tutorials) for fallback persistence
    - Use Redis ACLs to isolate users, enforce quotas, and monitor usage
