# Database Design

**Document:** `docs/05-database/database-design.md`  
**Status:** Draft  
**Version:** 1.0  
**Database:** PostgreSQL  
**ORM:** TBD  
**Migration Tool:** Alembic  
**Backend:** FastAPI  
**Primary Key Strategy:** UUID  
**Currency:** INR (`INR`) for the initial India-first release

---

## 1. Purpose

This document defines the database architecture and data model for the Event Planner application.

It serves as the source of truth for:

- Database entities
- Table structures
- Relationships
- Primary and foreign keys
- Constraints
- Indexes
- Enumerations
- Data ownership and authorization
- Timestamp conventions
- Deletion behavior
- Financial data storage
- File metadata
- Subscription and payment records
- Database migration strategy

The database must support the current MVP while avoiding design decisions that would make future expansion unnecessarily difficult.

---

# 2. Database Goals

The database must be:

1. **Relational**
   - PostgreSQL is the system of record.

2. **Secure**
   - Users must only access data they are authorized to access.

3. **Consistent**
   - Referential integrity and database constraints must prevent invalid relationships.

4. **Extensible**
   - The schema should support future features such as multiple events, collaboration, budgeting and notifications.

5. **Auditable**
   - Important records should contain creation and modification timestamps.

6. **Migration-safe**
   - All schema changes must be managed through version-controlled migrations.

7. **Backend-authoritative**
   - Business rules involving authentication, ownership, subscriptions and payments must not depend solely on frontend logic.

---

# 3. Database Architecture

The initial database uses PostgreSQL.

```text
                         ┌──────────────┐
                         │    users     │
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │    events    │
                         └──────┬───────┘
                                │
          ┌─────────────┬───────┼────────┬─────────────┐
          │             │       │        │             │
          ▼             ▼       ▼        ▼             ▼
       tasks         vendors  guests    ideas      event_files
          │             │
          │             ▼
          │       vendor_payments
          │
          │
          └──────────────────────────────┐
                                         │
                                         ▼
                                  notifications


users
  │
  ├── subscriptions
  │       │
  │       └── payments
  │
  ├── sessions
  │
  └── audit_logs
```

---

# 4. Design Principles

## 4.1 UUID Primary Keys

All major entities should use UUIDs rather than sequential integer IDs.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

Advantages:

- Prevents predictable IDs
- Suitable for distributed systems
- Safer for public API identifiers
- Easier future scaling
- Avoids exposing record counts

PostgreSQL should generate UUIDs where appropriate.

---

## 4.2 UTC Timestamps

All timestamps stored by the backend must use UTC.

Recommended PostgreSQL type:

```sql
TIMESTAMPTZ
```

Example:

```text
created_at TIMESTAMPTZ NOT NULL
updated_at TIMESTAMPTZ NOT NULL
```

The frontend converts timestamps into the user's local timezone.

---

## 4.3 Ownership

Every event-owned resource must ultimately be associated with an event.

Examples:

```text
event
 ├── task
 ├── vendor
 ├── guest
 ├── idea
 └── file
```

The backend must verify:

```text
authenticated_user
        ↓
event membership / ownership
        ↓
requested resource
```

A user must never be able to access another user's event data simply by changing an ID in an API request.

---

# 5. Entity Overview

The primary MVP entities are:

| Entity             | Purpose                                   |
| ------------------ | ----------------------------------------- |
| `users`            | User accounts                             |
| `events`           | Events being planned                      |
| `event_members`    | Event membership and future collaboration |
| `tasks`            | Event planning tasks                      |
| `vendors`          | Vendor records                            |
| `vendor_payments`  | Vendor payment history                    |
| `guests`           | Guest records                             |
| `ideas`            | Saved planning inspiration                |
| `idea_attachments` | Images/files attached to ideas            |
| `event_files`      | Event-level uploaded files                |
| `subscriptions`    | Subscription state                        |
| `payments`         | Payment transactions                      |
| `sessions`         | Authenticated sessions                    |
| `notifications`    | User notifications                        |
| `audit_logs`       | Security and important activity records   |

---

# 6. Entity Relationship Model

```text
users
  │
  ├───────────────┐
  │               │
  ▼               ▼
events        subscriptions
  │               │
  │               ▼
  │            payments
  │
  ├────────────── event_members
  │
  ├────────────── tasks
  │
  ├────────────── vendors
  │                  │
  │                  ▼
  │            vendor_payments
  │
  ├────────────── guests
  │
  ├────────────── ideas
  │                  │
  │                  ▼
  │            idea_attachments
  │
  └────────────── event_files


users
  ├── sessions
  ├── notifications
  └── audit_logs
```

---

# 7. Table: `users`

Stores application user accounts.

## Purpose

