# Design Dropbox

## Requirements
### Functional Requirements
1. Users should be able to upload files.
2. Users should be able to download files.
3. Users should be able to share files and view files shared with them.
4. Users should be able to auto-sync files across devices.

### Non-functional Requirements
1. The system should be highly available.
2. The system should be able to support large files (50 GB max).
3. The system should be secure and reliable.
4. The system should be able to recover lost or corrupted files.
5. The system should make upload, download, and sync times as fast as possible (low latency).

## Core Entities
1. File
2. File Metadata
3. User

## API
RESTful API. Always give yourself the minimum APIs to meet requirements. Eventually, the APIs can change as you design the system further.
```
POST /v1/files
body: {
    file: File,
    metadata: FileMetadata
}

GET /v1/files/:fileId -> File & FileMetadata

POST /v1/files/:fileId/share
body: {
    users: User[]
}

GET /v1/files/:fileId/changes -> FileMetadata[]
```
## High Level Design
![Design](/dropbox/design.png)

We upload files directly to our blob storage (S3) using a presigned URL.

Users will be able to download files from a CDN to cache the files closer to the users' locations. This reduces latency and speeds up downloads.

### CDN Challenges
1. Expensive; use deliberate caching and eviction policies. Only files that are frequently accessed are cached.

### Sharelist
1. Separate collection
2. Query using an index

### Sync 
#### Local to Remote
1. Monitor local Dropbox folder changes
2. When change detected, monitoring agent queues modified file for upload
3. Use upload API to send changes to server along with updated metadata
4. Conflicts are resolved using a "last write wins" strategy

Versioning can be done using updated pointers to newer versions of files and metadata.

#### Remote to Local
Polling vs WebSockets

Hybrid Approach
1. Polling for stale files that are changed infrequently
2. WebSockets for files changed recently

## Deep Dive
### How to Support Large Files?
1. Progress Indicator
2. Resumable Uploads

Usage of chunking files on the client.
Track chunks uploaded and the status in database.

Use S3 Event Notifications to know when a chunk for a file has been uploaded. We can update the database and status of a chunk using these notifications to trigger a Lambda function.

Get fingerprint of files (MDHash).
### How to Make Uploads/Downloads/Syncs as Fast as Possible
Use CDNs for download
Use chunking for upload (potentially parallelize chunk uploads to maximize user bandwidth) and syncing

Utilize compression to speed up uploads/downloads.
Depends on file type and size and network conditions.
Write logic to decide if you compress a file or not before upload/download.

### How do you Ensure File Security
1. Encryption in Transit - HTTPS
2. Encryption at Rest - Feature of S3
3. Access Control with shareList

Signed URLs only work for a short period and for the requested user


