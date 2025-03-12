# Design Medical Imaging Storage System

## Requirements

### Functional Requirements
1. Upload extremely large medical images
2. Download extremely large medical images
3. View extremely large medical images quickly
4. Only authorized users should be able to view certain medical images

### Non-functional Requirements
1. The system should be able to store large volumes of medical images
2. The system should be able to compress medical images without losing image quality
3. The system should have low latency (< 200ms) when retrieving medical images
4. The system should be highly available and have eventual consistency
5. The system meets DICOM standards

## Core Entities
1. Medical Image
2. Medical Image Metadata
3. User

## API
RESTful API
```
GET /v1/images/:imageId -> MedicalImageMetadata
POST /v1/images -> presignedURL
```

## High Level Design
![HLD](/healthcare/medical-imaging-storage-system/design.png)


## Deep Dive