A user represents a person who can authenticate and manage events.

## Columns

| Column              | Type         | Nullable | Default   | Description             |
| ------------------- | ------------ | -------: | --------- | ----------------------- |
| `id`                | UUID         |       No | generated | Primary key             |
| `mobile_number`     | VARCHAR(20)  |       No | —         | Unique mobile number    |
| `name`              | VARCHAR(150) |      Yes | NULL      | User's display name     |
| `email`             | VARCHAR(255) |      Yes | NULL      | Optional email          |
| `profile_image_url` | TEXT         |      Yes | NULL      | Profile image           |
| `is_active`         | BOOLEAN      |       No | TRUE      | Account status          |
| `created_at`        | TIMESTAMPTZ  |       No | now       | Creation timestamp      |
| `updated_at`        | TIMESTAMPTZ  |       No | now       | Last modification       |
| `deleted_at`        | TIMESTAMPTZ  |      Yes | NULL      | Soft deletion timestamp |

## Constraints

```text
mobile_number UNIQUE
```

The mobile number should be normalized before storage.

Example:

```text
+919876543210
```

rather than storing multiple representations such as:

```text
9876543210
+91 9876543210
0919876543210
```

---

# 8. Table: `events`

Represents an event being planned.

A user can eventually manage multiple events.

## Columns

| Column        | Type         | Nullable | Description             |
| ------------- | ------------ | -------: | ----------------------- |
| `id`          | UUID         |       No | Primary key             |
| `created_by`  | UUID         |       No | User who created event  |
| `name`        | VARCHAR(200) |       No | Event name              |
| `event_type`  | VARCHAR(50)  |       No | Event category          |
| `event_date`  | DATE         |       No | Main event date         |
| `location`    | VARCHAR(255) |      Yes | Event location          |
| `description` | TEXT         |      Yes | Optional description    |
| `image_url`   | TEXT         |      Yes | Event image             |
| `currency`    | CHAR(3)      |       No | Financial currency      |
| `created_at`  | TIMESTAMPTZ  |       No | Creation timestamp      |
| `updated_at`  | TIMESTAMPTZ  |       No | Modification timestamp  |
| `deleted_at`  | TIMESTAMPTZ  |      Yes | Soft deletion timestamp |

## Foreign Keys

```text
created_by → users.id
```

## Event Types

Initial values:

```text
WEDDING
ENGAGEMENT
BIRTHDAY
ANNIVERSARY
BABY_SHOWER
PRIVATE_EVENT
OTHER
```

The application should store stable machine-readable values.

Example:

```text
WEDDING
```

rather than:

```text
Wedding
```

---

# 9. Table: `event_members`

Although the MVP initially focuses on a single primary organizer, the schema should support multiple users per event.

This avoids requiring a major database redesign when collaboration is introduced.

## Columns

| Column       | Type        | Nullable | Description        |
| ------------ | ----------- | -------: | ------------------ |
| `id`         | UUID        |       No | Primary key        |
| `event_id`   | UUID        |       No | Event              |
| `user_id`    | UUID        |       No | User               |
| `role`       | VARCHAR(30) |       No | Membership role    |
| `created_at` | TIMESTAMPTZ |       No | Creation timestamp |

## Roles

Initial supported role:

```text
OWNER
```

Future roles:

```text
EDITOR
VIEWER
```

## Constraints

```text
UNIQUE(event_id, user_id)
```

This prevents duplicate memberships.

---

# 10. Table: `tasks`

Stores planning tasks associated with an event.

## Columns

| Column         | Type         | Nullable | Description            |
| -------------- | ------------ | -------: | ---------------------- |
| `id`           | UUID         |       No | Primary key            |
| `event_id`     | UUID         |       No | Event                  |
| `title`        | VARCHAR(200) |       No | Task title             |
| `description`  | TEXT         |      Yes | Task details           |
| `due_date`     | DATE         |      Yes | Task deadline          |
| `priority`     | VARCHAR(20)  |       No | Priority               |
| `category`     | VARCHAR(50)  |      Yes | Planning category      |
| `status`       | VARCHAR(20)  |       No | Current status         |
| `completed_at` | TIMESTAMPTZ  |      Yes | Completion timestamp   |
| `created_at`   | TIMESTAMPTZ  |       No | Creation timestamp     |
| `updated_at`   | TIMESTAMPTZ  |       No | Modification timestamp |
| `deleted_at`   | TIMESTAMPTZ  |      Yes | Soft deletion          |

## Priority

```text
LOW
MEDIUM
HIGH
```

## Status

```text
PENDING
COMPLETED
```

## Categories

```text
VENUE
CATERING
PHOTOGRAPHY
DECORATION
INVITATIONS
CLOTHING
BEAUTY
TRAVEL
OTHER
```

