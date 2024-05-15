# Designing a Video Streaming Platform like Netflix/YouTube

## Functional Requirements
1. Users can post videos.
2. Users can watch videos.
3. Users can comment on videos.
4. Users can search for videos by name.

## Non-Functional requirements
1. High availability with minimal latency.
2. High reliability, no uploads should be lost.
3. The system should be scalable and efficient.

## Capacity Estimates
- 1 billion users
- Average video size: 100 MB
- 1 million videos posted per day
- Storage required per year: 40 PB

## Video Streaming Concepts
- Support multiple devices and network speeds
- Adaptive bitrate streaming
- Video chunking for efficient loading and playback

## Video Chunking
- Split videos into smaller chunks
- Enables parallel uploads, lower barrier to start watching, and adaptive streaming
- Chunks stored in a distributed file system or object store

## Database Schema
1. Subscribers Table
  - User ID (partition key)
  - Subscribed User ID

2. User Videos Table
  - User ID (partition key)
  - Video ID
  - Timestamp
  - Video Metadata

3. Users Table
  - User ID (partition key)
  - Email
  - Password

4. Comments Table
  - Video ID (partition key)
  - Comment Poster ID
  - Timestamp
  - Comment Content

5. Video Chunks Table
  - Video ID (partition key)
  - Encoding
  - Resolution
  - Chunk Order
  - Chunk Hash
  - Chunk URL

## Database Choices
- MySQL for read-heavy tables (Subscribers, User Videos, Users)
- Cassandra for write-heavy tables (Video Comments)

## API Design

```
/uploadVideo(title,description,data:bytes)
/streamVideo(videoID,codec-videoQuality,resolution,offset)
/searchVideo(query)
/addAComment(videoID,comment)

```

### Architecture
- We can microservices architecture since it will make it easier to horizontally scale and decouple our services. Each service will have ownership of its own data model

#### User Service
- This is an HTTP-based service that handles user-related concerns such as authentication and user information.

#### Stream Service
- The tweet service will handle video streaming-related functionality.

#### Search Service
- The service is responsible for handling search-related functionality

#### Analytics Service
- This service will be used for metrics and analytics use cases.

#### Media service
- This service will handle the video uploads and processing. 


### Video Upload Process

1. Raw video footage can be large (e.g., 4 TB for 8K footage), requiring processing to reduce storage and delivery costs.
2. Uploaded videos are queued for processing in a message queue.
3. Processing pipeline steps:
- File Chunker: Splits video into smaller chunks based on timestamps or scenes.
- Content Filter: Checks video adherence to platform's content policy using machine learning.
- Transcoder: Decodes and re-encodes video to optimize format and reduce size.
- Quality Conversion: Converts transcoded media into different resolutions (4K, 1080p, etc.).
4. Processed videos are stored in a distributed file storage or object storage for retrieval during streaming.
5. Message queue (e.g., Amazon SQS, RabbitMQ) is used to decouple video processing pipeline from uploads functionality.

## Search Index
- Build an inverted index for video title and description
- Partition the index based on term popularity
- Denormalize video metadata in the search index for faster reads
- Use change data capture to update the search index when video metadata changes

## Final Points

Here are the key points from the given text in a shortened format:

1. Searching:
- Traditional DBMS may not be performant enough for storing, searching, and analyzing huge volumes of data quickly.
- Elasticsearch, built on Apache Lucene, is a distributed, open-source search and analytics engine suitable for this use case.

2. Trending Content:
- Cache most frequently searched queries in the last N seconds and update every M seconds using a batch job mechanism.

3. Sharing:
- Implement a URL shortener service to generate short URLs for users to share content.

4. Data Partitioning:
- Use horizontal partitioning (sharding) to scale out databases.
- Partitioning schemes: Hash-Based, List-Based, Range-Based, Composite Partitioning.
- Consistent hashing can help with uneven data and load distribution.

4. Geo-blocking:
- Restrict content based on geographical areas or countries due to legal distribution laws.
- Determine user's location using IP or region settings and use services like Amazon CloudFront or Amazon Route53 for geo-restriction.

5. Recommendations:
- Use machine learning models (e.g., Collaborative Filtering, Netflix Recommendation Engine) to predict user preferences based on various data points.

6. Metrics and Analytics:
- Capture data from different services and use Apache Spark for large-scale data processing and analytics.
- Store critical metadata in views table to increase data points.

7. Caching:
- Cache static media content to improve user experience.
- Use solutions like Redis or Memcached with Least Recently Used (LRU) cache eviction policy.
- On cache miss, servers can directly hit the database and update the cache.

8. Media Streaming and Storage:
- Use distributed file storage (HDFS, GlusterFS) or object storage (Amazon S3) for storing and streaming media files.

9. Content Delivery Network (CDN):
- Use CDN (Amazon CloudFront, Cloudflare) to increase content availability, reduce bandwidth costs, and serve static files.


## Final Design

![screenshot](https://github.com/khansamad99/System-Design-Questions-for-Interview/blob/main/images/youtube/Screenshot%202024-05-15%20at%2010.26.53%E2%80%AFPM.png)