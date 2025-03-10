# Design Patient Management System

## Requirements
### Functional Requirements
1. Patients and providers will be able to schedule appointments.
2. Providers will be able to create prescriptions for patients.
3. Only authorized providers will be able to view patient records.

---
1. Providers can view a list of available prescriptions.
2. Providers can share patient records.
3. Providers can get notified when a patient picks up their prescription.
4. Patients can view providers by nearest proximity.

### Non-functional Requirements
1. HIPAA and GDPR compliant.
2. Low latency system (< 500ms).
3. Replication, failover, and backup mechanisms.
4. Appointments cannot be double booked. Strong consistency for appointment scheduling.
5. Manage 100M patient records. High availability here.

## Core Entities
1. Patient
2. Provider
3. Appointments
4. Patient Records
5. Prescriptions

## API
RESTful API
```
POST /v1/appointments -> Appointment
Scheduler info (providerId or patientId is in JWT token)
body: {
    appointmentTime
    ...
}
POST /v1/prescriptions -> Prescription
body: {
    prescription
    ...
}

GET /v1/records -> Partial<Record>[]
GET /v1/records/:recordId -> Record
POST /v1/records -> Record
providerId and patientId is in JWT token
body: {
    diagnosis
}
```

## High Level Design
![HLD](/healthcare/patient-management-system/hld.png)

## Deep Dive
### How can we avoid double booking providers with patients?
1. Distributed lock with TTL using Redis

### How can we make sure viewing records has low latency? There can be 100M records.
Combination of DB indexing and caching.
Index on patientId.
Maybe even full-text indexes if we want to search for records based on diagnoses for research purposes for providers/researchers. Can use ElasticSearch for this.

Cache records in Redis. LFU eviction policy with TTL.

Keep in mind that we have to anonymize/de-identify these records when sharing with other providers/researchers if they don't have jurisdiction over these patient records.

### HIPAA/GDPR Compliance
1. HTTPS for encryption in transit
2. API Gateway for OAuth 2.0 with scope; providers can only view certain patients and their records
3. Data encrypted at rest; MongoDB is HIPAA compliant
4. We have redundancy, failover, and backup databases with sharding

### How can we notify patients and providers about appointments and prescriptions?
Separate notification service. Event driven architecture with queues.

![FinalDesign](/healthcare/patient-management-system/finaldesign.png)