# Workflow Guide

This guide summarizes the important user workflows in the private CRM. It is written for a public portfolio reader, so examples are generalized and do not include private records.

## 1. Create A Traveler Profile

```mermaid
flowchart TD
    Start["Open People"] --> Add["Add person"]
    Add --> Details["Enter profile and contact details"]
    Details --> Validate["Validate required fields"]
    Validate --> Save["Save profile"]
    Save --> Profile["Open person detail page"]
```

Purpose:

- Gives staff one searchable home for traveler contact information.
- Keeps profile-level data separate from trip-specific operational data.

## 2. Add A Traveler To A Trip

```mermaid
flowchart TD
    Person["Open person detail"] --> Choose["Choose Add to Trip"]
    Choose --> Trip["Select trip"]
    Trip --> Registration["Enter registration details"]
    Registration --> Status["Set operational status"]
    Status --> Save["Save registration"]
    Save --> Drawer["Open registration drawer for follow-up"]
```

The registration stores details that can vary by trip, such as:

- Number of travelers in the party.
- Custom guest names.
- Registration status.
- Contract file.
- Emergency contact.
- Special needs.
- Flight or payment-related operational notes.
- Contact history.

## 3. Manage A Trip

```mermaid
flowchart TD
    Trips["Open Trips"] --> Select["Select trip"]
    Select --> Edit["Edit route, dates, guide, notes, and reminders"]
    Edit --> Itinerary["Upload or replace itinerary"]
    Itinerary --> Travelers["Review traveler list"]
    Travelers --> Status["Check status summary"]
    Status --> Export["Export relevant traveler list"]
```

The trip detail page is the operational workspace for a single departure.

## 4. Contact Follow-Up

```mermaid
sequenceDiagram
    actor Staff
    participant PersonPage as Person Detail
    participant Drawer as Registration Drawer
    participant API as API
    participant DB as Database

    Staff->>PersonPage: Open traveler
    Staff->>Drawer: Open trip registration
    Staff->>Drawer: Add contact note
    Drawer->>API: Save contact log
    API->>DB: Store note with registration
    DB-->>API: Saved log
    API-->>Drawer: Updated contact history
```

Contact logs are attached to the registration rather than only the person. That makes follow-up history specific to the trip context.

## 5. Import People From A Spreadsheet

```mermaid
flowchart TD
    Upload["Upload CSV/XLSX"] --> Parse["Parse file"]
    Parse --> Mapping["Map spreadsheet columns"]
    Mapping --> Normalize["Normalize row data"]
    Normalize --> Validate["Flag invalid rows"]
    Validate --> Match["Detect duplicates"]
    Match --> Preview["Preview New / Duplicate / Invalid rows"]
    Preview --> Resolve["Resolve row issues"]
    Resolve --> Confirm["Confirm final import"]
    Confirm --> Save["Persist approved records"]
```

Design goals:

- Avoid blind bulk inserts.
- Let users see what will happen before data changes.
- Reduce duplicate customer profiles.
- Preserve a draft review state when staff need to pause.

## 6. Export Operational Lists

```mermaid
flowchart LR
    View["Current filtered view"] --> Export["Export action"]
    Export --> Scope["Apply current search/filter scope"]
    Scope --> Format["CSV or XLSX"]
    Format --> File["Download dated export file"]
```

Exports are used for operational review, backups before major edits, and trip-specific traveler lists.

## 7. Soft Delete And Restore

```mermaid
stateDiagram-v2
    [*] --> Active
    Active --> SoftDeleted: user deletes record
    SoftDeleted --> Restored: admin restores
    Restored --> Active
    SoftDeleted --> PermanentlyRemoved: admin confirms permanent delete
    PermanentlyRemoved --> [*]
```

Soft deletes reduce the risk of accidental data loss. Admins can review deleted people, trips, registrations, and contract records before deciding whether to restore or permanently remove them.

## 8. Admin User Management

Admin-only pages support:

- Creating staff accounts.
- Assigning user/admin roles.
- Restricting sensitive views.
- Keeping deleted-record recovery away from standard users.
