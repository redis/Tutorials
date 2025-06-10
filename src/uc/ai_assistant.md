Imagine you are building a smart AI assistant that:
    - Remembers chats, but only temporarily
    - Thinks by meaning, not by matching exact words
    - Cleans up after itself, with no manual scripts

Redis 8 has two powerful capabilities to make this happen:
    - Field-level expiration: Let individual chat messages expire on their own
    - Vector similarity search: Find past messages based on meaning, not keywords

Let’s dive in.

### Short-Term Memory with Field-Level Expiry
Each chat session is stored as a Redis Hash.
Each message is a field in the hash.
Redis 8’s new HSETEX command allows you to assign a TTL to each field, perfect for building ephemeral, session-level memory.

```redis:[run_confirmation=true] Upload Session Data
// session:42 is the session ID
// msg:<timestamp> ensures uniqueness and traceability
HSETEX session:42 msg:1717935301 120 "Hi Chatbot!"
HSETEX session:42 msg:1717935361 180 "What can you do?"
HSETEX session:42 msg:1717935440 90 "Can you remind me about my tasks?"
HSETEX session:42 msg:1717935720 30  "What's the news today?"
```

Each field automatically expires after its TTL (in seconds).
No need for cron jobs or background workers.
What you get:
    - Clean memory
    - Zero manual cleanup
    - Session-scoped retention, just like short-term memory in humans


Try it: After a few minutes, run `HGETALL session:42` and see what's left.

### Vector Search for Semantic Recall
Now, your assistant needs to “recall” semantically related messages, not just match by words.
To do that, you’ll:
    - Convert messages to vector embeddings
    - Store them in Redis
    - Use Vector Search with FT.SEARCH for semantic retrieval

```redis:[run_confirmation=true] Create a Vector Index
FT.CREATE idx:memory ON HASH PREFIX 1 memory: SCHEMA
    message TEXT
    embedding VECTOR FLAT // FLAT = exact vector search
    6
        TYPE FLOAT32
        DIM 8 // DIM = embedding size, DIM 8 is just for demo purposes. In real use, embeddings are usually 128–1536 dimensions.
        DISTANCE_METRIC COSINE // COSINE = measures semantic closeness
```

Now, let’s add entries for your chatbots:

```redis:[run_confirmation=true] Add entries for the chatbot
// Embeddings are stored as binary FLOAT32 vectors - this is a compact format required by Redis Vector Serch indexes
HSET memory:1 message "Book a dentist appointment"       embedding "\x00\x00\x80?\x00\x00\x00@\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x00@"
HSET memory:2 message "Remind me to water plants"        embedding "\x00\x00\x80@\x00\x00\x80@\x00\x00\x80@\x00\x00\x80?\x00\x00\x80?\x00\x00@@"
HSET memory:3 message "What’s the weather like?"         embedding "\x00\x00@@\x00\x00\x00@\x00\x00\x00@\x00\x00\x00@\x00\x00\x80?\x00\x00\x80?"
HSET memory:4 message "Cancel my gym session"            embedding "\x00\x00@@\x00\x00\x00@\x00\x00\x80?\x00\x00\x80@\x00\x00\x00@\x00\x00\x00@"
HSET memory:5 message "Start a new shopping list"        embedding "\x00\x00\x00@\x00\x00\x00@\x00\x00\x80?\x00\x00\x80@\x00\x00\x80?\x00\x00@@"
```

Now your messages are vectorized and ready for search.

### Let Chatbot Think – Semantic Search with Vectors
When a user sends a new message, convert it to an embedding and run a KNN search:

```redis:[run_confirmation=true] Search For Similar Messages
// Returns the top 3 semantically similar messages, even if no words match directly.
FT.SEARCH idx:memory "*=>[KNN 3 @embedding $vec AS score]"
   PARAMS 2 vec "\x00\x00@@\x00\x00\x80@\x00\x00\x00@\x00\x00\x80?\x00\x00@@\x00\x00\x00@"
   SORTBY score
   DIALECT 2
```

Now your assistant “remembers” things it’s heard before - by meaning.

### Real-Time Session Cleanup – Redis Handles It
Want to check what's still in memory?

```redis:[run_confirmation=false] Check Sessions
HGETALL session:42
```

Only the unexpired fields remain. Redis does the cleanup invisibly in the background.
Your assistant has a clean, focused mind at all times.

### Next Steps
Now that your assistant has memory and meaning, you can:
    - Tie session messages to store embeddings for per-session recall
    - Use RAG (Retrieval-Augmented Generation) by combining Redis Vector Search with LLMs
    - Add per-user memory: prefix session keys with a user ID (user:42:session:...)
    - Introduce a fallback to persistent storage for long-term memory using Redis Flex