---

# 11. Table: `vendors`

Stores vendors associated with an event.

## Columns

| Column           | Type          | Nullable | Description            |
| ---------------- | ------------- | -------: | ---------------------- |
| `id`             | UUID          |       No | Primary key            |
| `event_id`       | UUID          |       No | Event                  |
| `name`           | VARCHAR(200)  |       No | Vendor name            |
| `category`       | VARCHAR(50)   |       No | Vendor category        |
| `phone`          | VARCHAR(20)   |      Yes | Contact number         |
| `email`          | VARCHAR(255)  |      Yes | Email                  |
| `estimated_cost` | NUMERIC(12,2) |      Yes | Expected cost          |
| `actual_cost`    | NUMERIC(12,2) |      Yes | Final cost             |
| `notes`          | TEXT          |      Yes | Notes                  |
| `created_at`     | TIMESTAMPTZ   |       No | Creation timestamp     |
| `updated_at`     | TIMESTAMPTZ   |       No | Modification timestamp |
| `deleted_at`     | TIMESTAMPTZ   |      Yes | Soft deletion          |

Money must **not** be stored using floating-point types.

Use:

```sql
NUMERIC(12,2)
```

---

# 12. Table: `vendor_payments`

Vendor payment history should be stored separately rather than maintaining only a single `amount_paid` value.

This provides a foundation for future financial reporting.

## Columns

| Column           | Type          | Nullable | Description        |
| ---------------- | ------------- | -------: | ------------------ |
| `id`             | UUID          |       No | Primary key        |
| `vendor_id`      | UUID          |       No | Vendor             |
| `amount`         | NUMERIC(12,2) |       No | Payment amount     |
| `payment_date`   | DATE          |       No | Payment date       |
| `payment_method` | VARCHAR(30)   |      Yes | Payment method     |
| `reference`      | VARCHAR(100)  |      Yes | Payment reference  |
| `notes`          | TEXT          |      Yes | Notes              |
| `created_at`     | TIMESTAMPTZ   |       No | Creation timestamp |

## Payment Methods

Initial values may include:

```text
CASH
UPI
BANK_TRANSFER
CARD
OTHER
```

The total paid amount should be calculated from payment records.

```text
total_paid = SUM(vendor_payments.amount)
```

---

# 13. Table: `guests`

Stores people invited to an event.

## Columns

| Column            | Type         | Nullable | Description            |
| ----------------- | ------------ | -------: | ---------------------- |
| `id`              | UUID         |       No | Primary key            |
| `event_id`        | UUID         |       No | Event                  |
| `name`            | VARCHAR(200) |       No | Guest name             |
| `phone`           | VARCHAR(20)  |      Yes | Phone number           |
| `group_name`      | VARCHAR(100) |      Yes | Family/group           |
| `rsvp_status`     | VARCHAR(30)  |       No | RSVP status            |
| `attendee_count`  | INTEGER      |       No | Number attending       |
| `meal_preference` | VARCHAR(50)  |      Yes | Meal preference        |
| `notes`           | TEXT         |      Yes | Notes                  |
| `created_at`      | TIMESTAMPTZ  |       No | Creation timestamp     |
| `updated_at`      | TIMESTAMPTZ  |       No | Modification timestamp |
| `deleted_at`      | TIMESTAMPTZ  |      Yes | Soft deletion          |

## RSVP Status

```text
PENDING
ATTENDING
NOT_ATTENDING
MAYBE
```

## Constraints

```text
attendee_count >= 0
```

---

# 14. Table: `ideas`

Stores inspiration and planning ideas.

An idea may contain text, an external URL, or attachments.

## Columns

| Column         | Type         | Nullable | Description            |
| -------------- | ------------ | -------: | ---------------------- |
| `id`           | UUID         |       No | Primary key            |
| `event_id`     | UUID         |       No | Event                  |
| `title`        | VARCHAR(200) |      Yes | Idea title             |
| `description`  | TEXT         |      Yes | Idea notes             |
| `category`     | VARCHAR(50)  |      Yes | Idea category          |
| `external_url` | TEXT         |      Yes | External reference     |
| `created_at`   | TIMESTAMPTZ  |       No | Creation timestamp     |
| `updated_at`   | TIMESTAMPTZ  |       No | Modification timestamp |
| `deleted_at`   | TIMESTAMPTZ  |      Yes | Soft deletion          |

At least one meaningful content field should be supplied.

Possible content:

```text
description
external_url
attachment
```

---

# 15. Table: `idea_attachments`

Stores metadata for files associated with ideas.

Actual file content should be stored in object storage rather than PostgreSQL.

## Columns

