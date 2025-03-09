# Design Leetcode

## Requirements
### Functional Requirements
1. Users should be able to view a list of problems.
2. Users should be able to view a problem and code a solution in multiple languages.
3. Users should be able to submit a solution and get instant feedback (5 seconds).
4. Users should be able to view a live leaderboard for competitions.

### Non-functional Requirements
1. The system should prioritize availability over consistency.
2. The system should support isolation and security when running user code.
3. The system should return submission results within 5 seconds.
4. The system should be able to scale and support a competition with 100,000 users.

## Core Entities
1. Problem
2. Submission
3. Leaderboard
4. User

## API
RESTful API

```
GET /v1/problems?page={number}&limit={number} -> Partial<Problem>[]

GET /v1/problems/:problemId?language={language} -> Problem

POST /v1/problems/:problemId
body: {
    "code": string,
    "language": string
} -> Submission

GET /v1/leaderboards/:competitionId?page={number}&limit={number} -> Leaderboard
```
## High Level Design
![High Level Design of Leetcode](/leetcode/high-level-design.png)


## Deep Dive
### How will the system support isolation and security when running code?
1. Mount code as read-only; output to a temp directory to delete shortly after
2. CPU and Memory Bound container
3. Explicit timeout such as 5 seconds.
4. Disable network access in container
5. No System Calls

### How would you make fetching the leaderboard more efficient?
#### Redis Sorted Set with Periodic Polling
When a submission is processed, both DB and Redis sorted set are updated for that particular competition.
Clients can poll the server every 5 seconds for leaderboard updates, and the server returns the top N users in the sorted set based on time solved/score.

WebSockets is overkill.
Periodic polling is sufficient for LeetCode.
We might even change the frequency of the polling if the competition just started versus is about to end.

### How would the system scale to support 100K DAUs?

Traffic spikes from competitions at certain times in day. We can use ECS to manage our containers and auto scale reach runtime.

An added buffer is use a queue like SQS. Potentially overengineering, but gives us retry capabilities if a container crashes.

### How would the system handle test cases?
Can't really write test cases for each runtime environment. Impossible to scale.

Standardize a way to serialize test case inputs and outputs.