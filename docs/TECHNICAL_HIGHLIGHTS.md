# Technical Highlights

This document summarizes the engineering work represented by the private CRM in a public, source-safe way.

## Product Modeling

The application is centered on a clean relational model:

- Traveler profile data belongs to the person.
- Departure-level data belongs to the trip.
- Workflow state belongs to the registration between a person and a trip.

This supports repeat travelers, multi-trip history, trip-specific contracts, trip-specific notes, and operational status tracking without duplicating customer profiles.

## Frontend Engineering

Key frontend work:

- React and TypeScript single-page application.
- Protected routes for authenticated users.
- Admin-only route guards.
- Directory views with search, sorting, and alternate visual layouts.
- Detail views for both people and trips.
- Registration drawer for focused operational edits.
- Client-side form validation.
- Import review UI with mapping, row status, and confirmation behavior.
- Export actions tied to the user's current view or selected trip.

## Backend Engineering

Key backend work:

- FastAPI REST API organized by domain.
- SQLAlchemy models for people, trips, registrations, contact logs, users, deleted contracts, reminders, and import drafts.
- Pydantic request and response validation.
- Token-based authentication.
- Role-aware admin endpoints.
- Service layer for reusable business operations.
- File upload handling for contracts and itineraries.
- Soft-delete and restore workflow.
- Audit metadata for important record changes.

## Import And Export Design

The import flow is intentionally conservative:

1. Parse spreadsheet input.
2. Map source columns to application fields.
3. Normalize incoming values.
4. Validate required information.
5. Flag invalid records.
6. Detect possible duplicates.
7. Let the user review the preview.
8. Persist only after confirmation.

This helps prevent accidental duplicate creation and reduces cleanup after bulk data entry.

Exports support operational handoff and review by generating CSV or XLSX files from relevant people, trips, or trip-member views.

## Reliability And Safety

Safety-oriented features:

- Soft deletes instead of immediate permanent deletion.
- Admin recovery area for deleted people, trips, registrations, and contracts.
- Unique person-trip registration relationship to prevent duplicate enrollments for the same trip.
- Contract-aware status behavior so document state and registration state stay aligned.
- Persistent upload storage separated from application containers.

## Deployment

The private project is Docker-ready:

- Frontend container serves built React assets through nginx.
- Backend container runs the FastAPI API.
- Frontend proxies API requests to the backend.
- Persistent data is stored outside the container images.

The public documentation intentionally describes this pattern without publishing real production configuration.