| Column              | Type         | Nullable | Description        |
| ------------------- | ------------ | -------: | ------------------ |
| `id`                | UUID         |       No | Primary key        |
| `idea_id`           | UUID         |       No | Related idea       |
| `storage_key`       | TEXT         |       No | Object storage key |
| `original_filename` | VARCHAR(255) |       No | Original filename  |
| `mime_type`         | VARCHAR(100) |       No | MIME type          |
| `file_size`         | BIGINT       |       No | Size in bytes      |
| `created_at`        | TIMESTAMPTZ  |       No | Upload timestamp   |

---

# 16. Table: `event_files`

Stores event-level uploaded file metadata.

Examples:

- Event cover image
- Future documents
- Future contracts
- Other event assets

## Columns

| Column              | Type         | Nullable | Description        |
| ------------------- | ------------ | -------: | ------------------ |
| `id`                | UUID         |       No | Primary key        |
| `event_id`          | UUID         |       No | Event              |
| `storage_key`       | TEXT         |       No | Object storage key |
| `original_filename` | VARCHAR(255) |       No | Original filename  |
| `mime_type`         | VARCHAR(100) |       No | MIME type          |
| `file_size`         | BIGINT       |       No | Size in bytes      |
| `created_at`        | TIMESTAMPTZ  |       No | Upload timestamp   |

---

# 17. Table: `subscriptions`

Stores the application's subscription state.

Subscription status must be determined by the backend.

## Columns

| Column                     | Type         | Nullable | Description              |
| -------------------------- | ------------ | -------: | ------------------------ |
| `id`                       | UUID         |       No | Primary key              |
| `user_id`                  | UUID         |       No | Subscriber               |
| `provider`                 | VARCHAR(30)  |       No | Payment provider         |
| `provider_customer_id`     | VARCHAR(255) |      Yes | Provider customer ID     |
| `provider_subscription_id` | VARCHAR(255) |      Yes | Provider subscription ID |
| `plan_code`                | VARCHAR(50)  |       No | Subscription plan        |
| `status`                   | VARCHAR(30)  |       No | Subscription state       |
| `trial_started_at`         | TIMESTAMPTZ  |      Yes | Trial start              |
| `trial_ends_at`            | TIMESTAMPTZ  |      Yes | Trial end                |
| `current_period_start`     | TIMESTAMPTZ  |      Yes | Billing period start     |
| `current_period_end`       | TIMESTAMPTZ  |      Yes | Billing period end       |
| `cancelled_at`             | TIMESTAMPTZ  |      Yes | Cancellation time        |
| `created_at`               | TIMESTAMPTZ  |       No | Creation                 |
| `updated_at`               | TIMESTAMPTZ  |       No | Modification             |

## Status

```text
TRIAL
ACTIVE
PAST_DUE
CANCELLED
EXPIRED
```

## Important Rule

The frontend must never be considered the source of truth for subscription access.

The backend determines:

```text
Can user access application?
```

based on the subscription/trial state.

---

# 18. Table: `payments`

Stores payment transaction records.

A payment and a subscription are separate concepts.

One subscription may have multiple payments.

## Columns

| Column                | Type          | Nullable | Description             |
| --------------------- | ------------- | -------: | ----------------------- |
| `id`                  | UUID          |       No | Primary key             |
| `user_id`             | UUID          |       No | User                    |
| `subscription_id`     | UUID          |      Yes | Related subscription    |
| `provider`            | VARCHAR(30)   |       No | Payment provider        |
| `provider_payment_id` | VARCHAR(255)  |       No | Provider transaction ID |
| `provider_order_id`   | VARCHAR(255)  |      Yes | Provider order ID       |
| `amount`              | NUMERIC(12,2) |       No | Amount                  |
| `currency`            | CHAR(3)       |       No | Currency                |
| `status`              | VARCHAR(30)   |       No | Payment status          |
| `paid_at`             | TIMESTAMPTZ   |      Yes | Payment timestamp       |
| `created_at`          | TIMESTAMPTZ   |       No | Creation timestamp      |

## Payment Status

```text
PENDING
SUCCESS
FAILED
REFUNDED
```

Provider webhook events must update payment state on the backend.

---

# 19. Table: `sessions`

Stores authenticated application sessions if server-managed sessions are used.

## Columns

| Column         | Type        | Nullable | Description          |
| -------------- | ----------- | -------: | -------------------- |
| `id`           | UUID        |       No | Primary key          |
| `user_id`      | UUID        |       No | User                 |
| `token_hash`   | TEXT        |       No | Hashed session token |
| `expires_at`   | TIMESTAMPTZ |       No | Expiration           |
| `revoked_at`   | TIMESTAMPTZ |      Yes | Revocation time      |
| `created_at`   | TIMESTAMPTZ |       No | Creation             |
| `last_used_at` | TIMESTAMPTZ |      Yes | Last activity        |

