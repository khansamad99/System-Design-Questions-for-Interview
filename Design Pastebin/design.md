# Designing Pastebin (GitHub Gist)

Pastebin is an online platform that allows you to:
- Store any text file (content) and make it public to anyone
- Share it with anyone (publicly or secretly) via a unique URL 
- Set an expiration for auto deletion
- Files can be long (10MB max)
- One who created the paste can edit it as well 

## Functional Requirements
- Allow users to store and share text files up to 10MB in size
- Generate unique URLs for each paste to allow sharing
- Support setting an expiration time for auto deletion of pastes
- Allow the creator of a paste to edit it
- Provide options for public or secret (password-protected) sharing

## Non-Functional Requirements
- High availability and low latency for paste access
- Scalability to handle a large number of paste uploads and reads
- Durability and reliability of stored pastes
- Security measures to prevent unauthorized access to secret pastes

## Capacity Estimation
- Assume 10 million new pastes per month
- Average max paste size: 10 MB
- Total storage req = 10M * 10MB = 10,000,000 * 10,000,000 Bytes = 100 TB per month
- Write to Read ratio = 1:50
- Total data read = 5 PB/month (Bandwidth requirement)

## File Storage Options
We have two options for file storage: 
1. Database (e.g. relational DB)
2. Blob store (e.g. S3)

Since files can be large and per file size is long enough, there is no rationale to store files in a database. Storing files on S3 and metadata in a relational DB is recommended.

## Database Schema
A simple database schema for storing the file metadata:

| Field       | Type        | NumberOfBytes              |
|-------------|-------------|----------------------------|
| ID          | Primary Key | 36(uuid)                   |
| Name        | String      | 120                        |
| createdAt   | String      | 4                          |
| visiblity   | String      | 4 (Public/Secret)          |
| ownerId     | DateTime    | 4                          |

## API and Compute
API servers are needed for uploading and accessing the files. A Load Balancer can be added to distribute traffic.

The API flow for paste creation:
1. User sends paste content via HTTP POST to the API server
2. API server generates a unique UUID for the paste
3. API server stores the paste content in S3 with the UUID as the key
4. API server creates an entry in the metadata DB with paste details
5. API server returns the unique URL for the paste to the user

The API flow for paste retrieval:
1. User sends a GET request with the paste UUID to the API server
2. API server looks up the metadata in the DB based on the UUID
3. API server fetches the paste content from S3 using the UUID
4. API server returns the paste content in the HTTP response

## Caching (Optional)
- Frequently accessed pastes can be cached to improve read latency
- Implement a cache eviction strategy based on access frequency
- Consider adding caching if there are pastes with "huge" access frequency
- Be cautious about cache size and cost vs. performance tradeoff
- If access frequency is not high, serve directly from S3

## Expiration
- The owner of the file can set an expiration time for a file, after which it becomes    inaccessible and eventually deleted.
- When a file is accessed, the system checks the expiration time. If the expiration time has passed, the system returns a 404 error. Otherwise, the system fetches the file and returns it to the user.
- A background process periodically cleans up expired files from the storage system.

## Analytics

- Whenever a file is accessed, the API server captures metadata about the request, such as the IP address of the user and the time of access.
- This metadata is stored in a separate database (e.g., Elasticsearch) for analytics purposes.
- The analytics system can be used to track how often files are being accessed, from where they are being accessed, and by whom.

## Fault Tolerance

- Backups: Regular backups of the metadata database and the stored files should be created. These backups can be used to recover data in case of a system failure.
- Replication: The metadata database and the file storage system can be replicated across multiple servers. This ensures that if one server fails, the other servers can still be used to access data.

## Final Design
