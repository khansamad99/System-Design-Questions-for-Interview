# System Design Requirements for URL Shortening Service

## Functional Requirements

1. **Generate Short URLs:** The system must generate a unique, short URL for a given long URL.
2. **URL Redirection:** Upon accessing a short URL, the system should redirect the user to the original long URL.
3. **Analytics Tracking:** Track the number of clicks per short URL for analytics purposes.

## Non-Functional Requirements

1. **High Availability:** The system must be highly available, ensuring that short URLs are accessible at all times.
2. **Scalability:** Must be able to support a large number of URL shortenings and redirects (e.g., a trillion URLs, with median URL receiving 10,000 clicks).
3. **Performance:** System must offer fast response times for both creating short URLs and redirecting to long URLs.
4. **Consistency:** Click analytics should eventually be consistent, ensuring accuracy in the number of clicks reported.
5. **Durability:** Analytics data should be durable and not lost in case of system failures.

## Considerations

1. For url has of n=8, We have 36 chracters say(A-Z,a-z,0-9) so we have around 36*36.. 2 trillion combinations

2. Hash Collisions ? Channing or probing - Implement probing for the next available hash in case of collisions.

// Speed up writes
// Not important to mention initially, but good to know for follow-up
3. We need to use Single Leader replications DBs as in multi-leader replication/leaderless replication, if there are collisions of short-url for 2 different URLs, client met get re-directed to different URL. This rules out DB like Cassandra, Scylla etc. Use Single-leader replication DB.

4. Same above issue with write-back caching to speed up writes.

5. We can do URL - Partioning to speed up

- Not only is partioning very important if we have lot of data, it can help us speed up our reads and writes by reducing load on every node.

- Can partition by range of short-URL as they are already hashed so should be relatively even.

- Allow probing to another hash to stay local to one DB.

## Database Schema

| tinyURL   | actual URL       | User Id | Create Time | Expire Time | Clicks |
|-----------|------------------|---------|-------------|-------------|--------|
| ac1zueb6  | google.com       | 22      | 11/13/23    | 11/20/23    | 500    |
| d4k5a7221 | amazon.com       |         |             |             |        |
| d4k5a7221 | apple.com        |         |             |             |        |

- Index on tinyURL, can make queries faster
- We can use MySQL as data is partioned, we need single leader replication and it also use B-Tree based so faster reads.
- We can use a KV store as well depends on the use-case nothing is fixed.



## Increase Read Speeds


**Cache:** Reduces database load by caching popular short URLs and their corresponding long URLs.

**Read Replicas;** - Add read replicas to master

## Analytics-Stream Processing

- Database would be slow - if we keep on adding Clicks data to DB
- We can use Log based message broker , it will give throughput for large amount of Data- say Kafka

## Analytics

- HDFS + Spark, Flink