Raw session tokens should not be stored in plaintext.

---

# 20. OTP Data

OTP verification is an authentication mechanism rather than a permanent business entity.

The implementation may use a dedicated table such as:

```text
otp_verifications
```

if OTP lifecycle management requires server-side persistence.

Suggested fields:

| Column          | Type        |
| --------------- | ----------- |
| `id`            | UUID        |
| `mobile_number` | VARCHAR(20) |
| `otp_hash`      | TEXT        |
| `purpose`       | VARCHAR(30) |
| `attempt_count` | INTEGER     |
| `expires_at`    | TIMESTAMPTZ |
| `verified_at`   | TIMESTAMPTZ |
| `created_at`    | TIMESTAMPTZ |

OTP values must never be stored in plaintext.

The SMS provider must remain abstracted behind the backend OTP service.

---

# 21. Table: `notifications`

Stores application notifications.

## Columns

| Column       | Type         | Nullable |
| ------------ | ------------ | -------: |
| `id`         | UUID         |       No |
| `user_id`    | UUID         |       No |
| `event_id`   | UUID         |      Yes |
| `type`       | VARCHAR(50)  |       No |
| `title`      | VARCHAR(200) |       No |
| `message`    | TEXT         |       No |
| `is_read`    | BOOLEAN      |       No |
| `created_at` | TIMESTAMPTZ  |       No |
| `read_at`    | TIMESTAMPTZ  |      Yes |

Examples:

```text
TASK_DUE
TASK_OVERDUE
TRIAL_EXPIRING
SUBSCRIPTION_EXPIRED
PAYMENT_FAILED
```

---

# 22. Table: `audit_logs`

Stores security-sensitive and important system activity.

## Columns

| Column        | Type         | Nullable |
| ------------- | ------------ | -------: |
| `id`          | UUID         |       No |
| `user_id`     | UUID         |      Yes |
| `event_id`    | UUID         |      Yes |
| `action`      | VARCHAR(100) |       No |
| `entity_type` | VARCHAR(50)  |       No |
| `entity_id`   | UUID         |      Yes |
| `metadata`    | JSONB        |      Yes |
| `ip_address`  | INET         |      Yes |
| `user_agent`  | TEXT         |      Yes |
| `created_at`  | TIMESTAMPTZ  |       No |

Examples:

```text
USER_LOGIN
USER_LOGOUT
EVENT_CREATED
EVENT_DELETED
SUBSCRIPTION_CREATED
PAYMENT_COMPLETED
ACCOUNT_DELETED
```

Sensitive data should not be unnecessarily stored in audit metadata.

---

# 23. Relationships

## User → Events

```text
users 1 ──── N events
```

One user can create multiple events.

---

## Event → Tasks

```text
events 1 ──── N tasks
```

Every task belongs to exactly one event.

---

## Event → Vendors

```text
events 1 ──── N vendors
```

---

## Vendor → Payments

```text
vendors 1 ──── N vendor_payments
```

---

## Event → Guests

```text
events 1 ──── N guests
```

---

## Event → Ideas

```text
events 1 ──── N ideas
```

---

## Idea → Attachments

```text
ideas 1 ──── N idea_attachments
```

---

## Event → Files

```text
events 1 ──── N event_files
```

---

## User → Subscription

```text
users 1 ──── N subscriptions
```

The application may enforce a business rule that only one subscription is active at a time.

Historical subscription records should be retained where required for billing history.

---

## Subscription → Payments

```text
subscriptions 1 ──── N payments
```

---

# 24. Indexing Strategy

Indexes should support common application queries rather than being added indiscriminately.

## Users

```text
UNIQUE(mobile_number)
```

---

## Events

```text
INDEX(created_by)
INDEX(event_date)
```

---

## Event Members

```text
INDEX(event_id)
INDEX(user_id)
UNIQUE(event_id, user_id)
```

---

## Tasks

```text
INDEX(event_id)
INDEX(event_id, status)
INDEX(event_id, due_date)
INDEX(event_id, priority)
```

---

## Vendors

```text
INDEX(event_id)
INDEX(event_id, category)
```

---

## Vendor Payments

```text
INDEX(vendor_id)
INDEX(payment_date)
```

---

## Guests

```text
INDEX(event_id)
INDEX(event_id, rsvp_status)
INDEX(event_id, group_name)
```

---

## Ideas

```text
INDEX(event_id)
INDEX(event_id, category)
```

---

## Subscriptions

```text
INDEX(user_id)
INDEX(status)
INDEX(provider_subscription_id)
```

---

## Payments

```text
INDEX(user_id)
INDEX(subscription_id)
INDEX(provider_payment_id)
INDEX(status)
```

