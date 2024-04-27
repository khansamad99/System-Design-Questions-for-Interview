# Designing a Rate Limiter

## Introduction
A rate limiter is a mechanism used to prevent malicious or negligent users from submitting too many requests to a service, which could potentially bring it down or add extra load. The goal is to build a rate limiter that introduces minimal latency and supports various rate limiting techniques.

## Capacity Estimates
Assuming:
- 1 billion users
- 20 services to rate limit
- User ID (8 bytes) and request count (4 bytes) per user per service

Total storage required: 1 billion users * 20 services * 12 bytes = 240 GB

If using an in-memory solution, the data will likely need to be partitioned across multiple nodes.

## Rate Limiting Criteria
1. User ID
   - Pros: Easy to track, passed in every request
   - Cons: Users can create multiple accounts to bypass rate limiting, not applicable for non-authenticated endpoints
2. IP Address
   - Pros: Can track users creating multiple accounts from the same network, works for non-authenticated endpoints
   - Cons: Can lead to false negatives (e.g., multiple users behind the same IP)

A hybrid approach using IP address for non-authenticated endpoints and user ID for authenticated endpoints is recommended.

## Rate Limiting Interface
```typescript
function rateLimit(userId: string | null, ipAddress: string, serviceName: string, requestTime: timestamp): boolean
```


# Rate Limiter Placement

## Local rate limiting (on each service)
- **Pros:** No extra network calls
- **Cons:** Requests still reach the service (using bandwidth), application and rate limiting scaling are tightly coupled, rate limiting data lost if service goes down

## Dedicated distributed rate limiter layer
- **Pros:** Shields application servers from large bursts of traffic, can scale rate limiter independently
- **Cons:** Extra network call to rate limiter, increasing latency

To reduce extra network requests, a write-back cache can be implemented on the load balancer for popular user IDs.

# Database Choice

To ensure minimal latency, an in-memory database like Redis or Memcached is recommended. Redis is preferred for its built-in data structures and easy replication setup.

# Replication Choice

Single-leader replication is recommended to ensure accurate rate limiting counts from the start. Multi-leader or leaderless replication with CRDTs could be used, but eventual consistency may lead to temporary inaccuracies in rate limiting.

# Rate Limiting Algorithms

## Fixed Window Rate Limiting
- Requests are limited within a fixed time window (e.g., 2 requests per minute)
- Implemented using a map data structure: service -> userID -> (currentWindow, requestCount)

## Sliding Window Rate Limiting
- Requests are limited within a sliding time window (e.g., max 2 requests within any 1-minute window)
- Implemented using a linked list data structure
- On each request:
  1. Purge old entries from the linked list
  2. Add new request to the linked list if valid, otherwise rate limit

# Concurrency Considerations

Both the fixed window (counter) and sliding window (linked list) implementations require proper locking or single-threaded execution to avoid race conditions and concurrent modification issues.

# Admin Console

It is better to have a small Admin Console (Frontend/Backend) that is used by the developers to react countings and debug when things go wrong.
The service will also provide observability on infra and rate limiting DBs like #keys, # request blocked, cpu/mem load, etc.
# Final Design

- Load balancer with a rate limiter cache for common requests
- Redis partitions with single-leader replication for rate limiting data
- Rate limiting service is called first, followed by the backend service if not rate limited
