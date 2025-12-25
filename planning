Hospital Management System - Phase 1 Architecture
=======

| Status        | Build               |
|---------------|---------------------|
| Category      | System Design       |
| Team          | Healthcare Platform |
| Author(s)     | Himanshu Yadav      |
| Created Date  | January 2025        |
| Modified Date | January 2025        |

**Abstract**

This document provides a comprehensive technical architecture for Phase 1 of the Hospital Management System, focusing on multi-tenant hospital registration, user authentication and authorization, doctor management, and appointment slot allocation. The system implements a microservices-lite architecture using Django for identity and discovery services, and Spring Boot for transactional booking and slot management services. Phase 1 establishes the foundation for a scalable healthcare platform that enables hospitals to register, manage their doctors and departments, and allows patients to discover and book appointments with available doctors.

## Terminology

| Term                 | Description                                                                                                   |
|----------------------|---------------------------------------------------------------------------------------------------------------|
| Hospital             | A tenant organization that provides healthcare services and manages doctors, departments, and appointment slots |
| Doctor               | A healthcare provider associated with a hospital who offers consultation services and has available time slots |
| Patient              | An end user who searches for doctors and books appointments                                                      |
| Slot                 | A time interval (e.g., 30 minutes) during which a doctor is available for consultation                            |
| Appointment          | A confirmed booking between a patient and doctor for a specific time slot                                         |
| Multi-tenant         | Architecture pattern where multiple hospitals operate independently within the same system instance             |
| RBAC                 | Role-Based Access Control - authorization mechanism based on user roles (SUPER_ADMIN, HOSPITAL_ADMIN, DOCTOR, PATIENT) |
| JWT                  | JSON Web Token - stateless authentication mechanism used for cross-service authentication                        |
| Optimistic Locking   | Concurrency control mechanism using version fields to prevent race conditions during slot booking                 |

## Introduction

The Hospital Management System Phase 1 addresses the core requirements of establishing a multi-tenant healthcare platform where:

1. **Multiple hospitals** can register and manage their organizational structure (departments, doctors, services)
2. **Doctors** can be onboarded by hospitals and manage their availability through time slots
3. **Patients** can discover doctors through search and filtering capabilities
4. **Appointment booking** system ensures proper slot allocation without double-booking conflicts

This phase establishes the foundational architecture using Django for identity management and Spring Boot for transactional operations, setting the stage for future phases that will include payment processing and Electronic Health Records (EHR) management.

## High-Level Architecture

### System Overview

The system follows a **microservices-lite** architecture pattern, separating concerns between identity/discovery services and transactional/booking services:

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend (React)                        │
│                    Single Source of Truth for UI                │
└────────────┬───────────────────────────────┬───────────────────┘
             │                               │
             │                               │
    ┌────────▼────────┐            ┌────────▼─────────┐
    │  Django Service │            │ Spring Boot       │
    │  "The Director" │            │ "The Vault"       │
    │                 │            │                   │
    │ • User Auth     │            │ • Slot Management │
    │ • Profiles      │            │ • Appointments    │
    │ • Discovery     │            │ • Concurrency     │
    │ • RBAC          │            │   Control         │
    └────────┬────────┘            └────────┬──────────┘
             │                             │
             │                             │
    ┌────────▼────────┐            ┌────────▼─────────┐
    │   PostgreSQL    │            │   PostgreSQL      │
    │  db_identity    │            │   db_clinical     │
    │                 │            │                   │
    │ • Users         │            │ • TimeSlots       │
    │ • Hospitals     │            │ • Appointments    │
    │ • Doctors       │            │                   │
    │ • Patients      │            │                   │
    └─────────────────┘            └───────────────────┘
                                             │
                                    ┌────────▼─────────┐
                                    │      Redis       │
                                    │  Slot Locking    │
                                    └──────────────────┘
```

### Architecture Principles

1. **Service Separation**: Django handles identity and discovery (read-heavy), Spring Boot handles transactions (write-heavy with concurrency requirements)
2. **Shared Authentication**: JWT tokens issued by Django are validated by Spring Boot for stateless cross-service authentication
3. **Database Isolation**: Separate schemas for identity (`db_identity`) and clinical/transactional data (`db_clinical`)
4. **Concurrency Control**: Redis-based distributed locking combined with optimistic locking in Spring Boot for slot allocation
5. **Multi-tenancy**: Hospital-level data isolation through foreign key relationships and query filtering

### Technology Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| Frontend | React | Modern, component-based UI framework |
| Identity Service | Django + Django REST Framework | Rapid development, built-in admin, ORM for complex queries |
| Booking Service | Spring Boot | Strong transaction support, JPA for data consistency, Redis integration |
| Identity Database | PostgreSQL | ACID compliance, complex joins for search/filter |
| Clinical Database | PostgreSQL | Transactional integrity for bookings |
| Caching/Locking | Redis | Distributed locking for concurrent slot bookings |
| Authentication | JWT (Django) | Stateless, cross-service compatible |

## Low-Level Architecture

### Django Service Architecture

#### Service Layer Structure

```
django-service/
├── identity/
│   ├── models.py          # User, Hospital, Department, DoctorProfile, PatientProfile
│   ├── serializers.py     # DRF serializers for API responses
│   ├── views.py           # ViewSets for REST endpoints
│   ├── permissions.py     # Custom permission classes (RBAC)
│   └── filters.py         # Django-filter for search/filtering
├── authentication/
│   ├── backends.py        # JWT authentication backend
│   ├── tokens.py           # JWT token generation/validation
│   └── middleware.py      # Request authentication middleware
└── core/
    ├── settings.py         # Django configuration
    └── urls.py            # URL routing
```

#### Key Design Patterns

1. **Model Inheritance**: `AbstractUser` base for User, with OneToOne relationships to `DoctorProfile` and `PatientProfile`
2. **Multi-tenancy**: Hospital FK on DoctorProfile enables hospital-scoped queries
3. **Search/Filter**: Django-filter with Q objects for complex doctor discovery queries
4. **Permissions**: Django REST Framework permission classes enforcing RBAC

### Spring Boot Service Architecture

#### Service Layer Structure

```
spring-boot-service/
├── booking/
│   ├── entity/
│   │   ├── TimeSlot.java      # JPA entity with version field
│   │   └── Appointment.java   # JPA entity with status enum
│   ├── repository/
│   │   ├── TimeSlotRepository.java
│   │   └── AppointmentRepository.java
│   ├── service/
│   │   ├── SlotService.java           # Slot availability logic
│   │   ├── BookingService.java        # Booking orchestration
│   │   └── ConcurrencyService.java    # Redis locking logic
│   └── controller/
│       └── BookingController.java     # REST endpoints
├── security/
│   ├── JwtAuthenticationFilter.java    # JWT validation
│   └── SecurityConfig.java            # Spring Security config
└── config/
    ├── RedisConfig.java               # Redis connection
    └── JpaConfig.java                 # JPA/Hibernate config
```

#### Key Design Patterns

1. **Optimistic Locking**: `@Version` annotation on TimeSlot entity prevents concurrent modifications
2. **Distributed Locking**: Redis SETNX for temporary slot reservation during booking flow
3. **State Machine**: Appointment status transitions (PENDING → CONFIRMED → COMPLETED/CANCELLED)
4. **Transaction Management**: `@Transactional` for atomic booking operations

### Integration Architecture

#### Authentication Flow

```
┌─────────┐         ┌──────────┐         ┌──────────────┐
│ Patient │         │  Django  │         │ Spring Boot │
│ (React) │         │  Service │         │   Service   │
└────┬────┘         └────┬─────┘         └──────┬──────┘
     │                   │                      │
     │ POST /auth/login  │                      │
     │──────────────────>│                      │
     │                   │                      │
     │ JWT (access+refresh)                      │
     │<──────────────────│                      │
     │                   │                      │
     │ POST /bookings/initiate                    │
     │ Authorization: Bearer <JWT>                │
     │──────────────────────────────────────────>│
     │                   │                      │
     │                   │  Validate JWT        │
     │                   │  Extract user_id     │
     │                   │<─────────────────────│
     │                   │                      │
     │ Booking Response  │                      │
     │<──────────────────────────────────────────│
```

#### Slot Allocation Flow

```
┌─────────┐    ┌──────────────┐    ┌──────┐    ┌──────────┐
│ Patient │    │ Spring Boot │    │Redis │    │PostgreSQL│
└────┬────┘    └──────┬───────┘    └──┬───┘    └────┬─────┘
     │                │               │             │
     │ GET /slots     │               │             │
     │───────────────>│               │             │
     │                │ Query available slots       │
     │                │───────────────────────────>│
     │                │<───────────────────────────│
     │ Available slots│               │             │
     │<───────────────│               │             │
     │                │               │             │
     │ POST /bookings/initiate                      │
     │ {slotId, doctorId}                          │
     │───────────────>│               │             │
     │                │ Try acquire lock            │
     │                │ SETNX slot:{slotId}         │
     │                │───────────────────────────>│
     │                │<───────────────────────────│
     │                │ Lock acquired?              │
     │                │                             │
     │                │ @Transactional              │
     │                │ 1. SELECT FOR UPDATE        │
     │                │───────────────────────────>│
     │                │<───────────────────────────│
     │                │ 2. Check version & isBooked│
     │                │ 3. Create Appointment       │
     │                │───────────────────────────>│
     │                │<───────────────────────────│
     │                │ 4. Update TimeSlot         │
     │                │───────────────────────────>│
     │                │<───────────────────────────│
     │                │ Release lock                │
     │                │ DEL slot:{slotId}          │
     │                │───────────────────────────>│
     │                │<───────────────────────────│
     │ Booking created│                             │
     │<───────────────│                             │
