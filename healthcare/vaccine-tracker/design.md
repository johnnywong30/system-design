# Design Vaccine Tracker

## Problem
SD: realtime analytics for COVID immunizations. I want to track the immunization status of individuals. Different vaccine brands. Some require 2 inoculations. People can get different vaccines at different clinics with different providers. Need to track individual vaccination status and also be able to anytime answer how many people are fully vaccinated, partially vaccinated, not vaccinated. To resolve the patient identity across the different patients tokenization can be used (a core datavant service).

## Requirements
### Functional Requirements
1. Users should be able to determine the immunization status of an individual at any time.
2. Users should be able to update the immunization status of individuals when they get vaccinated. Some vaccine types might require multiple inoculations.
3. Users should be able to determine how many people are fully vaccinated, partially vaccinated, and not vaccinated.
4. Patients should be able to get different vaccines at different clinics with different providers.

### Non-functional Requirements
1. The system should have 99.99% availability.
2. The system should be able to handle bursts of traffic (i.e. during the winter).
3. The system should be HIPAA compliant.
4. The system should be able to encrypt PHI and identify the same patient across different providers/clinics despite clerical issues.
5. The system should have near real-time response times and low latency.

## Core Entities
1. Provider
2. Vaccine
3. Clinic
4. Patient
5. Vaccination

## API
RESTful API
```
GET /patients -> Partial<Patient>[]
GET /patients/:patientId -> Patient

POST /patients -> Patient
body: {
    phi
}

POST /vaccines -> Vaccination
body: {
    vaccine,
    provider,
    clinic,
    phi
}
```

## High Level Design
![hld](/healthcare/vaccine-tracker/hld.png)

## Deep Dive
Initial Deep Dive
![dd](/healthcare/vaccine-tracker/dd.png)

Deep Dive Practice on Text:

To process vaccinations and query analytics in real time, we can have these two approaches.

Batch Process
- Separate DB (handles heavy writes) holding raw vaccination events
- CRON job runs to aggregate the vaccination events and provide the stats for fully vax, partial vax, and not vax
- CRON job pushes that aggregation to database for analytics service to query the for the most recent aggregation when a analyst wants to know the stats

- Not real time
- Processing is delayed to CRON schedule

Real Time Process
- Utilize the same DB
- Vax service produces event to a stream like Kafka
- Stream consumer like Spark Streaming can read from Kafka and aggregate vax analytics in real time (real time experience depends on aggregation window)
- Push updated aggregation to DB in real time