---

## Notifications

```text
INDEX(user_id, is_read)
INDEX(user_id, created_at)
```

---

## Audit Logs

```text
INDEX(user_id)
INDEX(event_id)
INDEX(entity_type, entity_id)
INDEX(created_at)
```

---

# 25. Soft Delete Strategy

The following user-facing entities should support soft deletion:

```text
users
events
tasks
vendors
guests
ideas
```

Soft deletion uses:

```text
deleted_at
```

Example:

```text
deleted_at = NULL
```

means active.

```text
deleted_at = 2026-10-07T10:30:00Z
```

means deleted.

Normal application queries must exclude soft-deleted records.

Example:

```sql
WHERE deleted_at IS NULL
```

Permanent deletion may be performed separately according to account deletion and data-retention policies.

---

# 26. Hard Delete Strategy

Hard deletion should be restricted.

Possible hard-delete candidates:

```text
OTP records
expired sessions
temporary files
temporary system records
```

Business data should generally use controlled deletion rather than immediate physical deletion.

---

# 27. Foreign Key Delete Behavior

Recommended behavior:

| Relationship         | Delete behavior                |
| -------------------- | ------------------------------ |
| User → Events        | RESTRICT / controlled deletion |
| Event → Tasks        | CASCADE or controlled cleanup  |
| Event → Vendors      | CASCADE or controlled cleanup  |
| Vendor → Payments    | CASCADE                        |
| Event → Guests       | CASCADE                        |
| Event → Ideas        | CASCADE                        |
| Idea → Attachments   | CASCADE                        |
| Event → Files        | CASCADE                        |
| User → Sessions      | CASCADE                        |
| User → Notifications | CASCADE                        |
| User → Audit Logs    | RETAIN according to policy     |

Application-level deletion workflows should be used for sensitive account/event deletion.

---

# 28. Data Validation

Validation should exist at multiple layers.

```text
Frontend validation
        ↓
API validation
        ↓
Business validation
        ↓
Database constraints
```

The database must remain the final integrity layer.

Examples:

### Guest count

```text
attendee_count >= 0
```

### Monetary values

```text
amount >= 0
```

### Event date

The API should validate business rules around event dates.

### Required fields

Database `NOT NULL` constraints should exist for mandatory fields.

---

# 29. Enum Strategy

For MVP, application-level enums are recommended for most business statuses.

Example:

```python
class TaskStatus(str, Enum):
    PENDING = "PENDING"
    COMPLETED = "COMPLETED"
```

Database implementation may use:

- PostgreSQL enum types, or
- constrained `VARCHAR` values.

For MVP, constrained strings can provide greater migration flexibility.

The application must use stable machine-readable values.

---

# 30. Monetary Data

All financial amounts must use fixed-precision numeric types.

Recommended:

```sql
NUMERIC(12,2)
```

Never use:

```text
FLOAT
REAL
DOUBLE PRECISION
```

for financial values.

Each event should have a currency:

```text
currency = INR
```

Future international support can introduce additional currencies.

---

# 31. File Storage

Binary files should not be stored directly inside PostgreSQL.

Recommended architecture:

```text
Next.js
   ↓
FastAPI
   ↓
Object Storage
   ↓
PostgreSQL stores metadata
```

PostgreSQL stores:

```text
storage_key
filename
mime_type
file_size
owner/event relationship
```

Object storage stores the actual file.

---

# 32. File Security

Uploaded files must be:

- Authenticated
- Authorized
- Type validated
- Size validated
- Stored using generated object keys
- Protected from unauthorized access

Example storage key:

```text
events/{event_id}/ideas/{idea_id}/{uuid}.jpg
```

Do not use the user's original filename as the storage key.

---

# 33. Event Authorization Model

Every event-owned resource must be authorized through its event.

Example:

```text
GET /api/v1/events/{event_id}/tasks
```

The backend must perform:

```text
1. Authenticate user
2. Load event
3. Verify user has access to event
4. Query tasks belonging to event
5. Return authorized data
```

It must not simply execute:

```sql
SELECT *
FROM tasks
WHERE id = :task_id;
```

without verifying ownership.

---

# 34. Multi-Event Support

Although the MVP user experience may initially focus on creating one event, the database must support:

```text
User
 ├── Event A
 ├── Event B
 └── Event C
```

This prevents a future migration from:

```text
users → single event
```

to:

```text
users → multiple events
```

The application can still simplify the MVP UI by presenting the user's primary/current event.

---

# 35. Future Collaboration

The `event_members` table intentionally exists even though multi-user collaboration is outside the MVP scope.

Future structure:

```text
Event
 ├── Owner
 ├── Editor
 └── Viewer
```

Potential future features:

