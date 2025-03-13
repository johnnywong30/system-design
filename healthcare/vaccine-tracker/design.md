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
![dd](/healthcare/vaccine-tracker/dd.png)