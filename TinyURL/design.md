# System Design Requirements for URL Shortening Service

## Functional Requirements

1. **Generate Short URLs:** The system must generate a unique, short URL for a given long URL.
2. **URL Redirection:** Upon accessing a short URL, the system should redirect the user to the original long URL.
3. **Analytics Tracking:** Track the number of clicks per short URL for analytics purposes.

## Non-Functional Requirements

1. **High Availability:** The system must be highly available, ensuring that short URLs are accessible at all times.
2. **Scalability:** Must be able to support a large number of URL shortenings and redirects (e.g., 100M URLs/month, with median URL receiving 10,000 clicks).
3. **Performance:** System must offer fast response times for both creating short URLs and redirecting to long URLs.
4. **Consistency:** Click analytics should eventually be consistent, ensuring accuracy in the number of clicks reported.
5. **Durability:** Analytics data should be durable and not lost in case of system failures.


## Considerations

1. For url has of n=8, We have 36 chracters say(A-Z,a-z,0-9) so we have around 62*62.. 2 trillion combinations

2. Hash Collisions ? Channing or probing - Implement probing for the next available hash in case of collisions.

- Speed up writes
- Not important to mention initially, but good to know for follow-up
3. We need to use Single Leader replications DBs as in multi-leader replication/leaderless replication, if there are collisions of short-url for 2 different URLs, client met get re-directed to different URL. This rules out DB like Cassandra, Scylla etc. Use Single-leader replication DB.

4. Same above issue with write-back caching to speed up writes.

5. We can do URL - Partioning to speed up

- Not only is partioning very important if we have lot of data, it can help us speed up our reads and writes by reducing load on every node.

- Can partition by range of short-URL as they are already hashed so should be relatively even.

- Allow probing to another hash to stay local to one DB.

## Database Schema and Storage

Amount of data/month = 100M * (8 bytes(short-url) + 120(url) + 60(extra data))
                     = 18.8 GB

Storage is not the concern, Load is()

| tinyURL   | actual URL       | User Id | Create Time | Expire Time | Clicks |
|-----------|------------------|---------|-------------|-------------|--------|
| ac1zueb6  | google.com       | 22      | 11/13/23    | 11/20/23    | 500    |
| d4k5a7221 | amazon.com       |         |             |             |        |
| d4k5a7221 | apple.com        |         |             |             |        |

- Index on tinyURL, can make queries faster
- We can use MySQL as data is partioned, we need single leader replication and it also use B-Tree based so faster reads.



## What happens when someone vists a URL url.com/abqewrw

- When someone visits a short URL, the request comes to our API server and we get the actual URL from shard and we return 301 redirect with this URL.

- But before returning the URL , we emit an event to register the analytics
## Increase Read Speeds


**Cache:** Newly published and populur URLs are expected to get more reads so it reduces database load by caching popular and new short URLs and their corresponding long URLs.

**Read Replicas;** - Add read replicas to master

## Analytics-Stream Processing

- Database would be slow - if we keep on adding Clicks data to DB
- We can use Log based message broker , it will give throughput for large amount of Data- say Kafka

## Analytics

- HDFS + Spark, Flink


## How to Shorten URL
### Random Integer Generation
- Generating random integers sequentially or randomly without considering uniqueness is prone to collisions.
- To ensure uniqueness, a few database servers are set up with the job of atomically issuing one unique random integer.
- This makes it difficult for multiple requests to generate the same integer, regardless of the number of requests hitting the database concurrently.

### Partitioning Technique
- The range of possible integers (e.g., 0 to 1000) is partitioned into smaller ranges (e.g., 0-250, 251-500, 501-750, 751-1000).
- Each partition is assigned to a separate "Ticket Server" that manages and issues integers from its range.
- When a user requests a short URL, the API server selects a Ticket Server at random and obtains the "current" value from the selected range.
- The Ticket Server increments the "current" value by 1 and returns it to the API server.
- If a range is exhausted, the Ticket Server removes that range from its in-memory store and loads a new range.

### Mapping and Storage
- The API server encodes the issued random integer into a short URL using a suitable mapping technique (e.g., base conversion, hash function).
- The mapping of the short URL to the original long URL is stored in a separate database for redirection purposes.

### Characteristics
- Within each range, the order of issued integers is sequential, but across ranges, the order appears pseudorandom due to the random selection of ranges.
- This approach leverages partitioning, sharding, and transactions to achieve uniqueness and scalability while maintaining a pseudorandom order of generated short URLs.
- The design aims to distribute the load across multiple Ticket Servers and database servers, preventing any single point of bottleneck or failure.

This high-level design provides a robust and scalable solution for generating unique short URLs in a distributed system while ensuring proper load distribution and collision avoidance.

## Final Design

![screenshot](https://github.com/khansamad99/Adventure-Space/blob/main/frontend/src/images/image%20(1).png)