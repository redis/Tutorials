This tutorial demonstrates how to build an AI assistant's memory system with Redis as its memory core.

**Note**: Requires [Redis 8](https://hub.docker.com/_/redis/tags) for `HSETEX`, which adds per-field TTL for hashes - ideal for rate limiting to ensure fair resource usage.

### Architecture overview
| Layer | Description | Data type |
| ---------- | ---------- | ---------- |
| `Session History`| `Recent conversation context` | List |
| `Rate Limiting` | `Per-user request throttling` | Hash |
| `User Memory` | `Long-term facts and preferences` | Hash |

### Session history
AI assistants need context from previous messages to provide coherent responses. Without conversation history, each interaction would be isolated. Redis lists are simple, ordered, and efficient for storing chat transcripts.

```redis:[run_confirmation=true] Store conversation history
// Add user message to session
LPUSH user:alice:history:session_001 '{"type": "human", "content": "What is the weather like?", "timestamp": 1717935001}'

// Add AI response
LPUSH user:alice:history:session_001 '{"type": "ai", "content": "It is sunny with 75°F temperature.", "timestamp": 1717935002}'

// Add another user message
LPUSH user:alice:history:session_001 '{"type": "human", "content": "Should I bring an umbrella?", "timestamp": 1717935003}'

// Add AI response
LPUSH user:alice:history:session_001 '{"type": "ai", "content": "No umbrella needed today!", "timestamp": 1717935004}'
```
### Reading conversation history
You can retrieve recent messages to provide context to the AI.

```redis:[run_confirmation=no] Read conversation history
// Get the 5 most recent messages
LRANGE user:alice:history:session_001 0 4

// Get the full session
LRANGE user:alice:history:session_001 0 -1
```
You may want to limit the size of history to retain only the N most recent items. Use LTRIM:
```redis:[run_confirmation=yes] Read conversation history
LTRIM user:alice:history:session_001 0 29  // keep only latest 30 items
```

### Session expiration
Without expiration, session history will accumulate indefinitely. Expiring keys improves memory usage and ensures privacy.

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

### Rate limiting
Rate limiting prevents abuse and ensures fair usage across users. Redis hashes with field-level TTL via `HSETEX` are ideal for this.

```redis:[run_confirmation=true] Initialize Rate Limiting
// On first request - set counter with 1-minute TTL
HSETEX user:alice:rate_limit EX 60 FIELDS 1 requests_per_minute 1
```

The `HINCR` command allows you to atomically increment the counter, preventing race conditions in high-concurrency scenarios.

```redis:[run_confirmation=true] Increment Requests
// Increment request counter
HINCRBY user:alice:rate_limit requests_per_minute 1

// Check if field exists and get count
HEXISTS user:alice:rate_limit requests_per_minute
HGET user:alice:rate_limit requests_per_minute

// Check TTL on the hash
TTL user:alice:rate_limit
```
**Optionally**: if the count exceeds the allowed threshold, deny the operation.

Different time windows serve different purposes - per-minute prevents burst attacks, per-hour prevents sustained abuse, per-day enforces usage quotas.
```redis:[run_confirmation=true] Rate Limiting with Different Time Windows
// Set multiple rate limits with different TTLs
HSETEX user:alice:rate_limit EX 60 FIELDS 2 requests_per_minute 1 requests_per_hour 1

// Daily rate limit (24 hours)
HSETEX user:alice:rate_limit EX 86400 FIELDS 1 requests_per_day 1

// Check all rate limits
HGETALL user:alice:rate_limit
```

### User memory (persistent preferences)
AI assistants become more helpful when they remember user preferences, schedules, or relevant facts. This persistent memory enables personalization over time.

```redis:[run_confirmation=true] Store User Preferences
// Always secure sensitive data using encryption at rest, access control (Redis ACLs), and comply with data protection laws (e.g., GDPR).
// These values are stored with embeddings to support semantic recall later using vector search.
HSET user:alice:pref:001 user_id "alice" content "prefers morning appointments before 10 AM" importance 9 timestamp 1717935000 embedding "\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x3f\x40\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00"

// Storing communication preference
HSET user:alice:pref:002 user_id "alice" content "likes detailed explanations with examples" importance 8 timestamp 1717935000 embedding "\x3f\x40\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"

// Store work schedule preference
HSET user:alice:pref:003 user_id "alice" content "works remotely on Fridays" importance 7 timestamp 1717935000 embedding "\x40\x80\x00\x00\x3f\x00\x00\x00\x40\x40\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"

// Store health information
HSET user:alice:personal:001 user_id "alice" content "allergic to shellfish and nuts" importance 10 timestamp 1717935000 embedding "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"

// Store pet information
HSET user:alice:personal:002 user_id "alice" content "has a golden retriever named Max, 3 years old" importance 7 timestamp 1717935000 embedding "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"

// Store family information
HSET user:alice:personal:003 user_id "alice" content "married to Bob, two kids Sarah (8) and Tom (5)" importance 9 timestamp 1717935000 embedding "\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x3f\x00\x00\x00"
```

### Vector search: semantic memory recall
Semantic search allows AI to retrieve relevant memory even when exact keywords don't match. For example, a query about "meetings" might return facts about "morning appointments."

Indexing persistent memory (User Memory) for semantically meaningful search.

```redis:[run_confirmation=true] Create a Vector Index
// Create index for semantic search
FT.CREATE idx:preferences
    ON HASH
    PREFIX 1 user:
    SCHEMA
        user_id TAG
        content TEXT
        importance NUMERIC
        timestamp NUMERIC
        embedding VECTOR HNSW 6
            TYPE FLOAT32
            DIM 8 // DIM 8 is only for demonstration purposes. Real embeddings are typically 128–1536 dimensions depending on the model (e.g., sentence-transformers).
            DISTANCE_METRIC COSINE
```

### Search for most relevant memory entries

```redis:[run_confirmation=false] Find Top 5 Most Relevant User Memory Items
FT.SEARCH idx:preferences 
    "(@user_id:{alice}) => [KNN 5 @embedding $vec AS score]" 
    PARAMS 2 vec "\x40\x00\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x60\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x3f\x00\x00\x00"
    RETURN 4 content importance score timestamp
    SORTBY score ASC
    DIALECT 2
```

```redis:[run_confirmation=false] Search High-Importance Items Only
FT.SEARCH idx:preferences
    "(@user_id:{alice} @importance:[8 +inf]) => [KNN 3 @embedding $vec AS score]"
    PARAMS 2 vec "\x40\x40\x00\x00\x3f\x00\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x80\x00\x00\x40\x60\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00"
    RETURN 4 content importance score timestamp
    SORTBY score ASC
    DIALECT 2
```
```redis:[run_confirmation=false] Search Recent User Memories
FT.SEARCH idx:preferences
    "(@user_id:{alice} @timestamp:[1717935000 +inf]) => [KNN 5 @embedding $vec AS score]"
    PARAMS 2 vec "\x3f\x80\x00\x00\x40\x40\x00\x00\x40\x00\x00\x00\x3f\x40\x00\x00\x40\x80\x00\x00\x40\x20\x00\x00\x3f\x00\x00\x00\x40\x60\x00\x00"
    RETURN 3 content timestamp score
    SORTBY score ASC
    DIALECT 2
```

### Memory state monitoring
Understanding what's stored in memory helps debug issues, optimize performance, and ensure data quality. It's also essential for user privacy compliance.
```redis:[run_confirmation=false] Monitor user sessions
// Get approximate memory usage of session
MEMORY USAGE user:alice:history:session_001

// Get session statistics
LLEN user:alice:history:session_001
TTL user:alice:history:session_001
```
### Data cleanup
Remove all data related to a user (e.g., for GDPR compliance).

```redis:[run_confirmation=true] Delete user data
// Remove all user data (GDPR compliance)
DEL user:alice:history:session_001
DEL user:alice:history:session_002
DEL user:alice:rate_limit
DEL user:alice:pref:001
DEL user:alice:pref:002
DEL user:alice:pref:003
DEL user:alice:personal:001
DEL user:alice:personal:002
DEL user:alice:personal:003
DEL user:alice:work:001
DEL user:alice:work:002
DEL user:alice:work:003
```

### Next steps
Now that your assistant has memory and meaning, you can:
    - Combine with RAG Pipelines
    - Use sentence-transformers to generate high-dimensional vectors
    - Add [Redis Flex](https://redis.io/solutions/flex/?utm_source=redisinsight&utm_medium=app&utm_campaign=tutorials) for fallback persistence
    - Use Redis ACLs to isolate users, enforce quotas, and monitor usage
