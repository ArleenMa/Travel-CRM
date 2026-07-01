# Architecture Overview

This document describes the system at a portfolio-safe level. It explains how the application is structured without exposing private source code, production settings, credentials, or company data.

## High-Level Shape

```mermaid
flowchart TB
    Staff["Operations Staff"] --> Web["React SPA"]
    Admin["Admin User"] --> Web

    Web --> Router["Client Routing"]
    Router --> AuthUI["Auth Context + Guards"]
    AuthUI --> APIClient["API Client"]
    APIClient --> Nginx["Frontend Container / Reverse Proxy"]
    Nginx --> FastAPI["FastAPI Application"]

    FastAPI --> Routers["REST Routers"]
    Routers --> Schemas["Pydantic Schemas"]
    Routers --> Services["Domain Services"]
    Services --> ORM["SQLAlchemy Models"]
    ORM --> DB[("Relational Database")]
    Services --> Uploads["Upload Storage"]

    subgraph Frontend
        Web
        Router
        AuthUI
        APIClient
    end

    subgraph Backend
        FastAPI
        Routers
        Schemas
        Services
        ORM
    end
```

## Frontend

The frontend is a React and TypeScript single-page application. It uses route guards to separate authenticated app screens from the login page and admin-only areas.

Primary screens:

- People directory
- Person detail
- Trip directory
- Trip detail
- User management
- Deleted records recovery

Common frontend responsibilities:

- Search and sorting controls.
- List and card presentation modes.
- Form validation before API submission.
- Protected routes for signed-in users.
- Admin-only route protection.
- Import review and export initiation.
- Registration drawer for person-trip operational details.

## Backend

The backend is a FastAPI application organized around domain routers and services.

Primary API areas:

- Authentication
- Users
- People
- Trips
- Registrations
- Contact logs
- Import/export
- Deleted records
- Notifications/reminders

Service responsibilities:

- Authentication and password handling.
- Audit metadata updates.
- Import parsing, mapping, validation, and duplicate detection.
- Contract and itinerary file handling.
- Soft-delete and restore behavior.
- Trip reminder synchronization.

## Data Storage

The application uses a relational model with a central many-to-many relationship:

- One person can join many trips.
- One trip can include many people.
- The registration record between the two stores operational details.

```mermaid
erDiagram
    PEOPLE ||--o{ REGISTRATIONS : "has"
    TRIPS ||--o{ REGISTRATIONS : "has"
    REGISTRATIONS ||--o{ CONTACT_LOGS : "records"
    USERS ||--o{ PEOPLE : "creates_updates_deletes"
    USERS ||--o{ TRIPS : "creates_updates_deletes"
    USERS ||--o{ REGISTRATIONS : "creates_updates_deletes"

    PEOPLE {
        int id
        string name
        string contact_fields
        string profile_fields
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    TRIPS {
        int id
        string route
        string dates
        string guide
        string itinerary_file
        string operational_notes
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    REGISTRATIONS {
        int id
        int person_id
        int trip_id
        string status
        string contract_file
        string emergency_contact
        string special_needs
        string custom_fields
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    CONTACT_LOGS {
        int id
        int registration_id
        string note
        datetime created_at
        datetime updated_at
    }

    USERS {
        int id
        string username
        string role
    }
```

## Deployment Shape

```mermaid
flowchart LR
    Browser["Browser"] --> Frontend["Nginx + Built React Assets"]
    Frontend -->|/api| Backend["FastAPI Container"]
    Backend --> Data["Persistent Data Volume"]
    Data --> Database["Database File"]
    Data --> Contracts["Contract Uploads"]
    Data --> Itineraries["Itinerary Uploads"]

    subgraph Docker Compose
        Frontend
        Backend
        Data
    end
```

The public description only documents the deployment pattern. It does not include real server IPs, domains, production secrets, private repository URLs, or uploaded files.

## Request Lifecycle

```mermaid
sequenceDiagram
    actor User
    participant SPA as React SPA
    participant API as FastAPI
    participant Service as Domain Service
    participant DB as Database

    User->>SPA: Perform an action
    SPA->>SPA: Validate local form state
    SPA->>API: Send authenticated request
    API->>API: Validate token and request schema
    API->>Service: Run domain operation
    Service->>DB: Read/write records
    DB-->>Service: Return persisted data
    Service-->>API: Return domain result
    API-->>SPA: Return response
    SPA-->>User: Refresh view and feedback
```
