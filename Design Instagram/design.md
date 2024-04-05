# Design Instagram/Twitter/Facebook

## Functional Requirements
- Create Posts/Upload posts
- Follow Users
- View Feed
- Seach for a user
- React on a post

## Asumptions

- User profile is created
- Auth Service is provided

## Non-Functional Requirements
- Adding Like/Comments(Low Latency)
- View Post/Likes/Comments(Eventual Consistency)

## Capacity Estimation:
- DAU = 100 MAU /30 days = (2 * 10 ^ 9)/30 = 6.6 M approx
- Assume 10% of users post 4 times a month
  = 10m users * 4 posts/30 days
  = (10*10^6 * 3)/30 days
  = 1.33M posts/day

- Average post size: 10 MB
- Storage requirement: 13.3 TB per day

## Database Design

## User Database

| Field       | Type        | Description                |
|-------------|-------------|----------------------------|
| User ID     | Primary Key | Unique identifier for user |
| Username    | String      | User's chosen name         |
| Name        | String      | User's full name           |
| Email       | String      | User's email address       |
| Phone Number| String      | User's phone number        |
| Login Times | DateTime    | Timestamps of user logins  |

## Follow/Follower- Friends/Connection Database

| Field         | Type            | Description                                            |
|---------------|-----------------|--------------------------------------------------------|
| User ID 1     | Foreign Key     | Identifier for the first user                          |
| User ID 2     | Foreign Key     | Identifier for the second user                         |
| Relationship  | Directed/Undirected | Defines the type of relationship (follow/friend) |

## Post Database

| Field        | Type          | Description                                |
|--------------|---------------|--------------------------------------------|
| Post ID      | Primary Key   | Unique identifier for the post             |
| User ID      | Foreign Key   | Identifier for the user who created the post |
| Date and Time| DateTime      | When the post was created                  |
| Description  | Text          | Text content of the post                   |
| Media Link   | Object Storage Link | Link to the media object stored in services like Amazon S3 |

## Reactions Database

| Field        | Type          | Description                                |
|--------------|---------------|--------------------------------------------|
| Post ID      | Foreign Key   | Identifier for the post being interacted with |
| User ID      | Foreign Key   | Identifier for the user interacting with the post |
| Liked        | Boolean       | Indicates if the post is liked             |
| Comment      | Text          | Text of the user's comment on the post     |
| Media        | Object Storage Link | Link to the emoji or other media files   |


## APIs

- **Get Feed Service**: Returns the top 20 posts for a user's news feed.
- **Create Post Service**: Allows users to create posts and handles media uploading.
- **Interaction Service**: Handles likes and comments on posts.
- **Search Service**: Returns top search results based on the query string.
- **Follow Service**: Establishes following relationships between users.


## High-Level System Design for Scaling:
- Add load balancers with standby to distribute load and avoid single points of failure.
- Use caching for frequently accessed data, such as user feeds.
- Have multiple service instances for high availability.
- Multiple database replicas for data redundancy.
- Messaging queues to decouple services like media upload from post creation.
- Archive older posts to separate databases for optimized performance.
- Elasticsearch for improved search capabilities with caching for frequent queries.
- Content Distribution Network (CDN) for distributing posts for all users from different regions around the world
- Special sharding considerations to handle the large following of celebrity users.

## Special Considerations:
- Archiving strategies for posts and media for cost savings and performance.
- Sharding strategies must account for high-traffic users to prevent system overloads.

## Final Design

![screenshot](https://github.com/khansamad99/Famous-System-Design-Problems-/blob/main/images/gdrive/Screenshot%202024-04-05%20at%204.41.23%20PM.png)