- Couple collaboration
- Wedding planner access
- Family members
- Event team members
- Role-based permissions
- Comments
- Activity history

These should not be implemented in the MVP unless scope is explicitly expanded.

---

# 36. Database Naming Conventions

Use:

```text
snake_case
```

for database identifiers.

Examples:

```text
user_id
event_id
created_at
updated_at
payment_status
```

Tables should use plural nouns:

```text
users
events
tasks
vendors
guests
ideas
```

Primary keys should consistently use:

```text
id
```

Foreign keys should use:

```text
<entity>_id
```

Example:

```text
event_id
vendor_id
user_id
```

---

# 37. Timestamp Conventions

Every primary business table should contain:

```text
created_at
updated_at
```

Entities supporting soft deletion should also contain:

```text
deleted_at
```

The backend should update `updated_at` whenever a mutable record changes.

---

# 38. Migration Strategy

Alembic should manage all database schema changes.

Example migration history:

```text
001_initial_schema
002_add_event_members
003_add_vendor_payments
004_add_notifications
...
```

Developers must not make undocumented production schema changes manually.

Migration workflow:

```text
Model change
     ↓
Generate migration
     ↓
Review migration
     ↓
Run locally
     ↓
Run tests
     ↓
Run staging migration
     ↓
Run production migration
```

Production migrations must be reviewed before execution.

---

# 39. Seed Data

The application may include controlled seed data for development and testing.

Examples:

```text
Event Types
Task Categories
Vendor Categories
Idea Categories
Subscription Plans
```

Production seed data must be version-controlled.

Seed data must not contain real customer information.

---

# 40. Backup and Recovery

Production PostgreSQL must have automated backups.

The system should define:

- Backup frequency
- Retention period
- Recovery procedure
- Recovery Point Objective (RPO)
- Recovery Time Objective (RTO)

The exact values will be finalized in the infrastructure documentation.

Backups must be tested periodically.

A backup that has never been restored should not be considered fully validated.

---

# 41. Data Retention

Retention rules must be defined for:

```text
User accounts
Events
Deleted records
Payments
Audit logs
Sessions
OTP records
Notifications
Uploaded files
```

Payment and audit information may require longer retention than temporary authentication records.

Final retention periods should be documented in the security and privacy documentation.

---

# 42. Privacy Considerations

The database may contain personal information including:

```text
Mobile numbers
Email addresses
Names
Guest information
Vendor contact information
Payment metadata
```

Access to this information must be restricted.

The application should collect only information required for the product.

Sensitive information should not be unnecessarily duplicated across tables.

---

# 43. Security Requirements

The database implementation must enforce:

- Strong authentication
- Parameterized queries
- ORM/query-builder safety
- Foreign key constraints
- Authorization checks
- Secure database credentials
- Encrypted database connections
- Restricted database network access
- No database credentials in source control
- No plaintext OTP storage
- No plaintext session token storage
- Controlled production access

---

# 44. Database Environment Separation

The application must have separate database environments.

```text
Development
     ↓
Staging
     ↓
Production
```

Production data must never be copied into development environments without an approved anonymization process.

---

# 45. Recommended MVP Schema

The initial MVP database should contain:

```text
users
events
event_members
tasks
vendors
vendor_payments
guests
ideas
idea_attachments
event_files
subscriptions
payments
sessions
otp_verifications
notifications
audit_logs
```

This provides enough structure to support the defined MVP without prematurely introducing complex financial, collaboration or analytics systems.

---

# 46. Logical Data Flow

## Authentication

```text
Mobile Number
      ↓
OTP Request
      ↓
otp_verifications
      ↓
OTP Verification
      ↓
users
      ↓
sessions
```

---

## Event Planning

```text
users
   ↓
events
   ↓
┌─────────┬──────────┬─────────┬─────────┐
│         │          │         │
tasks   vendors    guests    ideas
          │                    │
          ▼                    ▼
 vendor_payments       idea_attachments
```

---

## Subscription

```text
users
   ↓
subscriptions
   ↓
payments
   ↓
provider webhooks
   ↓
subscription status
```

The backend determines whether the user can continue using the application.

---

# 47. Dashboard Data Sources

The dashboard should not require a dedicated dashboard table for MVP.

Dashboard metrics should be derived from existing entities.

Example:

```text
Planning Progress
    ↓
completed tasks / total tasks

Pending Tasks
    ↓
tasks.status = PENDING

Vendor Count
    ↓
COUNT(vendors)

Vendor Payments
    ↓
SUM(vendor_payments.amount)

Guest Count
    ↓
COUNT(guests)

Attending Guests
    ↓
SUM(attendee_count)
WHERE rsvp_status = ATTENDING
```

This avoids unnecessary denormalized data.

Caching or materialized views can be considered later if performance requires them.

