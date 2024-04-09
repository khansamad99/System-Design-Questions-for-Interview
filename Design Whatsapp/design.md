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
- Utilizes websockets for a persistent, bi-directional communication channel between clients and chat servers.
- Employs load balancers and zookeeper for consistent hashing and to direct users to the correct chat server.
- Incorporates heartbeats between clients and servers to manage connections and avoid Thundering Herd issues with reconnections.

### Operations
- Sending messages involves the client, chat service, Kafka, Flink, and ultimately the HBase chats table.
- Receiving messages uses Flink to identify which users need to receive a message and routes it through load balancers to the correct chat service and then to the client.
- Historical messages are fetched from the HBase chats table, leveraging its efficient read capabilities.

## Conclusion
The design aims to handle the massive scale of global messaging apps by carefully choosing storage solutions, optimizing for both reads and writes, and using real-time data processing to manage message delivery efficiently.
