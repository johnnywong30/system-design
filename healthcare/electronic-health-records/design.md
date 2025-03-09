# Design Electronic Health Record System

## Requirements

### Functional Requirements
1. Users should be able to upload and view PHI and records they can access.
2. Users should be able to integrate and share PHI and records with providers.
3. Users should be able to restrict access to sensitive PHI and records.
4. Users should be able to edit and delete PHI and records.

### Non-functional Requirements
1. The system should be secure and compliant with HIPAA.
2. The system should be able to provide an audit trail of logged events.
3. The system should be able to manage large datasets and high traffic.
4. The system should have 99.99% availability with failover, redundancy, and backup mechanisms.

## Core Entities
1. Patient
2. Provider
3. Record

## API
RESTful API with Event Driven Architecture
The interface may evolve over time over the discussion as we aim to fit certain requirements at larger scales.
```
GET /v1/patients/:patientId -> Patient
GET /v1/patients?page={number}&limit={number} -> Partial<Patient>[]

POST /v1/patients
body: {
    name: string,
    dateOfBirth: Date,
    gender: string,
}

PUT /v1/patients/:patientId
body: {
    name?: string,
    dateOfBirth?: Date,
    gender?: string,
    ...
}

POST /v1/patients/:patientId
body: {
    providerId: string,
    diagnosis: string,
    ...
}

GET /v1/patients/:patientId/records -> Record[]
GET /v1/patients/:patientId/records/:recordId -> Record

GET /v1/providers?/page={number}&limit={number} -> Partial<Provider>[]
GET /v1/providers/:providerId -> Provider
```

## High Level Design
![Design](/healthcare/electronic-health-records/design.png)

## Deep Dive

### HIPAA
1. Encryption - HTTPS for transport; MongoDB encrypted at rest
2. RBAC - Only users with correct scope and roles will be able to access certain PHI
3. Patient information is encrypted with a per patient encryption key. That way, if the database is breached, each patients' data will still be secure.
4. Sessions have to be timed-out over time.

### Audit Trail
1. Kafka allows us to stream each transaction (creation/update/delete) to our ingestion service. This allows us to audit these transactions, rollback changes, and/or reconstruct the state of a patient's records to a previous state by replaying events.

### Large Datasets and High Traffic
1. Utilize a cache like Redis to handle high traffic and provide low latency responses for frequently requested data.
   1. Concerns here include cache size and eviction policies.
   2. It might take time to warm up the cache with popular records, but eventually the system will have higher performance with more frequently used records if we utilize a LFU policy.
2. Kafka lets us queue up writes to our database through an event driven architecture.

### Failover, Redundancy, and Backups
1. Database sharding
2. Replica sets to failover to.
3. Periodic backup databases with CRON jobs.
