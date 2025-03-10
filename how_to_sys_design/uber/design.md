# Design Uber

## Requirements
### Functional Requirements
1. Riders should be able to input a start location and destination and get an output fare estimate.
2. Riders should be able to request a ride based on an estimate fare.
3. Upon request, riders should be matched with a nearby available driver.
4. Drivers should be able to accept or reject a request and navigate to pickup and drop-off locations.
---
1. Riders should be able to rate and tip their drivers.
2. Drivers should be able to rate their riders.
3. Riders should be able to schedule rides in advance.
4. Riders should be able to request different ride categories.

### Non-functional Requirements
1. The system should prioritize low latency matching (< 1 minute to match or fail)
2. The system should prioritize strong consistency to prevent over booking drivers.
3. The system should be able to handle high bursts of throughput (100k requests from same location).
---
1. Security and privacy of user/driver data (GDPR)
2. Resilient to failures and have redundancy and failover mechanisms.
3. Monitoring, logging, and alerting to identify and resolve issues quickly.
4. Faciliate easy updates and maintenance with CI/CD pipelines.

## Core Entities
1. Rider
2. Driver
3. Fare
4. Ride
5. Location

## API
RESTful API
```
POST /v1/fares -> Fare
body: {
    pickup,
    destination
}

POST /v1/rides -> Ride
body: {
    fareId
}

matching process happens in the backend, no need for endpoint

POST /v1/drivers/location -> Success/Error
body: {
    lat,
    lng
}

PATCH v1/rides/:rideId -> Ride
body: {
    accept/reject
}
```

## High Level Design
![HLD](/how_to_sys_design/uber/hld.png)

## Deep Dive
### How do we handle frequent driver location updates and efficient proximity searches on location data?
10M drivers, updates every 5 seconds; 2M writes/second
Query efficiency is poor if we read from a disk based Database

Great Solution: Use In-Memory Datastore Redis
1. `GEOADD` and `GEOSEARCH`
2. Uses geohashing to encode latitude and longitude coordinates into a single string
3. Handle high throughput and have low latency finding nearby drivers
4. TTL for driver locations 
5. Redis Sentinal for failover and replication
6. 5 seconds to recover so Redis crashing is not much of an issue as more updates will come in

### How can we manage system overload from frequent driver location updates while ensuring location accuracy?
Adaptive location update intervals based on the client.
Use ondevice sensors and algorithms based on speed, direction of travel, proximity to nearby ride requests, and driver status.

### How do we prevent multiple ride requests from being sent to the same driver simultaneously?
Distributed Lock with TTL (Redis)
Lock with TTL of 10 seconds when a ride request is sent to a driver. Driver acquires the lock and no other ride request can request that driver.

### How can we ensure no ride requests are dropped during peak demand periods?

Queue up Ride Requests with SQS or Kafka
Handles scaling and failover automatically as managed services
Can utilize a priority queue

### How can you further scale the system to reduce latency and improve throughput?
Geosharding with Read Replicas
![final_design](/how_to_sys_design/uber/final_design.png)