```

## Services

| Service | Description | Metrics | Dependencies<sup>3</sup> |
|---------|-----------|---------|---------------------------|
| `identity-service` (Django) | Manages user authentication, hospital registration, doctor/patient profiles, and discovery/search functionality. Handles RBAC and multi-tenant data isolation. | Requests/sec<sup>1</sup>: 100<br/>Instances<sup>2</sup>: 2 | <ul><li>`PostgreSQL (db_identity)`: Primary data store for users, hospitals, doctors, patients</li><li>`Redis`: Optional caching for frequently accessed doctor profiles</li></ul> |
| `booking-service` (Spring Boot) | Manages time slot availability, appointment booking with concurrency control, and appointment lifecycle management. Ensures data consistency for transactional operations. | Requests/sec<sup>1</sup>: 50<br/>Instances<sup>2</sup>: 2 | <ul><li>`PostgreSQL (db_clinical)`: Stores time slots and appointments</li><li>`Redis`: Distributed locking for concurrent slot bookings</li><li>`identity-service`: Validates JWT tokens for authentication</li></ul> |

> <sup>1</sup> Initial traffic estimates based on expected user base. Metrics to be refined based on production data.

> <sup>2</sup> Minimum number of instances for the service, subject to autoscaling based on load.

> <sup>3</sup> A list of internal or external services that a service communicates with to consume data


**Load Testing**: Required before production deployment to validate concurrency handling, especially for slot booking operations under high load.

## Datasources

| Datastore | Description | Type | Performance |
|-----------|------------|------|-------------|
| `db_identity` | Stores user accounts, hospital information, doctor profiles, patient profiles, and reviews. Supports complex search and filtering queries. | PostgreSQL | <ul><li>Indexes on: `User.email`, `DoctorProfile.hospital_id`, `DoctorProfile.specialization`, `Hospital.name`</li><li>Composite index on `(hospital_id, specialization)` for doctor search</li><li>Read replicas for scaling search queries</li></ul> |
| `db_clinical` | Stores time slots, appointments, and booking-related transactional data. Optimized for write-heavy operations with concurrency control. | PostgreSQL | <ul><li>Indexes on: `TimeSlot.doctorId`, `TimeSlot.startTime`, `Appointment.patientId`, `Appointment.doctorId`</li><li>Composite index on `(doctorId, startTime, isBooked)` for slot queries</li><li>Row-level locking for concurrent booking operations</li></ul> |
| `Redis` | Provides distributed locking mechanism for slot reservation during booking flow. Prevents race conditions in multi-instance deployments. | Redis | <ul><li>TTL-based locks (30 seconds) for slot reservations</li><li>Key pattern: `slot:{slotId}:lock`</li><li>Single Redis instance sufficient for Phase 1, cluster for scale</li></ul> |

### Entity Relationship Models

#### Django Models (db_identity)

```mermaid
erDiagram
    User ||--o| DoctorProfile : "has"
    User ||--o| PatientProfile : "has"
    Hospital ||--o{ Department : "has"
    Hospital ||--o{ DoctorProfile : "employs"
    Department ||--o{ DoctorProfile : "categorizes"
    DoctorProfile ||--o{ Review : "receives"
    PatientProfile ||--o{ Review : "writes"
    
    User {
        uuid id PK
        string email UK
        string password_hash
        enum role
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }
    
    Hospital {
        uuid id PK
        string name
        text address
        string registration_number UK
        boolean is_verified
        timestamp created_at
    }
    
    Department {
        uuid id PK
        uuid hospital_id FK
        string name
    }
    
    DoctorProfile {
        uuid id PK
        uuid user_id FK "OneToOne"
        uuid hospital_id FK
        uuid department_id FK
        string specialization
        integer experience_years
        decimal consultation_fee
        text bio
        decimal rating_avg
    }
    
    PatientProfile {
        uuid id PK
        uuid user_id FK "OneToOne"
        date date_of_birth
        enum gender
        string blood_group
        json emergency_contact
    }
    
    Review {
        uuid id PK
        uuid doctor_id FK
        uuid patient_id FK
        integer rating "1-5"
        text comment
        timestamp created_at
    }
```

#### Spring Boot Entities (db_clinical)

```mermaid
erDiagram
    TimeSlot ||--o| Appointment : "books"
    
    TimeSlot {
        uuid id PK
        uuid doctorId FK "References Django User.id"
        timestamp startTime
        timestamp endTime
        boolean isBooked
        integer version "Optimistic locking"
        timestamp created_at
    }
    
    Appointment {
        uuid id PK
        uuid patientId FK "References Django User.id"
        uuid doctorId FK "References Django User.id"
        uuid timeSlotId FK
        enum status "PENDING, CONFIRMED, CANCELLED, COMPLETED"
        timestamp created_at
        timestamp updated_at
    }
```

### Schemas

#### PostgreSQL: db_identity

##### Entities

**User**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier for user | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| email | VARCHAR(255) | User email address | UNIQUE, NOT NULL |
| password_hash | VARCHAR(255) | Bcrypt hashed password | NOT NULL |
| role | ENUM | User role: SUPER_ADMIN, HOSPITAL_ADMIN, DOCTOR, PATIENT | NOT NULL |
| is_active | BOOLEAN | Account activation status | DEFAULT true |
| created_at | TIMESTAMP | Account creation timestamp | DEFAULT NOW() |
| updated_at | TIMESTAMP | Last update timestamp | DEFAULT NOW() |

**Hospital**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier for hospital | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| name | VARCHAR(255) | Hospital name | NOT NULL |
| address | TEXT | Physical address | |
| registration_number | VARCHAR(100) | Government registration number | UNIQUE, NOT NULL |
| is_verified | BOOLEAN | Verification status by super admin | DEFAULT false |
| created_at | TIMESTAMP | Creation timestamp | DEFAULT NOW() |

**Department**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| hospital_id | UUID | Reference to hospital | FOREIGN KEY, NOT NULL |
| name | VARCHAR(255) | Department name (e.g., Cardiology) | NOT NULL |

**DoctorProfile**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| user_id | UUID | Reference to User (OneToOne) | FOREIGN KEY, UNIQUE, NOT NULL |
| hospital_id | UUID | Reference to Hospital | FOREIGN KEY, NOT NULL |
| department_id | UUID | Reference to Department | FOREIGN KEY, NOT NULL |
| specialization | VARCHAR(255) | Medical specialization | NOT NULL |
| experience_years | INTEGER | Years of experience | DEFAULT 0 |
| consultation_fee | DECIMAL(10,2) | Fee per consultation | NOT NULL |
| bio | TEXT | Doctor biography | |
| rating_avg | DECIMAL(3,2) | Average rating (1.00-5.00) | DEFAULT 0.00 |

**PatientProfile**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| user_id | UUID | Reference to User (OneToOne) | FOREIGN KEY, UNIQUE, NOT NULL |
| date_of_birth | DATE | Patient date of birth | |
| gender | ENUM | MALE, FEMALE, OTHER | |
| blood_group | VARCHAR(10) | Blood group (e.g., A+, O-) | |
| emergency_contact | JSONB | Emergency contact information | |

**Review**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| doctor_id | UUID | Reference to DoctorProfile | FOREIGN KEY, NOT NULL |
| patient_id | UUID | Reference to PatientProfile | FOREIGN KEY, NOT NULL |
| rating | INTEGER | Rating (1-5) | CHECK (rating >= 1 AND rating <= 5) |
| comment | TEXT | Review comment | |
| created_at | TIMESTAMP | Review creation timestamp | DEFAULT NOW() |

**Key Information**

| Type | Partition Key | Sort Key | Use-case |
|------|--------------|----------|----------|
| Table | `{id}` | - | Primary access by ID |
| Index | `{email}` | - | User lookup by email |
| Index | `{hospital_id}` | - | Hospital-scoped queries |
| Index | `{hospital_id, specialization}` | - | Doctor search by hospital and specialty |
| Index | `{doctor_id}` | `{created_at}` | Reviews for a doctor |

#### PostgreSQL: db_clinical

##### Entities

**TimeSlot**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| doctor_id | UUID | Reference to Django User.id | NOT NULL, INDEXED |
| start_time | TIMESTAMP | Slot start time | NOT NULL, INDEXED |
| end_time | TIMESTAMP | Slot end time | NOT NULL |
| is_booked | BOOLEAN | Booking status | DEFAULT false, INDEXED |
| version | INTEGER | Optimistic locking version | DEFAULT 0, NOT NULL |
| created_at | TIMESTAMP | Creation timestamp | DEFAULT NOW() |

**Appointment**

| Name | Type | Description | Constraints |
|------|------|-------------|-------------|
| id | UUID | Unique identifier | PRIMARY KEY, DEFAULT uuid_generate_v4() |
| patient_id | UUID | Reference to Django User.id | NOT NULL, INDEXED |
| doctor_id | UUID | Reference to Django User.id | NOT NULL, INDEXED |
| time_slot_id | UUID | Reference to TimeSlot | FOREIGN KEY, NOT NULL, UNIQUE |
| status | ENUM | PENDING, CONFIRMED, CANCELLED, COMPLETED | DEFAULT 'PENDING', NOT NULL |
| created_at | TIMESTAMP | Creation timestamp | DEFAULT NOW() |
| updated_at | TIMESTAMP | Last update timestamp | DEFAULT NOW() |

**Key Information**

| Type | Partition Key | Sort Key | Use-case |
|------|--------------|----------|----------|
| Table | `{id}` | - | Primary access by ID |
| Index | `{doctor_id, start_time, is_booked}` | - | Query available slots for doctor |
| Index | `{patient_id}` | `{created_at}` | Patient appointment history |
| Index | `{doctor_id}` | `{created_at}` | Doctor appointment schedule |

### Data Classification Policy

- [ ] DSL 1
- [ ] DSL 2
- [ ] DSL 3
- [ ] DSL 4
- [x] DSL 5 - Patient health information (PII) stored in PatientProfile
- [ ] DSL 6

**Note**: Patient profiles contain PII (date of birth, emergency contacts). Doctor profiles contain professional information. Appointment data links patients and doctors. Data handling must comply with healthcare data protection regulations (HIPAA considerations for future phases).

### PCI Scope

- [ ] This is in PCI scope

**Note**: Phase 1 does not handle payment processing. PCI scope assessment required for Phase 2 when payment integration is added.

## Implementation

### APIs and Models

#### Django Service APIs

##### Authentication Endpoints

**POST** `/identity-service/v1/auth/register/`

Register a new user (Patient or Doctor).

**Request Body:**
```json
{
  "email": "patient@example.com",
  "password": "SecurePassword123!",
  "role": "PATIENT",
  "first_name": "John",
  "last_name": "Doe"
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "email": "patient@example.com",
  "role": "PATIENT",
  "is_active": true
}
```

**POST** `/identity-service/v1/auth/login/`

Authenticate user and return JWT tokens.

**Request Body:**
```json
{
  "email": "patient@example.com",
  "password": "SecurePassword123!"
}
```

**Response:** `200 OK`
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

**POST** `/identity-service/v1/auth/refresh/`

Refresh access token using refresh token.

**Request Body:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** `200 OK`
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

##### Hospital Management Endpoints

**GET** `/identity-service/v1/hospitals/`

List all hospitals with search and filter capabilities.

**Query Parameters:**
- `search` (string, optional): Search by hospital name
- `is_verified` (boolean, optional): Filter by verification status
- `page` (integer, optional): Page number for pagination
- `page_size` (integer, optional): Items per page (default: 20)

**Response:** `200 OK`
```json
{
  "count": 50,
  "next": "http://api.example.com/hospitals/?page=2",
  "previous": null,
  "results": [
    {
      "id": "uuid",
      "name": "City General Hospital",
      "address": "123 Main St, City, State",
      "registration_number": "HOS-2024-001",
      "is_verified": true,
      "created_at": "2024-01-15T10:00:00Z"
    }
  ]
}
```

**POST** `/identity-service/v1/hospitals/onboard/`

Create a new hospital (Super Admin only).

**Request Body:**
```json
{
  "name": "City General Hospital",
  "address": "123 Main St, City, State",
  "registration_number": "HOS-2024-001"
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "name": "City General Hospital",
  "address": "123 Main St, City, State",
  "registration_number": "HOS-2024-001",
  "is_verified": false,
  "created_at": "2024-01-15T10:00:00Z"
}
```

**PATCH** `/identity-service/v1/hospitals/{hospital_id}/verify/`

Verify a hospital (Super Admin only).

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "is_verified": true,
  "updated_at": "2024-01-15T11:00:00Z"
}
```

##### Doctor Discovery Endpoints

**GET** `/identity-service/v1/doctors/`

Global search for doctors with advanced filtering.

**Query Parameters:**
- `specialization` (string, optional): Filter by medical specialization
- `hospital_id` (uuid, optional): Filter by hospital
- `department_id` (uuid, optional): Filter by department
- `min_fee` (decimal, optional): Minimum consultation fee
- `max_fee` (decimal, optional): Maximum consultation fee
- `min_rating` (decimal, optional): Minimum average rating
- `location` (string, optional): Search by hospital location
- `search` (string, optional): Full-text search across name, specialization, bio
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page (default: 20)

**Response:** `200 OK`
```json
{
  "count": 150,
  "next": "http://api.example.com/doctors/?page=2",
  "previous": null,
  "results": [
    {
      "id": "uuid",
      "user": {
        "id": "uuid",
        "email": "doctor@hospital.com",
        "first_name": "Dr. Jane",
        "last_name": "Smith"
      },
      "hospital": {
        "id": "uuid",
        "name": "City General Hospital"
      },
      "department": {
        "id": "uuid",
        "name": "Cardiology"
      },
      "specialization": "Cardiologist",
      "experience_years": 10,
      "consultation_fee": 150.00,
      "bio": "Experienced cardiologist...",
      "rating_avg": 4.5,
      "review_count": 25
    }
  ]
}
```

**GET** `/identity-service/v1/doctors/{doctor_id}/`

Get detailed doctor profile including reviews.

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "user": {
    "id": "uuid",
    "email": "doctor@hospital.com",
    "first_name": "Dr. Jane",
    "last_name": "Smith"
  },
  "hospital": {
    "id": "uuid",
    "name": "City General Hospital",
    "address": "123 Main St"
  },
  "department": {
    "id": "uuid",
    "name": "Cardiology"
  },
  "specialization": "Cardiologist",
  "experience_years": 10,
  "consultation_fee": 150.00,
  "bio": "Experienced cardiologist...",
  "rating_avg": 4.5,
  "reviews": [
    {
      "id": "uuid",
      "patient": {
        "first_name": "John",
        "last_name": "Doe"
      },
      "rating": 5,
      "comment": "Excellent doctor!",
      "created_at": "2024-01-10T14:30:00Z"
    }
  ],
  "review_count": 25
}
```

**POST** `/identity-service/v1/doctors/{doctor_id}/reviews/`

Add a review for a doctor (Patient only).

**Request Body:**
```json
{
  "rating": 5,
  "comment": "Excellent consultation and very professional."
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "doctor_id": "uuid",
  "patient_id": "uuid",
  "rating": 5,
  "comment": "Excellent consultation and very professional.",
  "created_at": "2024-01-15T12:00:00Z"
}
```

##### User Profile Endpoints

**GET** `/identity-service/v1/users/me/`

Get current user's full profile context.

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "email": "patient@example.com",
  "role": "PATIENT",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "patient_profile": {
    "id": "uuid",
    "date_of_birth": "1990-05-15",
    "gender": "MALE",
    "blood_group": "O+",
    "emergency_contact": {
      "name": "Jane Doe",
      "phone": "+1234567890",
      "relation": "Spouse"
    }
  },
  "created_at": "2024-01-01T10:00:00Z"
}
```

**PATCH** `/identity-service/v1/users/me/`

Update current user's profile.

**Request Body:**
```json
{
  "first_name": "John",
  "last_name": "Doe",
  "patient_profile": {
    "date_of_birth": "1990-05-15",
    "gender": "MALE",
    "blood_group": "O+"
  }
}
```

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "email": "patient@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "patient_profile": {
    "date_of_birth": "1990-05-15",
    "gender": "MALE",
    "blood_group": "O+"
  }
}
```

##### Hospital Admin Endpoints

**POST** `/identity-service/v1/hospitals/{hospital_id}/doctors/`

Onboard a new doctor to the hospital (Hospital Admin only).

**Request Body:**
```json
{
  "email": "newdoctor@hospital.com",
  "password": "SecurePassword123!",
  "first_name": "Dr. Robert",
  "last_name": "Johnson",
  "department_id": "uuid",
  "specialization": "Neurologist",
  "experience_years": 8,
  "consultation_fee": 200.00,
  "bio": "Board-certified neurologist..."
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "user": {
    "id": "uuid",
    "email": "newdoctor@hospital.com",
    "first_name": "Dr. Robert",
    "last_name": "Johnson"
  },
  "hospital_id": "uuid",
  "department_id": "uuid",
  "specialization": "Neurologist",
  "consultation_fee": 200.00
}
```

#### Spring Boot Service APIs

##### Slot Management Endpoints

**GET** `/booking-service/v1/slots/`

Get available slots for a specific doctor on a given date.

**Query Parameters:**
- `doctor_id` (uuid, required): Doctor's user ID from Django
- `date` (date, required): Date in YYYY-MM-DD format
- `include_booked` (boolean, optional): Include booked slots (default: false)

**Response:** `200 OK`
```json
{
  "doctor_id": "uuid",
  "date": "2024-01-20",
  "slots": [
    {
      "id": "uuid",
      "start_time": "2024-01-20T09:00:00Z",
      "end_time": "2024-01-20T09:30:00Z",
      "is_booked": false
    },
    {
      "id": "uuid",
      "start_time": "2024-01-20T09:30:00Z",
      "end_time": "2024-01-20T10:00:00Z",
      "is_booked": false
    },
    {
      "id": "uuid",
      "start_time": "2024-01-20T10:00:00Z",
      "end_time": "2024-01-20T10:30:00Z",
      "is_booked": true
    }
  ]
}
```

**POST** `/booking-service/v1/slots/`

Create time slots for a doctor (Doctor or Hospital Admin only).

**Request Body:**
```json
{
  "doctor_id": "uuid",
  "slots": [
    {
      "start_time": "2024-01-20T09:00:00Z",
      "end_time": "2024-01-20T09:30:00Z"
    },
    {
      "start_time": "2024-01-20T09:30:00Z",
      "end_time": "2024-01-20T10:00:00Z"
    }
  ]
}
```

**Response:** `201 Created`
```json
{
  "created_count": 2,
  "slots": [
    {
      "id": "uuid",
      "doctor_id": "uuid",
      "start_time": "2024-01-20T09:00:00Z",
      "end_time": "2024-01-20T09:30:00Z",
      "is_booked": false
    }
  ]
}
```

##### Booking Endpoints

**POST** `/booking-service/v1/bookings/initiate`

Initiate a booking by locking a slot and creating a PENDING appointment.

**Request Body:**
```json
{
  "slot_id": "uuid",
  "doctor_id": "uuid"
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "patient_id": "uuid",
  "doctor_id": "uuid",
  "time_slot_id": "uuid",
  "status": "PENDING",
  "slot": {
    "id": "uuid",
    "start_time": "2024-01-20T09:00:00Z",
    "end_time": "2024-01-20T09:30:00Z"
  },
  "created_at": "2024-01-15T14:30:00Z",
  "expires_at": "2024-01-15T15:00:00Z"
}
```

**Note**: The slot is locked in Redis for 30 minutes. The appointment must be confirmed (via payment webhook in Phase 2) or it will expire and the slot will be released.

**POST** `/booking-service/v1/bookings/{appointment_id}/confirm`

Confirm a PENDING appointment (called by payment webhook in Phase 2, or manual confirmation in Phase 1).

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "status": "CONFIRMED",
  "updated_at": "2024-01-15T15:00:00Z"
}
```

**GET** `/booking-service/v1/bookings/my-appointments`

List appointments for the logged-in user (Patient sees their appointments, Doctor sees their schedule).

**Query Parameters:**
- `status` (enum, optional): Filter by status (PENDING, CONFIRMED, CANCELLED, COMPLETED)
- `from_date` (date, optional): Filter appointments from this date
- `to_date` (date, optional): Filter appointments to this date
- `page` (integer, optional): Page number
- `page_size` (integer, optional): Items per page (default: 20)

**Response:** `200 OK`
```json
{
  "count": 10,
  "results": [
    {
      "id": "uuid",
      "patient_id": "uuid",
      "doctor_id": "uuid",
      "time_slot_id": "uuid",
      "status": "CONFIRMED",
      "slot": {
        "start_time": "2024-01-20T09:00:00Z",
        "end_time": "2024-01-20T09:30:00Z"
      },
      "created_at": "2024-01-15T14:30:00Z"
    }
  ]
}
```

**POST** `/booking-service/v1/bookings/{appointment_id}/cancel`

Cancel an appointment and release the slot.

**Request Body:**
```json
{
  "reason": "Patient requested cancellation"
}
```

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "status": "CANCELLED",
  "updated_at": "2024-01-15T16:00:00Z"
}
```

**POST** `/booking-service/v1/bookings/{appointment_id}/complete`

Mark an appointment as completed (Doctor only). Unlocks EHR creation capability for Phase 3.

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "status": "COMPLETED",
  "updated_at": "2024-01-20T09:35:00Z"
}
```

### Workflows

#### User Registration and Hospital Onboarding Flow

```mermaid
sequenceDiagram
    participant SuperAdmin
    participant DjangoService
    participant HospitalAdmin
    participant Doctor
    participant PostgreSQL

    SuperAdmin->>DjangoService: POST /hospitals/onboard
    DjangoService->>PostgreSQL: Create Hospital record
    PostgreSQL-->>DjangoService: Hospital created
    DjangoService-->>SuperAdmin: Hospital ID + credentials

    SuperAdmin->>DjangoService: PATCH /hospitals/{id}/verify
    DjangoService->>PostgreSQL: Update is_verified = true
    PostgreSQL-->>DjangoService: Updated
    DjangoService-->>SuperAdmin: Hospital verified

    HospitalAdmin->>DjangoService: POST /hospitals/{id}/doctors
    DjangoService->>PostgreSQL: Create User + DoctorProfile
    PostgreSQL-->>DjangoService: Doctor created
    DjangoService-->>HospitalAdmin: Doctor profile

    Doctor->>DjangoService: POST /auth/login
    DjangoService->>PostgreSQL: Validate credentials
    PostgreSQL-->>DjangoService: User validated
    DjangoService-->>Doctor: JWT tokens
```

#### Doctor Discovery and Slot Booking Flow

```mermaid
sequenceDiagram
    participant Patient
    participant ReactFrontend
    participant DjangoService
    participant SpringBootService
    participant Redis
    participant PostgreSQL

    Patient->>ReactFrontend: Search doctors
    ReactFrontend->>DjangoService: GET /doctors/?specialization=Cardiology
    DjangoService->>PostgreSQL: Query with filters
    PostgreSQL-->>DjangoService: Doctor list
    DjangoService-->>ReactFrontend: Filtered doctors
    ReactFrontend-->>Patient: Display results

    Patient->>ReactFrontend: View doctor details
    ReactFrontend->>DjangoService: GET /doctors/{id}
    DjangoService->>PostgreSQL: Get doctor + reviews
    PostgreSQL-->>DjangoService: Doctor profile
    DjangoService-->>ReactFrontend: Full profile
    ReactFrontend-->>Patient: Display profile

    Patient->>ReactFrontend: Check availability
    ReactFrontend->>SpringBootService: GET /slots/?doctor_id={id}&date=2024-01-20
    SpringBootService->>PostgreSQL: Query available slots
    PostgreSQL-->>SpringBootService: Slot list
    SpringBootService-->>ReactFrontend: Available slots
    ReactFrontend-->>Patient: Display slots

    Patient->>ReactFrontend: Book appointment
    ReactFrontend->>SpringBootService: POST /bookings/initiate
    SpringBootService->>Redis: SETNX slot:{slotId}:lock
    alt Lock acquired
        Redis-->>SpringBootService: Lock acquired
        SpringBootService->>PostgreSQL: BEGIN TRANSACTION
        SpringBootService->>PostgreSQL: SELECT FOR UPDATE TimeSlot
        PostgreSQL-->>SpringBootService: Slot with version
        SpringBootService->>SpringBootService: Check version & isBooked
        alt Slot available
            SpringBootService->>PostgreSQL: INSERT Appointment (PENDING)
            SpringBootService->>PostgreSQL: UPDATE TimeSlot (isBooked=true, version++)
            SpringBootService->>PostgreSQL: COMMIT
            PostgreSQL-->>SpringBootService: Success
            SpringBootService->>Redis: SET lock TTL 30min
            SpringBootService-->>ReactFrontend: Appointment created (PENDING)
            ReactFrontend-->>Patient: Booking confirmed
        else Slot already booked
            SpringBootService->>PostgreSQL: ROLLBACK
            SpringBootService->>Redis: DEL lock
            SpringBootService-->>ReactFrontend: 409 Conflict
            ReactFrontend-->>Patient: Slot unavailable
        end
    else Lock not acquired
        Redis-->>SpringBootService: Lock exists
        SpringBootService-->>ReactFrontend: 409 Conflict (concurrent booking)
        ReactFrontend-->>Patient: Please try again
    end
```

#### Slot Allocation Mechanism

The slot allocation mechanism uses a two-phase approach to prevent race conditions:

**Phase 1: Distributed Lock (Redis)**
- When a booking is initiated, a Redis lock is acquired using `SETNX` with key pattern `slot:{slotId}:lock`
- Lock has a TTL of 30 minutes to prevent deadlocks
- Only one request can acquire the lock at a time

**Phase 2: Database-Level Consistency (PostgreSQL)**
- Within the locked section, a database transaction is started
- `SELECT FOR UPDATE` ensures row-level locking on the TimeSlot
- Optimistic locking via `version` field detects concurrent modifications
- If version mismatch or `isBooked=true`, transaction rolls back
- If successful, both Appointment and TimeSlot are updated atomically

**Concurrency Scenarios:**

1. **Two patients book same slot simultaneously:**
   - Patient A acquires Redis lock → proceeds to DB transaction
   - Patient B tries to acquire lock → fails, returns 409 Conflict
   - Patient A completes booking, releases lock

2. **Slot becomes unavailable during lock:**
   - Patient A acquires Redis lock
   - Doctor cancels slot (sets isBooked=true) in separate transaction
   - Patient A's transaction detects `isBooked=true` → rolls back → returns error

3. **Optimistic locking conflict:**
   - Patient A reads slot with version=5
   - Patient B (from different instance) reads slot with version=5
   - Patient A updates → version becomes 6
   - Patient B tries to update → version mismatch → rollback

### Phases

**Phase 1 Implementation Plan:**

1. **Week 1-2: Django Service Foundation**
   - Set up Django project with PostgreSQL
   - Implement User, Hospital, Department models
   - Implement authentication with JWT
   - Create basic CRUD APIs for hospitals

2. **Week 3-4: Doctor and Patient Profiles**
   - Implement DoctorProfile and PatientProfile models
   - Create doctor discovery APIs with filtering
   - Implement review system
   - Add RBAC permissions

3. **Week 5-6: Spring Boot Service Foundation**
   - Set up Spring Boot project with PostgreSQL
   - Implement TimeSlot and Appointment entities
   - Configure JWT validation from Django
   - Create slot management APIs

4. **Week 7-8: Booking and Concurrency**
   - Implement Redis distributed locking
   - Add optimistic locking to TimeSlot entity
   - Implement booking initiation and confirmation flows
   - Add appointment management APIs

5. **Week 9-10: Integration and Testing**
   - Frontend integration with both services
   - End-to-end testing of booking flow
   - Load testing for concurrency scenarios
   - Documentation and deployment preparation

## Security Considerations

### Authentication and Authorization

- [x] Is required and conforms to the mechanisms provided by the Ministry of Auth, as described in [Identity Management](/identity-access-management/authentication)

**Authentication Strategy:**

1. **Django Service (Identity Provider):**
   - Issues JWT tokens (access + refresh) upon successful login
   - Access token contains: `user_id`, `email`, `role`, `hospital_id` (if applicable)
   - Access token expiry: 1 hour
   - Refresh token expiry: 7 days
   - Uses HS256 algorithm with shared secret (or RS256 with public/private key pair for production)

2. **Spring Boot Service (Resource Server):**
   - Validates JWT signature using shared secret/public key
   - Extracts `user_id` and `role` from token claims
   - No database lookup required for authentication (stateless)
   - Spring Security filter chain validates token on each request

**Authorization (RBAC):**

| Role | Permissions |
|------|------------|
| SUPER_ADMIN | Create/verify hospitals, manage all users |
| HOSPITAL_ADMIN | Onboard doctors to their hospital, manage hospital departments |
| DOCTOR | Create/manage own time slots, view own appointments, complete appointments |
| PATIENT | Search doctors, book appointments, view own appointments, write reviews |

**API Authorization Matrix:**

| Endpoint | Method | SUPER_ADMIN | HOSPITAL_ADMIN | DOCTOR | PATIENT | Public |
|----------|--------|-------------|----------------|--------|---------|--------|
| `/auth/register` | POST | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/auth/login` | POST | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/hospitals/onboard` | POST | ✓ | | | | |
| `/hospitals/{id}/verify` | PATCH | ✓ | | | | |
| `/hospitals/{id}/doctors` | POST | ✓ | ✓ (own hospital) | | | |
| `/doctors/` | GET | ✓ | ✓ | ✓ | ✓ | ✓ |
| `/doctors/{id}/reviews` | POST | | | | ✓ | |
| `/slots/` | GET | ✓ | ✓ | ✓ | ✓ | |
| `/slots/` | POST | ✓ | ✓ | ✓ | | |
| `/bookings/initiate` | POST | | | | ✓ | |
| `/bookings/my-appointments` | GET | ✓ | ✓ | ✓ | ✓ | |
| `/bookings/{id}/cancel` | POST | | | ✓ | ✓ | |
| `/bookings/{id}/complete` | POST | | | ✓ | | |

### Logging

**Sensitive Information Handling:**
- Passwords: Never logged (only password hash stored)
- JWT tokens: Logged only as token identifiers (first/last 4 characters) for debugging
- Patient PII: Patient profiles contain sensitive data (DOB, emergency contacts) - logged only as IDs, not full data
- Appointment details: Logged with appointment IDs, not full patient/doctor details

**Audit Logging:**
- All authentication attempts (success/failure) logged with IP address
- Hospital verification actions logged with admin user ID
- Appointment status changes logged with user ID and timestamp
- Slot booking conflicts logged for monitoring concurrency issues

**Log Format:**
```
[2024-01-15T14:30:00Z] INFO booking-service Booking initiated: appointment_id=uuid-123, patient_id=uuid-456, slot_id=uuid-789
[2024-01-15T14:30:01Z] WARN booking-service Concurrent booking detected: slot_id=uuid-789, lock_acquired=false
```

## International Considerations

**Phase 1 Scope:**
- Initial deployment targeted for US market
- English language only
- Date/time formats: ISO 8601 (UTC timestamps stored, local time displayed in frontend)
- Currency: USD only (consultation fees)

**Future Considerations (Post-Phase 1):**
- Multi-language support for doctor bios and hospital information
- Timezone handling for slot display (doctor's timezone vs patient's timezone)
- Currency localization for consultation fees
- Date format localization (MM/DD/YYYY vs DD/MM/YYYY)

**Contact**: #international-rd on Slack for Phase 2+ internationalization planning

## References

### Normative References

- Django REST Framework Documentation. (2024). [Django REST Framework](https://www.django-rest-framework.org/)
- Spring Boot Documentation. (2024). [Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/reference/index.html)
- PostgreSQL Documentation. (2024). [PostgreSQL Documentation](https://www.postgresql.org/docs/)

### Informative References

- Yadav, Himanshu. (2025). Hospital Management System - Phase 1 Architecture. Healthcare Platform Team. Retrieved from [System Docs](/healthcare/hospital-management-system/phase-1-architecture)

Hospital Management System - Phase 1 Development Plan
=======

| Status        | Build               |
|---------------|---------------------|
| Category      | Implementation Plan |
| Team          | Healthcare Platform |
| Author(s)     | Himanshu Yadav      |
| Created Date  | January 2025        |
| Modified Date | January 2025        |

**Abstract**

This document provides a detailed development plan for Phase 1 of the Hospital Management System. The plan outlines a 10-week implementation timeline broken down into sprints, with specific tasks, deliverables, dependencies, and testing strategies. This plan serves as a roadmap for the development team to execute the architecture defined in the Phase 1 Architecture document.

## Overview

### Timeline
- **Total Duration**: 10 weeks
- **Sprint Length**: 2 weeks per sprint
- **Total Sprints**: 5 sprints

### Team Structure
- **Backend Developers**: 2 (1 Django, 1 Spring Boot)
- **Frontend Developer**: 1
- **DevOps Engineer**: 0.5 (shared)
- **QA Engineer**: 1
- **Tech Lead**: 0.5 (shared)

### Technology Prerequisites

**Development Environment:**
- Python 3.11+
- Java 17+
- Node.js 18+ (for React frontend)
- Docker & Docker Compose
- PostgreSQL 15+
- Redis 7+

**Tools:**
- Git (version control)
- Postman/Insomnia (API testing)
- pgAdmin/DBeaver (database management)
- IntelliJ IDEA / PyCharm / VS Code (IDEs)

## Sprint Breakdown

### Sprint 1: Project Setup & Django Foundation (Weeks 1-2)

**Goal**: Establish project infrastructure and implement core Django models and authentication.

#### Week 1: Infrastructure Setup

**Tasks:**

1. **Project Repository Setup**
   - [ ] Create GitHub repository with proper structure
   - [ ] Set up `.gitignore` files for Python and Java
   - [ ] Initialize README files for each service
   - [ ] Configure branch protection rules
   - [ ] Set up CI/CD pipeline basics (GitHub Actions)

2. **Django Service Setup**
   - [ ] Create Django project structure (`identity-service/`)
   - [ ] Set up virtual environment and `requirements.txt`
   - [ ] Install dependencies: Django 4.2+, DRF, django-cors-headers, djangorestframework-simplejwt
   - [ ] Configure `settings.py` with database, CORS, JWT settings
   - [ ] Set up environment variables (.env file)
   - [ ] Create Dockerfile and docker-compose.yml for local development
   - [ ] Configure PostgreSQL connection

3. **Database Setup**
   - [ ] Create PostgreSQL database `db_identity`
   - [ ] Set up database migrations structure
   - [ ] Configure database connection pooling

4. **Spring Boot Service Setup**
   - [ ] Create Spring Boot project (`booking-service/`)
   - [ ] Set up Maven/Gradle build configuration
   - [ ] Add dependencies: Spring Boot 3.1+, Spring Data JPA, Spring Security, Redis
   - [ ] Configure `application.yml` with database and Redis settings
   - [ ] Create Dockerfile and docker-compose integration
   - [ ] Set up PostgreSQL database `db_clinical`

5. **Development Environment**
   - [ ] Create docker-compose.yml with all services (Django, Spring Boot, PostgreSQL, Redis)
   - [ ] Set up local development scripts
   - [ ] Configure IDE settings and code formatting (Black for Python, Google Java Style)
   - [ ] Set up pre-commit hooks (linting, formatting)

**Deliverables:**
- ✅ Both services running locally via Docker Compose
- ✅ Database connections established
- ✅ Basic health check endpoints working
- ✅ CI/CD pipeline running basic checks

**Acceptance Criteria:**
- `GET /identity-service/v1/health` returns 200 OK
- `GET /booking-service/v1/health` returns 200 OK
- Database migrations can be run successfully
- Docker Compose brings up all services without errors

---

#### Week 2: Django Models & Authentication

**Tasks:**

1. **Django Models Implementation**
   - [ ] Create `User` model extending `AbstractUser`
     - [ ] Add UUID primary key
     - [ ] Add role enum (SUPER_ADMIN, HOSPITAL_ADMIN, DOCTOR, PATIENT)
     - [ ] Add email uniqueness constraint
   - [ ] Create `Hospital` model
     - [ ] Add fields: name, address, registration_number, is_verified
     - [ ] Add unique constraint on registration_number
   - [ ] Create `Department` model
     - [ ] Add foreign key to Hospital
     - [ ] Add name field
   - [ ] Create `DoctorProfile` model
     - [ ] Add OneToOne relationship with User
     - [ ] Add foreign keys to Hospital and Department
     - [ ] Add fields: specialization, experience_years, consultation_fee, bio, rating_avg
   - [ ] Create `PatientProfile` model
     - [ ] Add OneToOne relationship with User
     - [ ] Add fields: date_of_birth, gender, blood_group, emergency_contact (JSONB)
   - [ ] Create `Review` model
     - [ ] Add foreign keys to DoctorProfile and PatientProfile
     - [ ] Add rating (1-5) and comment fields
   - [ ] Create database migrations
   - [ ] Run migrations and verify schema

2. **Django Admin Setup**
   - [ ] Register all models in Django admin
   - [ ] Create custom admin classes with list filters and search
   - [ ] Test admin interface for CRUD operations

3. **JWT Authentication Implementation**
   - [ ] Configure `djangorestframework-simplejwt`
   - [ ] Create custom token serializer to include role and hospital_id
   - [ ] Implement `/auth/register/` endpoint
     - [ ] Validate email format and password strength
     - [ ] Hash password using bcrypt
     - [ ] Create User and associated profile (Doctor/Patient)
   - [ ] Implement `/auth/login/` endpoint
     - [ ] Validate credentials
     - [ ] Return access and refresh tokens
   - [ ] Implement `/auth/refresh/` endpoint
   - [ ] Create JWT authentication middleware
   - [ ] Write unit tests for authentication endpoints

4. **Basic API Structure**
   - [ ] Set up URL routing structure
   - [ ] Create API versioning (`/v1/`)
   - [ ] Implement basic serializers for User and Hospital
   - [ ] Create basic view sets with proper HTTP methods

**Deliverables:**
- ✅ All Django models created and migrated
- ✅ User registration and login working
- ✅ JWT tokens issued and validated
- ✅ Django admin accessible with all models

**Acceptance Criteria:**
- Can register a new user via `/auth/register/`
- Can login and receive JWT tokens via `/auth/login/`
- JWT token contains user_id, email, role claims
- All models visible and editable in Django admin
- Unit tests pass for authentication endpoints

**Testing:**
- Unit tests for User model creation
- Unit tests for authentication endpoints
- Integration tests for registration flow
- Manual testing via Postman

---

### Sprint 2: Django APIs & Hospital Management (Weeks 3-4)

**Goal**: Implement hospital management, doctor profiles, and discovery APIs.

#### Week 3: Hospital & Doctor Management APIs

**Tasks:**

1. **Hospital Management APIs**
   - [ ] Implement `GET /hospitals/` endpoint
     - [ ] Add search by name functionality
     - [ ] Add filtering by is_verified
     - [ ] Add pagination
     - [ ] Create serializer with proper field selection
   - [ ] Implement `POST /hospitals/onboard/` endpoint
     - [ ] Add permission check (SUPER_ADMIN only)
     - [ ] Validate registration_number uniqueness
     - [ ] Create hospital with is_verified=false
   - [ ] Implement `PATCH /hospitals/{id}/verify/` endpoint
     - [ ] Add permission check (SUPER_ADMIN only)
     - [ ] Update is_verified to true
   - [ ] Write unit tests for hospital endpoints

2. **Department Management**
   - [ ] Implement `GET /hospitals/{id}/departments/` endpoint
   - [ ] Implement `POST /hospitals/{id}/departments/` endpoint
     - [ ] Add permission check (HOSPITAL_ADMIN only, own hospital)
   - [ ] Create DepartmentSerializer
   - [ ] Write unit tests

3. **Doctor Profile APIs**
   - [ ] Implement `POST /hospitals/{id}/doctors/` endpoint
     - [ ] Add permission check (HOSPITAL_ADMIN or SUPER_ADMIN)
     - [ ] Create User with role=DOCTOR
     - [ ] Create DoctorProfile linked to hospital and department
     - [ ] Validate department belongs to hospital
   - [ ] Implement `GET /doctors/` endpoint
     - [ ] Add filtering by specialization, hospital_id, department_id
     - [ ] Add filtering by min_fee, max_fee, min_rating
     - [ ] Add search across name, specialization, bio
     - [ ] Add pagination
     - [ ] Optimize queries with select_related/prefetch_related
   - [ ] Implement `GET /doctors/{id}/` endpoint
     - [ ] Include hospital and department details
     - [ ] Include reviews with patient names (anonymized if needed)
     - [ ] Calculate review_count and rating_avg
   - [ ] Create DoctorProfileSerializer with nested serializers
   - [ ] Write unit tests for doctor endpoints

4. **Patient Profile APIs**
   - [ ] Implement `GET /users/me/` endpoint
     - [ ] Return user with associated profile (Doctor/Patient)
     - [ ] Include nested profile details
   - [ ] Implement `PATCH /users/me/` endpoint
     - [ ] Allow updating user fields and profile fields
     - [ ] Validate data (e.g., date_of_birth format)
   - [ ] Create PatientProfileSerializer
   - [ ] Write unit tests

**Deliverables:**
- ✅ Hospital CRUD operations working
- ✅ Doctor onboarding and listing working
- ✅ Patient profile management working
- ✅ All endpoints have proper permissions

**Acceptance Criteria:**
- Super admin can create and verify hospitals
- Hospital admin can onboard doctors to their hospital
- Patients can search and filter doctors
- Users can view and update their own profiles
- All permission checks working correctly

**Testing:**
- Unit tests for all endpoints
- Integration tests for hospital onboarding flow
- Permission tests for RBAC
- Performance tests for doctor search (with 1000+ doctors)

---

#### Week 4: Reviews & Advanced Filtering

**Tasks:**

1. **Review System**
   - [ ] Implement `POST /doctors/{id}/reviews/` endpoint
     - [ ] Add permission check (PATIENT only)
     - [ ] Validate rating (1-5)
     - [ ] Prevent duplicate reviews (one review per patient per doctor)
     - [ ] Create Review model entry
   - [ ] Implement review aggregation
     - [ ] Update DoctorProfile.rating_avg on review creation
     - [ ] Update review_count
     - [ ] Use database aggregation for accuracy
   - [ ] Include reviews in `GET /doctors/{id}/` response
     - [ ] Paginate reviews
     - [ ] Order by created_at descending
   - [ ] Create ReviewSerializer
   - [ ] Write unit tests

2. **Advanced Search & Filtering**
   - [ ] Enhance `GET /doctors/` endpoint
     - [ ] Add full-text search using PostgreSQL full-text search
     - [ ] Add location-based filtering (by hospital address)
     - [ ] Add sorting options (rating, fee, experience)
     - [ ] Optimize queries with proper indexes
   - [ ] Create database indexes
     - [ ] Index on DoctorProfile.hospital_id
     - [ ] Index on DoctorProfile.specialization
     - [ ] Composite index on (hospital_id, specialization)
     - [ ] Index on Hospital.name for search
   - [ ] Write performance tests

3. **API Documentation**
   - [ ] Set up drf-spectacular (OpenAPI/Swagger)
   - [ ] Add API documentation annotations
   - [ ] Generate OpenAPI schema
   - [ ] Create Postman collection
   - [ ] Document all endpoints with examples

4. **Error Handling & Validation**
   - [ ] Create custom exception handlers
   - [ ] Standardize error response format
   - [ ] Add input validation for all endpoints
   - [ ] Handle edge cases (e.g., hospital not found, invalid department)

**Deliverables:**
- ✅ Review system fully functional
- ✅ Advanced search and filtering working
- ✅ API documentation available
- ✅ Proper error handling implemented

**Acceptance Criteria:**
- Patients can add reviews for doctors
- Doctor ratings update automatically
- Search returns relevant results quickly (<200ms)
- API documentation is complete and accurate
- Error responses are consistent and helpful

**Testing:**
- Unit tests for review creation and aggregation
- Integration tests for search functionality
- Load testing for search endpoint (1000+ concurrent requests)
- API documentation review

---

### Sprint 3: Spring Boot Foundation & Slot Management (Weeks 5-6)

**Goal**: Implement Spring Boot service with slot management and JWT validation.

#### Week 5: Spring Boot Setup & JWT Integration

**Tasks:**

1. **Spring Boot Project Setup**
   - [ ] Create entity classes: `TimeSlot`, `Appointment`
   - [ ] Set up JPA repositories
   - [ ] Configure database connection (db_clinical)
   - [ ] Create Liquibase migrations for schema
   - [ ] Set up Redis connection and configuration
   - [ ] Create health check endpoint

2. **JWT Authentication Integration**
   - [ ] Add Spring Security dependencies
   - [ ] Create JWT validation filter
     - [ ] Extract token from Authorization header
     - [ ] Validate token signature (using Django's secret key)
     - [ ] Extract user_id and role from claims
     - [ ] Set SecurityContext with authentication
   - [ ] Configure Spring Security filter chain
   - [ ] Create custom UserDetails implementation
   - [ ] Test JWT validation with tokens from Django
   - [ ] Write unit tests for JWT filter

3. **TimeSlot Entity Implementation**
   - [ ] Create `TimeSlot` entity
     - [ ] UUID primary key
     - [ ] doctorId (UUID, references Django User.id)
     - [ ] startTime, endTime (TIMESTAMP)
     - [ ] isBooked (BOOLEAN)
     - [ ] version (INTEGER) for optimistic locking
     - [ ] Add `@Version` annotation
   - [ ] Create TimeSlotRepository with custom queries
     - [ ] Query by doctorId and date range
     - [ ] Query available slots (isBooked=false)
   - [ ] Create database indexes
     - [ ] Index on doctorId
     - [ ] Index on startTime
     - [ ] Composite index on (doctorId, startTime, isBooked)
   - [ ] Write unit tests for TimeSlot entity

4. **Appointment Entity Implementation**
   - [ ] Create `Appointment` entity
     - [ ] UUID primary key
     - [ ] patientId, doctorId (UUID, references Django User.id)
     - [ ] timeSlotId (FK to TimeSlot, UNIQUE)
     - [ ] status (ENUM: PENDING, CONFIRMED, CANCELLED, COMPLETED)
     - [ ] createdAt, updatedAt timestamps
   - [ ] Create AppointmentRepository
   - [ ] Create database indexes
     - [ ] Index on patientId
     - [ ] Index on doctorId
     - [ ] Index on status
   - [ ] Write unit tests

**Deliverables:**
- ✅ Spring Boot service running and connected to database
- ✅ JWT authentication working with Django tokens
- ✅ TimeSlot and Appointment entities created
- ✅ Database schema migrated

**Acceptance Criteria:**
- Spring Boot accepts JWT tokens from Django
- Can create TimeSlot records via repository
- Database indexes created correctly
- Health check endpoint returns 200 OK

**Testing:**
- Unit tests for JWT validation
- Integration tests for entity creation
- Manual testing with Postman using Django JWT tokens

---

#### Week 6: Slot Management APIs

**Tasks:**

1. **Slot Query API**
   - [ ] Implement `GET /slots/` endpoint
     - [ ] Query parameters: doctor_id (required), date (required)
     - [ ] Query available slots for doctor on given date
     - [ ] Filter by isBooked=false by default
     - [ ] Optional include_booked parameter
     - [ ] Return slots with start_time, end_time, is_booked
     - [ ] Optimize query with proper joins
   - [ ] Create SlotDTO for response
   - [ ] Write unit tests
   - [ ] Write integration tests

2. **Slot Creation API**
   - [ ] Implement `POST /slots/` endpoint
     - [ ] Add permission check (DOCTOR or HOSPITAL_ADMIN)
     - [ ] Validate doctor_id belongs to authenticated user (if DOCTOR)
     - [ ] Accept array of slots to create
     - [ ] Validate slot times (no overlaps, valid duration)
     - [ ] Create TimeSlot records in batch
     - [ ] Return created slots
   - [ ] Add business logic validation
     - [ ] Slot duration should be 30 minutes (configurable)
     - [ ] No overlapping slots for same doctor
     - [ ] Slots must be in the future
   - [ ] Write unit tests
   - [ ] Write integration tests

3. **Redis Configuration**
   - [ ] Set up Redis connection pool
   - [ ] Create RedisTemplate configuration
   - [ ] Create RedisService utility class
     - [ ] Methods: acquireLock, releaseLock, isLocked
     - [ ] Use SETNX with TTL
   - [ ] Write unit tests for Redis operations

4. **Error Handling**
   - [ ] Create custom exceptions (SlotNotFoundException, SlotAlreadyBookedException)
   - [ ] Create global exception handler
   - [ ] Standardize error response format (match Django format)
   - [ ] Add validation error handling

**Deliverables:**
- ✅ Slot query and creation APIs working
- ✅ Redis locking mechanism implemented
- ✅ Proper error handling in place

**Acceptance Criteria:**
- Can query available slots for a doctor
- Doctors can create their own slots
- Slot validation prevents overlaps
- Redis locking utility ready for use

**Testing:**
- Unit tests for slot queries
- Integration tests for slot creation
- Redis locking tests
- Manual testing via Postman

---

### Sprint 4: Booking System & Concurrency (Weeks 7-8)

**Goal**: Implement booking flow with concurrency control and appointment management.

#### Week 7: Booking Initiation & Concurrency Control

**Tasks:**

1. **Booking Service Implementation**
   - [ ] Create BookingService class
     - [ ] Method: initiateBooking(slotId, doctorId, patientId)
     - [ ] Implement two-phase locking:
       1. Acquire Redis lock
       2. Database transaction with optimistic locking
   - [ ] Implement Redis locking logic
     - [ ] Try to acquire lock with SETNX
     - [ ] Set TTL (30 minutes)
     - [ ] Handle lock acquisition failure
   - [ ] Implement database transaction
     - [ ] Use @Transactional annotation
     - [ ] SELECT FOR UPDATE on TimeSlot
     - [ ] Check version and isBooked
     - [ ] Create Appointment with status=PENDING
     - [ ] Update TimeSlot (isBooked=true, version++)
   - [ ] Handle rollback scenarios
     - [ ] Release Redis lock on failure
     - [ ] Handle OptimisticLockException
   - [ ] Write unit tests with mocked Redis and database

2. **Booking Initiation API**
   - [ ] Implement `POST /bookings/initiate` endpoint
     - [ ] Extract patientId from JWT token
     - [ ] Validate slotId and doctorId
     - [ ] Call BookingService.initiateBooking
     - [ ] Return appointment with status=PENDING
     - [ ] Include expires_at timestamp
   - [ ] Create BookingDTO for request/response
   - [ ] Add input validation
   - [ ] Write integration tests

3. **Concurrency Testing**
   - [ ] Create load test scenario
     - [ ] Simulate 10 concurrent booking requests for same slot
     - [ ] Verify only one succeeds
     - [ ] Verify others return 409 Conflict
   - [ ] Test optimistic locking
     - [ ] Simulate version conflict
     - [ ] Verify rollback and error handling
   - [ ] Document concurrency behavior

4. **Appointment Confirmation API**
   - [ ] Implement `POST /bookings/{id}/confirm` endpoint
     - [ ] Update appointment status: PENDING → CONFIRMED
     - [ ] Validate appointment exists and is PENDING
     - [ ] Add permission check (can be called by payment webhook or admin)
   - [ ] Write unit tests

**Deliverables:**
- ✅ Booking initiation working with concurrency control
- ✅ Redis locking preventing race conditions
- ✅ Optimistic locking preventing database conflicts
- ✅ Load tests passing

**Acceptance Criteria:**
- Multiple concurrent booking attempts handled correctly
- Only one booking succeeds per slot
- Failed bookings return appropriate error codes
- Database remains consistent under load

**Testing:**
- Unit tests for BookingService
- Integration tests for booking flow
- Load tests with concurrent requests
- Chaos testing (simulate Redis failures)

---

#### Week 8: Appointment Management APIs

**Tasks:**

1. **Appointment Query API**
   - [ ] Implement `GET /bookings/my-appointments` endpoint
     - [ ] Extract user_id from JWT
     - [ ] Determine user role (PATIENT or DOCTOR)
     - [ ] Query appointments based on role
       - PATIENT: appointments where patientId = user_id
       - DOCTOR: appointments where doctorId = user_id
     - [ ] Add filtering by status, from_date, to_date
     - [ ] Add pagination
     - [ ] Include slot details in response
     - [ ] Order by created_at descending
   - [ ] Create AppointmentDTO with nested SlotDTO
   - [ ] Write unit tests
   - [ ] Write integration tests

2. **Appointment Cancellation API**
   - [ ] Implement `POST /bookings/{id}/cancel` endpoint
     - [ ] Validate appointment exists
     - [ ] Check permissions (PATIENT can cancel own, DOCTOR can cancel own)
     - [ ] Validate status (can only cancel PENDING or CONFIRMED)
     - [ ] Update appointment status: → CANCELLED
     - [ ] Update TimeSlot: isBooked = false
     - [ ] Release the slot for re-booking
   - [ ] Add cancellation reason (optional)
   - [ ] Write unit tests
   - [ ] Write integration tests

3. **Appointment Completion API**
   - [ ] Implement `POST /bookings/{id}/complete` endpoint
     - [ ] Add permission check (DOCTOR only, own appointments)
     - [ ] Validate status (must be CONFIRMED)
     - [ ] Update appointment status: CONFIRMED → COMPLETED
     - [ ] This unlocks EHR creation capability (for Phase 3)
   - [ ] Write unit tests

4. **Appointment State Machine**
   - [ ] Document valid state transitions
     - PENDING → CONFIRMED (via confirm)
     - PENDING → CANCELLED (via cancel)
     - CONFIRMED → COMPLETED (via complete)
     - CONFIRMED → CANCELLED (via cancel)
   - [ ] Implement state validation logic
   - [ ] Add state transition logging
   - [ ] Write unit tests for state machine

5. **Slot Release Logic**
   - [ ] Create method to release slot on cancellation
   - [ ] Handle edge cases (slot already released, appointment not found)
   - [ ] Add idempotency checks
   - [ ] Write unit tests

**Deliverables:**
- ✅ Appointment query, cancellation, and completion working
- ✅ State machine properly enforced
- ✅ Slot release logic working correctly

**Acceptance Criteria:**
- Users can view their appointments with proper filtering
- Appointments can be cancelled with slot release
- Doctors can mark appointments as completed
- Invalid state transitions are rejected
- All edge cases handled

**Testing:**
- Unit tests for all appointment operations
- Integration tests for state transitions
- Edge case testing (cancelling already cancelled appointment)
- Permission testing

---

### Sprint 5: Integration, Testing & Deployment (Weeks 9-10)

**Goal**: Integrate frontend, perform end-to-end testing, and prepare for deployment.

#### Week 9: Frontend Integration & E2E Testing

**Tasks:**

1. **Frontend Setup (React)**
   - [ ] Create React project structure
   - [ ] Set up routing (React Router)
   - [ ] Configure API clients
     - [ ] axios instance for Django service
     - [ ] axios instance for Spring Boot service
     - [ ] JWT token management (store, refresh, attach to requests)
   - [ ] Set up state management (Redux/Context API)
   - [ ] Create authentication context/provider

2. **Frontend Pages Implementation**
   - [ ] Login/Register page
     - [ ] Form validation
     - [ ] API integration
     - [ ] Token storage
   - [ ] Hospital list page (for super admin)
     - [ ] Doctor search page
     - [ ] Doctor detail page with reviews
     - [ ] Slot selection page
     - [ ] Booking confirmation page
     - [ ] My Appointments page
   - [ ] Create reusable components (Header, Footer, Loading, Error)

3. **End-to-End Testing**
   - [ ] Test complete user flows:
     - [ ] User registration → Login → Search doctor → View slots → Book appointment
     - [ ] Hospital admin → Onboard doctor → Doctor creates slots
     - [ ] Patient → Book appointment → Cancel appointment
     - [ ] Doctor → View appointments → Complete appointment
   - [ ] Test error scenarios:
     - [ ] Concurrent booking attempts
     - [ ] Invalid slot selection
     - [ ] Permission errors
   - [ ] Create E2E test suite (Cypress/Playwright)

4. **API Integration Testing**
   - [ ] Test Django → Spring Boot integration
     - [ ] JWT token validation
     - [ ] User ID consistency
   - [ ] Test Redis → Database consistency
     - [ ] Verify locks are released properly
     - [ ] Verify no orphaned locks

5. **Performance Testing**
   - [ ] Load test doctor search endpoint (1000+ concurrent users)
   - [ ] Load test booking endpoint (100 concurrent bookings)
   - [ ] Test database query performance
   - [ ] Test Redis performance under load
   - [ ] Identify and fix bottlenecks

**Deliverables:**
- ✅ Frontend integrated with both backend services
- ✅ Complete user flows working end-to-end
- ✅ E2E test suite created
- ✅ Performance benchmarks established

**Acceptance Criteria:**
- All user flows work without errors
- Frontend handles API errors gracefully
- E2E tests pass consistently
- Performance meets SLOs (90% under 200ms)

**Testing:**
- Manual E2E testing
- Automated E2E tests
- Performance testing
- Security testing (OWASP Top 10)

---

#### Week 10: Documentation, Deployment & Handoff

**Tasks:**

1. **Documentation**
   - [ ] Update API documentation (OpenAPI/Swagger)
   - [ ] Create deployment guide
   - [ ] Create developer setup guide
   - [ ] Document environment variables
   - [ ] Create runbook for common operations
   - [ ] Document troubleshooting steps

2. **Deployment Preparation**
   - [ ] Set up staging environment
     - [ ] Configure staging databases
     - [ ] Set up staging Redis
     - [ ] Configure environment variables
   - [ ] Create deployment scripts
     - [ ] Database migration scripts
     - [ ] Service deployment scripts
   - [ ] Set up monitoring and logging
     - [ ] Configure application logs
     - [ ] Set up error tracking (Sentry/ELK)
     - [ ] Configure health checks
   - [ ] Set up CI/CD pipeline
     - [ ] Automated testing on PR
     - [ ] Automated deployment to staging
     - [ ] Manual approval for production

3. **Security Review**
   - [ ] Security audit of authentication flow
   - [ ] Review JWT token handling
   - [ ] Review database access patterns
   - [ ] Review API permissions
   - [ ] Penetration testing (if time permits)

4. **Production Deployment**
   - [ ] Deploy to staging
   - [ ] Run smoke tests on staging
   - [ ] Get stakeholder approval
   - [ ] Deploy to production (blue-green deployment)
   - [ ] Monitor production metrics
   - [ ] Create production runbook

5. **Knowledge Transfer**
   - [ ] Code review sessions
   - [ ] Architecture walkthrough
   - [ ] Demo to stakeholders
   - [ ] Create handoff document

**Deliverables:**
- ✅ Complete documentation
- ✅ Staging environment deployed
- ✅ Production deployment ready
- ✅ Monitoring and alerting configured

**Acceptance Criteria:**
- All documentation complete and reviewed
- Staging environment matches production setup
- Deployment process documented and tested
- Team trained on system operation

**Testing:**
- Deployment testing on staging
- Smoke tests on production
- Monitoring validation

---

## Dependencies & Prerequisites

### External Dependencies
- **PostgreSQL**: Database servers for both services
- **Redis**: Caching and distributed locking
- **Docker**: Containerization for local development
- **GitHub**: Version control and CI/CD

### Internal Dependencies
- Django service must be deployed before Spring Boot (for JWT validation)
- Database schemas must be migrated before API deployment
- Redis must be available before booking service deployment

### Team Dependencies
- Backend developers need database access
- DevOps support needed for infrastructure setup
- QA team needed for testing from Sprint 3 onwards

## Risk Mitigation

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| JWT token validation issues between services | High | Medium | Early integration testing, shared secret/key management |
| Concurrency issues in booking | High | High | Extensive load testing, Redis locking, optimistic locking |
| Database performance with large datasets | Medium | Medium | Proper indexing, query optimization, load testing |
| Redis single point of failure | Medium | Low | Redis cluster setup, fallback mechanisms |

### Schedule Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Scope creep | High | Medium | Strict sprint boundaries, change request process |
| Team member unavailability | Medium | Low | Knowledge sharing, pair programming, documentation |
| Integration delays | High | Medium | Early integration testing, API contracts defined upfront |

### Quality Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Insufficient testing | High | Medium | Test-driven development, mandatory test coverage |
| Security vulnerabilities | High | Low | Security review, penetration testing, code audits |

## Testing Strategy

### Unit Testing
- **Target Coverage**: 80% for business logic
- **Tools**: pytest (Django), JUnit (Spring Boot)
- **Focus**: Service layer, business logic, utilities

### Integration Testing
- **Target Coverage**: 60% for API endpoints
- **Tools**: Django TestClient, Spring Boot Test
- **Focus**: API endpoints, database operations, external service integration

### End-to-End Testing
- **Coverage**: Critical user flows
- **Tools**: Cypress or Playwright
- **Focus**: Complete user journeys, cross-service integration

### Performance Testing
- **Tools**: Apache JMeter or k6
- **Scenarios**: 
  - Doctor search under load (1000 concurrent users)
  - Booking concurrency (100 concurrent bookings)
  - Database query performance

### Security Testing
- **Tools**: OWASP ZAP, manual penetration testing
- **Focus**: Authentication, authorization, input validation, SQL injection

## Definition of Done

For each user story/task to be considered complete:

- [ ] Code implemented and reviewed
- [ ] Unit tests written and passing (80% coverage)
- [ ] Integration tests written and passing
- [ ] Code follows style guidelines (linted)
- [ ] API documentation updated
- [ ] Manual testing completed
- [ ] No critical bugs
- [ ] Deployed to staging (if applicable)
- [ ] Product owner acceptance

## Success Metrics

### Development Metrics
- **Code Coverage**: >80% for business logic
- **Build Success Rate**: >95%
- **Test Execution Time**: <10 minutes for full suite
- **Code Review Turnaround**: <24 hours

### System Metrics (Post-Deployment)
- **API Availability**: >99.5%
- **API Latency**: 90% under 200ms
- **Booking Success Rate**: >95%
- **Error Rate**: <1%

### Team Metrics
- **Sprint Velocity**: Consistent across sprints
- **Bug Escape Rate**: <5% to production
- **Technical Debt**: Tracked and addressed

## Post-Phase 1 Activities

### Immediate (Week 11)
- Production monitoring and bug fixes
- Performance optimization based on real traffic
- User feedback collection

### Short-term (Weeks 12-14)
- Phase 2 planning (Payment integration)
- Technical debt cleanup
- Documentation improvements

### Long-term
- Phase 3 planning (EHR system)
- Scalability improvements
- Feature enhancements based on user feedback

## References

### Internal References
- Phase 1 Architecture Document: `/healthcare/hospital-management-system/phase-1-architecture/README.md`
- Django REST Framework Documentation: [DRF Docs](https://www.django-rest-framework.org/)
- Spring Boot Documentation: [Spring Boot Docs](https://docs.spring.io/spring-boot/reference/index.html)

### External References
- PostgreSQL Performance Tuning: [PostgreSQL Docs](https://www.postgresql.org/docs/current/performance-tips.html)
- Redis Best Practices: [Redis Docs](https://redis.io/docs/manual/patterns/)
- JWT Best Practices: [JWT.io](https://jwt.io/introduction)

