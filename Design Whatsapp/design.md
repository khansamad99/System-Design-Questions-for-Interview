# Facebook Messenger / WhatsApp System Design


## Functional Requirements
- Support for individual and group chats (up to 100 users).
- Capability to send and receive messages in real-time on all client devices.
- Persistence of messages in a database for historical access.
- Should support file sharing (image, video, etc.)

## Non Functional Requirements
- High availability with minimal latency.
- The system should be scalable and efficient.

## Capacity Estimates
- 50M DAU
- Each user sends about 40 messages.
- This gives us 2 billion messages per day.
- 5 percent of these have media
- 200 million files we would need to store/day.
- 100 bytes avg message * 2B = 200GB/day
- 50KB each file = 10TB storage/day

## Database Schema

## Database Tables

### Users
| Field         | Type    |
|---------------|---------|
| id            | uuid    |
| name          | varchar |
| phoneNumber   | varchar |


### Users Members
- This table maps users and groups as multiple users can be a part of multiple groups (N:M relationship) and vice versa.

| Field    | Type |
|----------|------|
| id       | uuid |
| userID   | uuid |
| groupID  | uuid |

### Groups
| Field       | Type       |
|-------------|------------|
| id          | uuid       |
| name        | varchar    |
| description | text       |
| avatar_url  | varchar    |
| created_at  | timestamp  |


### Users Chats

- This table maps users and chats as multiple users can have multiple chats (N:M relationship) and vice versa.

| Field   | Type |
|---------|------|
| id      | uuid |
| userID  | uuid |
| chatID  | uuid |

### Messages
| Field       | Type       |
|-------------|------------|
| id          | uuid       |
| userID      | uuid       |
| chatID      | uuid?      |
| groupID     | uuid?      |
| type        | enum       |
| content     | varchar    |
| sentAt      | timestamp  |
| deliveredAt | timestamp? |
| seenAt      | timestamp? |

### Chats

- This table basically represents a private chat between two users and can contain multiple messages.

| Field      | Type |
|------------|------|
| id         | uuid |
| messageID  | uuid |


## API Design

```
/getAll(userID) - Get all chats or groups
/getMessages(userID,channelID) - Get all messages for a user given the channelID (chat or group id).
/sendMessage(userID,channelID,message) - Send a message from a user to a channel (chat or group).

/joinGroup(userID,channelID)
/leaveGroup(userID,channeID)

```

### Architecture
- We can microservices architecture since it will make it easier to horizontally scale and decouple our services. Each service will have ownership of its own data model

#### User Service
- This is an HTTP-based service that handles user-related concerns such as authentication and user information.

#### Chat Service
- The chat service will use WebSockets and establish connections with the client to handle chat and group message-related functionality. We can also use cache to keep track of all the active connections sort of like sessions which will help us determine if the user is online or not.

#### Notification Service
- This service will simply send push notifications to the users. It will be discussed in detail separately.

#### Presence Service
- The presence service will keep track of the last seen status of all users.

#### Media service
- This service will handle the media (images, videos, files, etc.) uploads.

### Real Time Messaging

- #### Pull Model : 
The client can periodically send an HTTP request to servers to check if there are any new messages. This can be achieved via something like Long polling.

- #### Push Model :
The client opens a long-lived connection with the server and once new data is available it will be pushed to the client. We can use WebSockets for this.

The pull model approach is not scalable as it will create unnecessary request overhead on our servers and most of the time the response will be empty, thus wasting our resources. To minimize latency, using the push model with WebSockets is a better choice because then we can push data to the client once it's available without any delay given the connection is open with the client.


### Last Scene
- To implement the last seen functionality, we can use a heartbeat mechanism, where the client can periodically ping the servers indicating its liveness. This functionality will be handled by the presence service combined with Redis as our cache.

### Data Partioning


### Caching
In a messaging application, we have to be careful about using cache as our users expect the latest data, but many users will be requesting the same messages, especially in a group chat. So, to prevent usage spikes from our resources we can cache older messages.

Some group chats can have thousands of messages and sending that over the network will be really inefficient, to improve efficiency we can add pagination to our system APIs. This decision will be helpful for users with limited network bandwidth as they won't have to retrieve old messages unless requested.

#### Which cache eviction policy to use?

We can use solutions like Redis or Memcached and cache 20% of the daily traffic but what kind of cache eviction policy would best fit our needs?

Least Recently Used (LRU) can be a good policy for our system. In this policy, we discard the least recently used key first.

#### How to handle cache miss?

Whenever there is a cache miss, our servers can hit the database directly and update the cache with the new entries.

### Few Bottlenecks to consider

- "What if one of our services crashes?"
- "How will we distribute our traffic between our components?"
- "How can we reduce the load on our database?"
- "How to improve the availability of our cache?"
- "Wouldn't API Gateway be a single point of failure?"
- "How can we make our notification system more robust?"
- "How can we reduce media storage costs"?
- "Does chat service has too much responsibility?"
### To make our system more resilient we can do the following:

- Running multiple instances of each of our services.
- Introducing load balancers between clients, servers, databases, and cache servers.
- Using multiple read replicas for our databases.
- Multiple instances and replicas for our distributed cache.
- We can have a standby replica of our API Gateway.
- Exactly once delivery and message ordering is challenging in a distributed system, we can use a dedicated message broker such as Apache Kafka or NATS to make our notification system more robust.
- We can add media processing and compression capabilities to the media service to compress large files similar to Whatsapp which will save a lot of storage space and reduce cost.
- We can create a group service separate from the chat service to further decouple our services.

