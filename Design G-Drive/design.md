# Designing Cloud Storage System (SaaS Based)

## Introduction

This document outlines the high-level design for a cloud storage system similar to Google Drive, specifically tailored for a B2B (Business-to-Business) SaaS (Software as a Service) model. The system will allow businesses to store, manage, and access their files securely in the cloud.

## Requirements

- Users (businesses) have dedicated folders to organize their files.
- Users can upload various file types, including images, videos, PDFs, etc.
- Users can download and delete their uploaded files.
- The system should handle a large volume of file uploads and downloads from multiple users concurrently.

## Capacity Estimation

- Assuming 10,000 B2B users.
- Each user uploads approximately 2 million files per day.
- File sizes range from 1 MB to 1 GB.
- Total storage requirement per day = 2 * 10^6 * 10^4 * 10^3 MB/day = 2 * 10^13 MB/day = 2 PB/day

## APIs

- **UploadFileRequest**: `{ filemetaData: { filename, filesize }, upload_by, upload_at }`
- **UploadFileResponse**: `{ url: string, headers: {} }`
- **DownloadFileRequest**: `{ file_name: string }`
- **DownloadFileResponse**: `{ url: string }`

## Database Schema

### Users Table
Information about user accounts.

| Field          | Type          | Description                       |
| -------------- | ------------- | --------------------------------- |
| id             | INT           | Primary key, Auto-increment       |
| username       | VARCHAR(255)  | Unique username                   |
| email          | VARCHAR(255)  | User's email address              |
| password_hash  | VARCHAR(255)  | Hashed password                   |
| created_at     | TIMESTAMP     | Account creation timestamp        |

### Files Table
Metadata for each file.

| Field             | Type         | Description                         |
| ----------------- | ------------ | ----------------------------------- |
| id                | INT          | Primary key, Auto-increment         |
| user_id           | INT          | Foreign key to Users table          |
| original_filename | VARCHAR(255) | Original name of the file           |
| size              | BIGINT       | Size of the file in bytes           |
| content_type      | VARCHAR(50)  | MIME type of the file               |
| created_at        | TIMESTAMP    | File creation timestamp             |

### FileVersions Table
Track each file version.

| Field           | Type         | Description                          |
| --------------- | ------------ | ------------------------------------ |
| id              | INT          | Primary key, Auto-increment          |
| file_id         | INT          | Foreign key to Files table           |
| version_number  | INT          | Version number of the file           |
| s3_object_key   | VARCHAR(255) | Unique S3 path for file version      |
| is_latest       | BOOLEAN      | Is this the latest version           |
| created_at      | TIMESTAMP    | Version creation timestamp           |

### FileAccess Table
Permissions and sharing data.

| Field              | Type          | Description                       |
| ------------------ | ------------- | --------------------------------- |
| id                 | INT           | Primary key, Auto-increment       |
| file_id            | INT           | Foreign key to Files table        |
| shared_with_user_id| INT           | Foreign key to Users table (NULL if shared via link) |
| permission_type    | ENUM          | Type of permission ('read', 'write', 'owner') |
| shared_at          | TIMESTAMP     | Timestamp of when the file was shared |

### UploadTransactions Table
Log each file upload transaction.

| Field            | Type         | Description                      |
| ---------------- | ------------ | -------------------------------- |
| id               | INT          | Primary key, Auto-increment      |
| file_version_id  | INT          | Foreign key to FileVersions table|
| user_id          | INT          | Foreign key to Users table       |
| status           | ENUM         | Status ('in_progress', 'completed', 'failed') |
| started_at       | TIMESTAMP    | Upload start timestamp           |
| completed_at     | TIMESTAMP    | Upload completion timestamp      |

### DownloadTransactions Table
Log each file download request.

| Field          | Type         | Description                       |
| -------------- | ------------ | --------------------------------- |
| id             | INT          | Primary key, Auto-increment       |
| file_id        | INT          | Foreign key to Files table        |
| user_id        | INT          | Foreign key to Users table        |
| status         | ENUM         | Status ('in_progress', 'completed', 'failed') |
| started_at     | TIMESTAMP    | Download start timestamp          |
| completed_at   | TIMESTAMP    | Download completion timestamp     |

### Additional Considerations
- Proper indexing on search-heavy columns for performance.
- Timestamp fields auto-record creation/modification dates.
- The schema is optimized for relational databases like MySQL or PostgreSQL.
- This design is scalable and ensures robust data management for cloud storage systems.

## Key Design Considerations

- Files should not be directly uploaded to the server due to their large size, as it would increase the load on the server.
- Allowing clients to upload files directly to a storage service like Amazon S3 poses security risks, as malicious users could potentially access files they should not have access to.
- The backend server will generate a pre-signed URL and send it back to the client for secure file uploads.
- Clients will upload files to the pre-signed URL endpoint, which is secure and time-limited.

## Storage Structure on Amazon S3 Buckets

- AWS S3 has a limit of 100 buckets per account, so creating separate buckets for each user/business is not feasible.
- Files will be stored using a naming convention like `user_id/file_name`, as S3 does not have a folder-like structure internally.
- Access control lists (ACLs) are used to control access to files at the file level.
- Pre-signed URLs provide short-term access to private objects in the S3 bucket, ensuring secure file uploads and downloads.

## Billing and Storage Tracking

- A separate metadata database will store file metadata, including file size, to accurately track and bill users based on their storage usage.
- To ensure consistency between the metadata database and the actual file sizes stored in S3, a Change Data Capture (CDC) mechanism can be implemented.
- CDC events from S3 logs can be captured using AWS Lambda and stored in a queue (e.g., AWS SQS) for processing and updating the metadata database.

## High-Level Architecture

1. **Client Applications**: Desktop, web, and mobile applications that enable users to interact with the cloud storage system.
2. **API Gateway**: A unified entry point for client applications, responsible for request routing, authentication, and rate limiting.
3. **Backend Server**: Handles file upload and download requests, generates pre-signed URLs, and manages file metadata.
4. **Metadata Database**: Stores file metadata, including file size, owner, permissions, and other relevant attributes.
5. **Amazon S3**: A distributed storage service used to store and retrieve file contents securely.
6. **AWS Lambda**: Used for capturing and processing CDC events from S3 logs.
7. **AWS SQS**: A queue service for storing CDC events and processing them asynchronously.
8. **Monitoring and Logging**: Collects and analyzes system metrics, logs, and error reports for monitoring, debugging, and performance optimization.
9. **Security and Access Control**: Implements authentication, authorization, and encryption mechanisms to ensure data privacy and secure access to files and services.

## Conclusion

This high-level design outlines the key components and considerations for building a cloud storage system tailored for B2B SaaS businesses. By leveraging services like Amazon S3, AWS Lambda, and AWS SQS, the system can achieve scalability, security, and accurate billing based on storage usage. Further refinement and detailed design decisions would be required to address specific requirements, scalability needs, and implementation details.

## Final Design

![screenshot](https://github.com/khansamad99/Famous-System-Design-Problems-/blob/main/images/gdrive/Screenshot 2024-04-05 at 4.41.23 PM.png)
