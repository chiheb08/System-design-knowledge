# System Design Fundamentals: Communication Patterns

> A beginner-friendly guide to understanding core communication concepts in system design.

---

## Table of Contents
1. [The Pipe (Pipeline Pattern)](#1-the-pipe-pipeline-pattern)
2. [Polling](#2-polling)
3. [Bidirectional WebSocket](#3-bidirectional-websocket)
4. [Throttling](#4-throttling)
5. [Quick Comparison](#quick-comparison-table)

---

## 1. The Pipe (Pipeline Pattern)

### What is it?
A **pipe** is like a water pipe in your house - data flows through it from one place to another. In computing, it's a way to connect the output of one process directly to the input of another.

### Real-World Analogy
Think of a factory assembly line:
- Station 1 makes the car frame -> passes it to Station 2
- Station 2 adds the engine -> passes it to Station 3
- Station 3 paints the car -> passes it to Station 4

Each station does ONE job and passes the result to the next.

### In Code (Example)
```bash
# Unix/Linux pipe example
cat file.txt | grep "error" | sort | uniq
```

Here's what happens step by step:
1. `cat file.txt` - reads the file (outputs text)
2. `|` - the PIPE sends that output to...
3. `grep "error"` - filters only lines with "error"
4. `|` - sends those filtered lines to...
5. `sort` - sorts them alphabetically
6. `|` - sends sorted lines to...
7. `uniq` - removes duplicates

### When to Use Pipes
- Processing data in stages
- When each step is independent
- Streaming data transformations
- Building modular, reusable components

### Key Benefits
- **Modularity**: Each stage does one thing well
- **Reusability**: Stages can be reused in different pipelines
- **Testability**: Each stage can be tested independently

---

## 2. Polling

### What is it?
**Polling** is when a client repeatedly asks a server "Do you have new data for me?" at regular intervals. It's like constantly refreshing your email inbox.

### Real-World Analogy
Imagine you're waiting for a package:
- **Polling approach**: You call the delivery company every 5 minutes asking "Is my package here yet?"
- You keep calling until they say "Yes, it arrived!"

### How It Works
```
Client                          Server
  |                               |
  |---"Any new messages?"-------->|
  |<--------"No, nothing"---------|
  |                               |
  |   (wait 5 seconds...)         |
  |                               |
  |---"Any new messages?"-------->|
  |<--------"No, nothing"---------|
  |                               |
  |   (wait 5 seconds...)         |
  |                               |
  |---"Any new messages?"-------->|
  |<--"Yes! Here's a message"-----|
```

### Types of Polling

| Type | Description | Pros | Cons |
|------|-------------|------|------|
| **Short Polling** | Ask frequently (every few seconds), get immediate response | Simple, predictable | Wastes bandwidth, delayed updates |
| **Long Polling** | Ask once, server holds the connection until it has data | More efficient, faster updates | More complex, connection timeouts |

### Short Polling Example
```python
import time
import requests

while True:
    response = requests.get("https://api.example.com/messages")
    if response.json().get("new_messages"):
        print("New message received!")
        break
    time.sleep(5)  # Wait 5 seconds before asking again
```

### Long Polling Example
```python
import requests

# Server holds this request until there's new data
# or timeout occurs (e.g., 30 seconds)
response = requests.get(
    "https://api.example.com/messages",
    params={"wait": True, "timeout": 30}
)
```

### Pros & Cons of Polling
**Pros:**
- Simple to implement
- Works everywhere (no special protocol needed)
- Compatible with all browsers and firewalls

**Cons:**
- Wastes resources (many unnecessary requests when there's no new data)
- Delays in getting updates (depends on polling interval)
- Not truly real-time

---

## 3. Bidirectional WebSocket

### What is it?
A **WebSocket** is a persistent, two-way communication channel between client and server. **Bidirectional** means BOTH sides can send messages anytime - no need to wait for the other to ask.

### Real-World Analogy
Think of a **phone call**:
- Once connected, BOTH people can talk whenever they want
- You don't need to hang up and call again to send a new message
- The connection stays open until someone hangs up

Compare this to **sending letters** (like traditional HTTP):
- You send a letter, wait for a response
- Want to say something else? Send another letter
- Very slow and inefficient for conversations

### How It Works
```
Client                          Server
  |                               |
  |===== Connection Opened =======|  (initial handshake)
  |                               |
  |---"Hello!"------------------>|  (client sends message)
  |<--"Hi there!"----------------|  (server responds)
  |<--"Breaking news: ..."-------|  (server can push anytime!)
  |---"Thanks!"----------------->|  (client responds)
  |<--"Score update: 2-1"--------|  (server pushes again)
  |                               |
  |===== Connection Stays Open ===|
```

### WebSocket vs HTTP

| Feature | HTTP | WebSocket |
|---------|------|-----------|
| Connection | New connection per request | Persistent connection |
| Direction | Client asks, server responds | Both can send anytime |
| Overhead | Headers sent every time | Minimal after handshake |
| Real-time | No (need polling) | Yes |

### JavaScript WebSocket Example
```javascript
// Client-side WebSocket
const socket = new WebSocket('wss://example.com/chat');

// Connection opened
socket.onopen = function(event) {
    console.log('Connected to server!');
    socket.send('Hello Server!');
};

// Listen for messages from server
socket.onmessage = function(event) {
    console.log('Message from server:', event.data);
};

// Handle errors
socket.onerror = function(error) {
    console.log('WebSocket Error:', error);
};

// Connection closed
socket.onclose = function(event) {
    console.log('Disconnected from server');
};
```

### Use Cases
- **Chat applications** (WhatsApp Web, Slack, Discord)
- **Live sports scores and updates**
- **Stock market / cryptocurrency price tickers**
- **Multiplayer online games**
- **Collaborative tools** (Google Docs, Figma)
- **Live notifications**

### Pros & Cons
**Pros:**
- True real-time communication
- Efficient (no repeated handshakes or headers)
- Low latency
- Server can push data without client asking

**Cons:**
- More complex to implement
- Requires persistent connections (uses server memory)
- May have issues with some firewalls/proxies
- Need to handle reconnection logic

---

## 4. Throttling

### What is it?
**Throttling** is limiting how often something can happen within a time period. It's a protective mechanism that says "You can only do X thing Y times per Z seconds."

### Real-World Analogy

**Example 1: Nightclub with a bouncer**
- The club can only fit 100 people
- Once it's full, the bouncer says "Wait, someone has to leave first"
- This prevents overcrowding and keeps everyone safe

**Example 2: Water faucet**
- You can turn it on full blast, but there's a maximum flow rate
- This prevents pipes from bursting due to pressure

**Example 3: Speed limit on highway**
- You CAN drive at 200 km/h, but the limit is 120 km/h
- This keeps everyone safe and traffic flowing smoothly

### How It Works
```
Without Throttling:
  Request 1 -----> Server [Process] OK
  Request 2 -----> Server [Process] OK
  Request 3 -----> Server [Process] OK
  Request 4 -----> Server [Process] OK
  Request 5 -----> Server [OVERLOADED!]
  Request 6 -----> Server [CRASH!]

With Throttling (max 3 requests per second):
  Request 1 -----> Server [Process] OK
  Request 2 -----> Server [Process] OK
  Request 3 -----> Server [Process] OK
  Request 4 -----> Server [429: Too Many Requests - Wait!]
  Request 5 -----> Server [429: Too Many Requests - Wait!]
  
  (after 1 second...)
  
  Request 4 -----> Server [Process] OK
  Request 5 -----> Server [Process] OK
```

### Common Throttling Strategies

| Strategy | How It Works | Best For |
|----------|--------------|----------|
| **Fixed Window** | Count requests in fixed time slots (e.g., per minute) | Simple rate limiting |
| **Sliding Window** | Track requests in a moving time window | Smoother limiting |
| **Token Bucket** | Tokens added over time, each request costs a token | Burst-friendly |
| **Leaky Bucket** | Requests processed at a fixed rate, excess queued | Consistent output rate |

### Token Bucket Example (Simplified)
```python
class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity      # Max tokens
        self.tokens = capacity         # Current tokens
        self.refill_rate = refill_rate # Tokens added per second
        self.last_refill = time.time()
    
    def allow_request(self):
        # Refill tokens based on time passed
        now = time.time()
        tokens_to_add = (now - self.last_refill) * self.refill_rate
        self.tokens = min(self.capacity, self.tokens + tokens_to_add)
        self.last_refill = now
        
        # Check if we have a token
        if self.tokens >= 1:
            self.tokens -= 1
            return True  # Request allowed
        return False     # Request denied (throttled)
```

### Use Cases
- **API protection**: Prevent abuse (e.g., max 100 requests per minute per user)
- **Login security**: Block brute-force attacks (e.g., max 5 login attempts per hour)
- **Email sending**: Prevent spam (e.g., max 100 emails per hour)
- **Social media**: Twitter/X limits tweets per hour
- **Search engines**: Limit crawling rate to not overload websites

### Throttling vs Rate Limiting
They're often used interchangeably, but there's a subtle difference:

| Aspect | Throttling | Rate Limiting |
|--------|------------|---------------|
| Behavior | Slows down or queues excess requests | Rejects excess requests immediately |
| User Experience | Smoother, requests eventually processed | Harder rejection, user must retry |
| Implementation | More complex (needs queue) | Simpler |

### HTTP Response for Throttled Requests
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
Content-Type: application/json

{
    "error": "Rate limit exceeded",
    "message": "You have made too many requests. Please wait 60 seconds.",
    "retry_after": 60
}
```

---

## Quick Comparison Table

| Concept | Direction | Connection Type | Best For |
|---------|-----------|-----------------|----------|
| **Pipe** | One-way flow | Chained processes | Data transformation, ETL |
| **Polling** | Client -> Server (repeatedly) | Short-lived | Simple updates, compatibility |
| **WebSocket** | Both ways (bidirectional) | Persistent | Real-time apps, chat, games |
| **Throttling** | N/A (control mechanism) | N/A | Protection, fair usage, stability |

---

## When to Use What?

### Choose Polling when:
- You need simple implementation
- Updates are not time-critical
- You need maximum compatibility
- Server doesn't support WebSockets

### Choose WebSocket when:
- You need real-time updates
- Both client and server need to send data frequently
- Low latency is crucial
- Building chat, games, or collaborative tools

### Use Throttling when:
- Protecting APIs from abuse
- Ensuring fair resource usage
- Preventing system overload
- Implementing security measures (login attempts)

---

## Further Reading
- [WebSocket Protocol (RFC 6455)](https://tools.ietf.org/html/rfc6455)
- [HTTP 429 Status Code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)
- [Rate Limiting Algorithms](https://en.wikipedia.org/wiki/Rate_limiting)

---

*Last updated: December 2024*

