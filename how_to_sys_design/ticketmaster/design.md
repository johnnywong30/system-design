# Design TicketMaster

## Requirements

### Functional Requirements
1. Users should be able to view events
2. Users should be able to book tickets to events
3. Users should be able to search for events
---
1. Users should be able to view their booked events.
2. Admins or event coordinators should be able to add events.
3. Popular events should have dynamic pricing.

### Non-functional Requirements
- CAP Theorem
- Read or Write Heavy
- Query access pattern
  
1. The system should prioritize availability for searching and viewing events, but should prioritize consistency for booking events.
2. The system should be scalable and handle high throughput in the form of popular events. 10M users for 1 event.
3. The system should have low latency search (< 500ms)
4. The system is read heavy, and should support high read throughput (100:1).
---
1. The system should protect user data and adhere to GDPR.
2. The system should be fault tolerant.
3. The system should provide secure transactions for purchase.
4. The system should be well tested and easy to deploy (CI/CD pipelines).
5. The system should have regular backups.


## Core Entities
1. Event
2. User
3. Performer
4. Venue
5. Ticket
6. Booking
   
## API
RESTful API
```
GET /v1/events?page={number}&limit={number}&search={string}&start={startDate}&end={endDate} -> Partial<Event>[]
GET /v1/events/:eventId -> Event

POST /v1/bookings/:eventId
body: {
    ticketIds: string[],
    payment: ...
}
```
## High Level Design
![HLD](/how_to_sys_design/ticketmaster/hld.png)

## Deep Dive
### How do we improve the booking experience by reserving tickets?
Use distributed locks with TTL in a Redis cache. Use the ticketId as the key with a TTL. If the user purchases the ticket before lock expires, then update ticket status. Lock manually released when purchased.
If TTL expires before purchase completed, Redis automatically releases lock and then ticket is up for grabs.

### How is the view API going to scale to support 10s of millions of concurrent requests during popular events?
Caching, Load balancing, and Horizontal Scaling

### How will the system ensure a good user experience during high-demand events with millions simultaneously booking tickets?

Virtual waiting queue with websocket.
Control flow of users accessing booking page of a popular event. Dequeue users after certain time or actions so they can proceed to book tickets.
Pushing updates in real time improves waiting experience

### How can you improve search to ensure we meet our low latency requirements?

Full text search index in DB or ElasticSearch for Full Text Search Engine

### How can you speed up frequently repeated search queries and reduce load on our search infrastructure?

Good solution: Caching with Redis

Great solution: Query Result Caching with Elasticsearch and Edge Caching via CDNs