---

# 48. Data Consistency Rules

The following rules must always hold:

### Rule 1

A task must belong to an existing event.

### Rule 2

A vendor must belong to an existing event.

### Rule 3

A guest must belong to an existing event.

### Rule 4

An idea must belong to an existing event.

### Rule 5

A vendor payment must belong to an existing vendor.

### Rule 6

An idea attachment must belong to an existing idea.

### Rule 7

An event member must reference an existing user and event.

### Rule 8

A payment must reference a valid user.

### Rule 9

Subscription status must be updated from trusted backend/provider events.

### Rule 10

Users must not be able to access resources belonging to events they do not have permission to access.

---

# 49. Performance Considerations

The MVP database should prioritize correctness and appropriate indexing over premature optimization.

Initial priorities:

1. Correct relationships
2. Authorization
3. Appropriate indexes
4. Efficient pagination
5. Avoiding N+1 queries
6. Proper connection pooling
7. Query monitoring

Pagination should be used for potentially large collections such as:

```text
tasks
vendors
guests
ideas
notifications
audit logs
```

---

# 50. API Pagination

Collection APIs should support pagination.

Recommended initial pattern:

```text
GET /api/v1/events/{event_id}/tasks?page=1&page_size=20
```

The API should enforce a maximum page size.

Example:

```text
default = 20
maximum = 100
```

Exact pagination strategy can be refined during API specification.

---

# 51. Transaction Requirements

Database transactions should be used whenever multiple related writes must succeed or fail together.

Example:

```text
Create subscription
      +
Create payment record
```

If one required operation fails, the transaction should be rolled back.

Another example:

```text
Create vendor
      +
Create initial payment
```

if both are introduced as one atomic operation.

---

# 52. Concurrency Considerations

The backend must account for concurrent updates.

Important areas include:

- Subscription updates
- Payment webhook processing
- Vendor payments
- Event deletion
- User account deletion

Payment webhook handlers must be idempotent.

The same provider webhook should not create duplicate payment records.

---

# 53. Idempotency

External payment operations should support idempotent processing.

Example:

```text
provider_payment_id = unique
```

This prevents duplicate payment records if a payment provider sends the same webhook multiple times.

The backend should safely process repeated webhook events.

---

# 54. Database Source of Truth

The database is authoritative for persisted application state.

The following must not be determined exclusively by frontend state:

```text
Authentication
Authorization
Event ownership
Trial status
Subscription status
Payment status
Task persistence
Vendor persistence
Guest persistence
Idea persistence
```

---

# 55. MVP Database Definition of Done

The database implementation is considered complete when:

- [ ] PostgreSQL database is provisioned
- [ ] All MVP tables are defined
- [ ] Primary keys are implemented
- [ ] Foreign keys are implemented
- [ ] Required constraints are implemented
- [ ] Required indexes are implemented
- [ ] Timestamp conventions are implemented
- [ ] Soft-delete strategy is implemented
- [ ] Alembic migrations are created
- [ ] Migrations run successfully on a clean database
- [ ] Seed data is available where required
- [ ] Database credentials are externalized
- [ ] Development and production databases are separated
- [ ] Backup strategy is configured
- [ ] Authorization model is implemented
- [ ] Payment records are idempotent
- [ ] No plaintext OTP/session secrets are stored
- [ ] Database tests cover critical constraints
- [ ] Production migration process is documented

---

# 56. Related Documentation

This document should be used together with:

```text
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/03-user-experience/user-flows.md
docs/03-user-experience/screen-inventory.md

docs/04-architecture/architecture.md
docs/06-api/api-specification.md
docs/08-security/security.md
docs/09-infrastructure/infrastructure.md
```

---

# 57. Future Database Extensions

The following are intentionally deferred:

```text
event_collaborators
budget_categories
budget_transactions
guest_groups
seating_tables
invitation_templates
invitation_recipients
vendor_reviews
vendor_bookings
calendar_events
chat_messages
comments
activity_feed
ai_recommendations
analytics_events
```

These should only be introduced when the corresponding product requirements are approved.

---

# 58. Final Database Principle

The database should remain **simple enough for the MVP but structurally ready for the product to grow**.

The most important architectural relationship is:

```text
User
  ↓
Event
  ↓
Event Resources
  ├── Tasks
  ├── Vendors
  ├── Vendor Payments
  ├── Guests
  ├── Ideas
  └── Files
```

while subscription and authentication remain user-level concerns:

```text
User
 ├── Sessions
 ├── Subscription
 │      └── Payments
 ├── Notifications
 └── Audit Logs
```

This separation keeps the core event-planning domain clean and gives the application a solid foundation for future collaboration, budgeting, invitations and other capabilities without overengineering the MVP.
