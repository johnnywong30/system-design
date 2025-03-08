# Design Bit.ly

## Requirements

### Functional Requirements
1. Users will be able to shorten a URL into a shortened version.
   1. Users will be able to provide a custom alias.
   2. Users will be able to set an expiration time for the shortened URL.
2. Users will be able to click on the shortened URL and be redirected to the original URL.

### Non-functional Requirements
1. The system should ensure uniqueness for shortened URLs.
2. The system should respond with minimal latency (less than 100ms).
3. The system should be reliable and highly available (>= 99.99% availability)
4. The system should scale to support 1B shortened URLs and 100M Daily Active Users

## Core Entities
1. Original URL
2. Shortened URL
3. User

## API
RESTful API
```
POST /v1/urls
body: {
    "originalUrl": string,
    "alias?: string,
    "expirationTime"?: number
} -> {
    "shortUrl": string
}

GET /v1/urls/{shortId} -> HTTP 302 Redirect to Original URL


```
## High Level Design
![High Level Design](/bitly/high-level-design.png)

## Deep Dive
### How do we ensure uniqueness for our shortened URLs?
#### Random Number Generator or Hash Function
We can generate a semi-unique random number to use as our shortId.

We can use a Hash function to hash the originalUrl. This is a deterministic way of getting a unique output for every unique input.

Then we can encode the output of either RNG or Hash with base62 (we don't use base64 to exclude "+" and "/" already used in URLs).

The tradeoff here is there might be collisions at some point (Birthday Problem). To avoid collisions, we need higher entropy (longer URLs), but that defeats the purpose of a URL shortener. Detecting and resolving collisions can add latency and complexity.

Tradeoff between efficiency, uniqueness, and length.

#### Better Solution: Unique Counter with Base62 Encoding
We can increment a Unique Counter, especially using Redis's atomic `INCR` function. Each unique URL will correspond to a unique number without duplicates or gaps. This eliminates the problem of collisions.

The challenge here is synchronizing this unique counter in a distributed system. We'll talk more about this when we discuss scaling the system.

If we have 1B URLs, in base62, we have shortIds of 6 characters long.
We need to move up to 7 characters when we approach 62^7 URLs.

### How do we respond with minimal latency?
#### Index shortId in Database
Our shortId can be the PK in the database.
Reduces search time.

Problem with indexing is support 100M DAUs.
Assume each user performs 5 redirects/day.
Means 500M redirects/day -> 500M / 86,400 seconds = 5787 redirects/second

A single database instance may struggle to keep up with this amount of traffic.

#### In-Memory Cache (Redis)
Stores frequently accessed mapping of shortIds to URLs.
Check cache first for URL to redirect to. If its not there, check database, cache URL if available, and then redirect to URL.

Challenges with a Cache Here Include:
1. Cache invalidation, especially with updates/deletes. Issue is minimized since URLS are more read heavy and rarely change.
2. Time to Warm Up for Initial Requests
3. Memory limitation; decisions about cache size and eviction policies

### How do we scale to support 1B shortened URLs and 100M DAUs?
Any database should suffice for 1B shortened URLs.

Recovery mechanisms:
1. Database Replication
2. Database Backup

Reads are more frequent than writes. Scale primary server by splitting into a microservice. Read microservice and write microservice.

Centralized Redis instance for global counter.
Don't need to get a unique counter each write request. Request a batch of counters (i.e. 1000/batch) and reference them locally. When the batch is exhausted, request a new batch.

### Final Design
![Final Bitly Design](/bitly/deep-dive-design.png)