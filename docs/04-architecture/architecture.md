# System Architecture

**Document:** `docs/04-architecture/architecture.md`  
**Status:** Draft  
**Version:** 1.0  
**Architecture Style:** Modular monolith + independently deployable frontend  
**Frontend:** Next.js + TypeScript + Tailwind CSS  
**Backend:** FastAPI + Python  
**Database:** PostgreSQL  
**API:** REST / JSON  
**API Version:** `/api/v1/`  
**Deployment:** Vercel + Render + Supabase  
**Initial Market:** India

---

# 1. Purpose

This document defines the technical architecture of the Event Planner application.

It establishes:

- System components
- Application boundaries
- Communication between services
- Frontend architecture
- Backend architecture
- Database architecture
- Authentication architecture
- Subscription architecture
- File storage architecture
- External service integrations
- Deployment architecture
- Environment separation
- Security boundaries
- Observability requirements
- Scalability considerations
- Development conventions

This document is intended to be the primary technical reference for implementation.

---

# 2. Architecture Goals

The architecture must prioritize:

1. **Simplicity**
2. **Security**
3. **Maintainability**
4. **Clear separation of responsibilities**
5. **Independent deployment**
6. **Strong API contracts**
7. **Reliable authentication**
8. **Server-side authorization**
9. **Reliable subscription enforcement**
10. **Future extensibility**

The MVP should avoid unnecessary microservices, distributed systems, event buses, Kubernetes, or other infrastructure that does not provide meaningful value at the initial scale.

---

# 3. Architectural Approach

The application will use a **modular monolith backend** with a separate frontend.

```text
┌─────────────────────────────────────────────────────────┐
│                       User Browser                      │
│                                                         │
│              Next.js Web Application                    │
└──────────────────────────┬──────────────────────────────┘
                           │
                           │ HTTPS / JSON
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    FastAPI Backend                      │
│                                                         │
│  Auth │ Events │ Tasks │ Vendors │ Guests │ Ideas      │
│       │ Billing │ Files │ Notifications │ Settings     │
└──────────────┬──────────────────────┬───────────────────┘
               │                      │
               │                      │
               ▼                      ▼
       ┌───────────────┐      ┌─────────────────┐
       │  PostgreSQL   │      │ Object Storage  │
       │   Supabase    │      │     Files       │
       └───────────────┘      └─────────────────┘

               │
               │ External integrations
               ▼

       ┌───────────────┬───────────────┬───────────────┐
       │               │               │
       ▼               ▼               ▼
    Fast2SMS       Razorpay/         Email/
      OTP          Cashfree          future services
```

---

# 4. System Components

The initial system consists of:

| Component       | Responsibility                      | Technology               |
| --------------- | ----------------------------------- | ------------------------ |
| Web Application | User interface                      | Next.js                  |
| API             | Business logic and API              | FastAPI                  |
| Database        | Persistent data                     | PostgreSQL               |
| Authentication  | OTP verification/session management | FastAPI + OTP provider   |
| SMS             | OTP delivery                        | Fast2SMS initially       |
| Payments        | Subscription billing                | Razorpay/Cashfree        |
| Object Storage  | Uploaded images/files               | S3-compatible storage    |
| CDN             | Static/frontend delivery            | Vercel/CDN               |
| Monitoring      | Errors/logs/health                  | TBD                      |
| Source Control  | Code repository                     | GitHub                   |
| CI/CD           | Automated build/test/deploy         | GitHub Actions + hosting |

---

# 5. Repository Architecture

The application will use a monorepo.

```text
event-planner/
│
├── apps/
│   │
│   ├── web/
│   │   └── Next.js application
│   │
│   └── api/
│       └── FastAPI application
│
├── docs/
│   ├── 01-product/
│   ├── 02-requirements/
│   ├── 03-user-experience/
│   ├── 04-architecture/
│   ├── 05-database/
│   ├── 06-api/
│   ├── 07-business/
│   ├── 08-security/
│   ├── 09-infrastructure/
│   └── 10-development/
│
├── infrastructure/
│   ├── docker/
│   └── deployment/
│
├── .github/
│   └── workflows/
│
├── .env.example
├── docker-compose.yml
├── .gitignore
└── README.md
```

---

# 6. Deployment Boundaries

The frontend and backend are independently deployable.

```text
GitHub Monorepo
       │
       ├──────────────► Vercel
       │                 │
       │                 ▼
       │              apps/web
       │
       └──────────────► Render
                         │
                         ▼
                      apps/api
```

This allows:

- Frontend deployments without backend deployments
- Backend deployments without frontend deployments
- Independent scaling
- Separate environment variables
- Clear ownership of runtime responsibilities

---

# 7. Frontend Architecture

The frontend is built using:

```text
Next.js
TypeScript
Tailwind CSS
```

The frontend is responsible for:

- Rendering the user interface
- Navigation
- Form handling
- Client-side validation
- API communication
- Loading states
- Error states
- Empty states
- Responsive layouts
- Authentication UI
- Subscription UI
- User interactions

The frontend must **not** be responsible for authoritative business decisions.

---

# 8. Frontend Application Structure

Recommended structure:

```text
apps/web/
│
├── app/
│   ├── (public)/
│   ├── (auth)/
│   ├── dashboard/
│   ├── tasks/
│   ├── vendors/
│   ├── guests/
│   ├── ideas/
│   ├── subscription/
│   └── settings/
│
├── components/
│   ├── ui/
│   ├── layout/
│   ├── dashboard/
│   ├── tasks/
│   ├── vendors/
│   ├── guests/
│   ├── ideas/
│   └── subscription/
│
├── lib/
│   ├── api/
│   ├── auth/
│   ├── validation/
│   └── utils/
│
├── hooks/
├── types/
├── constants/
├── styles/
└── public/
```

Exact folder structure may evolve during implementation, but domain boundaries should remain clear.

---

# 9. Frontend Rendering Strategy

Next.js should use the appropriate rendering strategy per page.

## Public pages

Examples:

```text
Landing
Pricing
Terms
Privacy
```

These can use:

- Server rendering
- Static generation
- Cached content

depending on requirements.

## Authenticated application

Examples:

```text
Dashboard
Tasks
Vendors
Guests
Ideas
Settings
```

These depend on authenticated user data and should retrieve protected information through the backend API.

---

# 10. Frontend State Management

The MVP should avoid introducing a large global state-management framework unless required.

State should be separated into:

```text
Server state
Client UI state
Form state
Authentication state
```

Examples:

### Server state

```text
events
tasks
vendors
guests
ideas
subscription
```

### UI state

```text
modal open/closed
sidebar state
selected filters
temporary form state
```

The frontend should not maintain a second authoritative copy of backend data.

---

# 11. API Client Architecture

The frontend should communicate with the backend through a centralized API layer.

Example:

```text
components
     ↓
hooks / feature logic
     ↓
API client
     ↓
FastAPI
```

Avoid making raw `fetch()` calls throughout individual UI components.

Recommended:

```text
lib/api/
├── client.ts
├── auth.ts
├── events.ts
├── tasks.ts
├── vendors.ts
├── guests.ts
├── ideas.ts
└── subscriptions.ts
```

This centralizes:

- Base URL
- Authentication
- Headers
- Error handling
- Serialization
- Response handling

---

# 12. API Contract

The backend exposes versioned APIs.

Base path:

```text
/api/v1/
```

Example:

```text
POST /api/v1/auth/request-otp
POST /api/v1/auth/verify-otp

GET  /api/v1/events
POST /api/v1/events

GET  /api/v1/events/{event_id}
PATCH /api/v1/events/{event_id}
DELETE /api/v1/events/{event_id}
```

The API specification will be documented separately in:

```text
docs/06-api/api-specification.md
```

FastAPI's OpenAPI specification should be treated as the technical API contract.

---

# 13. Backend Architecture

The FastAPI application will use a modular architecture.

Recommended structure:

```text
apps/api/
│
├── app/
│   │
│   ├── main.py
│   │
│   ├── api/
│   │   └── v1/
│   │
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   ├── database.py
│   │   └── logging.py
│   │
│   ├── modules/
│   │   ├── auth/
│   │   ├── events/
│   │   ├── tasks/
│   │   ├── vendors/
│   │   ├── guests/
│   │   ├── ideas/
│   │   ├── subscriptions/
│   │   ├── payments/
│   │   ├── files/
│   │   └── notifications/
│   │
│   ├── models/
│   ├── schemas/
│   ├── repositories/
│   ├── services/
│   └── dependencies/
│
├── migrations/
├── tests/
├── requirements.txt
└── Dockerfile
```

---

# 14. Backend Layering

Backend responsibilities should be separated.

```text
HTTP Request
     ↓
Router
     ↓
Schema Validation
     ↓
Authentication / Authorization
     ↓
Service Layer
     ↓
Repository / Data Access
     ↓
PostgreSQL
```

## Router

Responsible for:

- HTTP methods
- Path parameters
- Request/response handling
- Dependency injection

## Schema

Responsible for:

- Request validation
- Response serialization
- API contracts

## Service

Responsible for:

- Business rules
- Transactions
- Domain operations
- External service coordination

## Repository

Responsible for:

- Database queries
- Persistence operations

This separation prevents business logic from being embedded directly in API routes.

---

# 15. Domain Modules

The backend should be organized around business domains.

```text
auth
events
tasks
vendors
guests
ideas
subscriptions
payments
files
notifications
```

Each module can contain:

```text
router
schemas
models
service
repository
tests
```

Example:

```text
modules/tasks/
├── router.py
├── schemas.py
├── models.py
├── service.py
├── repository.py
└── tests/
```

---

# 16. Authentication Architecture

Authentication uses mobile number + OTP.

```text
User
  ↓
Enter mobile number
  ↓
POST /auth/request-otp
  ↓
FastAPI
  ↓
Fast2SMS
  ↓
User receives OTP
  ↓
POST /auth/verify-otp
  ↓
FastAPI verifies OTP
  ↓
Create/login user
  ↓
Create authenticated session
```

---

# 17. OTP Provider Abstraction

The backend must not tightly couple authentication logic to Fast2SMS.

Use an abstraction:

```text
OTPService
    │
    ├── Fast2SMSProvider
    │
    └── FutureProvider
```

Conceptually:

```python
class OTPProvider:
    def send_otp(...):
        ...

    def verify_delivery(...):
        ...
```

This allows the SMS provider to be replaced without rewriting authentication logic.

---

# 18. Authentication Security

OTP authentication must implement:

- OTP expiration
- Maximum verification attempts
- Resend cooldown
- Rate limiting
- Mobile number normalization
- Brute-force protection
- Secure session creation
- Session expiration
- Logout/revocation

OTP values must not be stored in plaintext.

---

# 19. Session Architecture

The application should use secure authenticated sessions.

Recommended approach:

```text
Browser
   ↓
Secure HTTP-only cookie
   ↓
FastAPI
   ↓
Session validation
```

Authentication tokens must not be unnecessarily exposed to client-side JavaScript.

Session cookies should use appropriate:

```text
HttpOnly
Secure
SameSite
```

settings according to deployment architecture.

---

# 20. Authorization Architecture

Authentication answers:

```text
Who is the user?
```

Authorization answers:

```text
What can this user access?
```

Every protected event resource must verify authorization.

Example:

```text
Request
  ↓
Authenticated user
  ↓
Event membership check
  ↓
Resource ownership
  ↓
Allow / deny
```

---

# 21. Event Ownership

The primary ownership relationship is:

```text
User
  ↓
Event
  ↓
Event Resources
```

Event resources include:

```text
Tasks
Vendors
Guests
Ideas
Files
```

The API must never trust a client-provided event ID without verifying that the authenticated user has access to it.

---

# 22. Database Architecture

PostgreSQL is the primary persistent data store.

```text
FastAPI
   │
   ▼
Database access layer
   │
   ▼
PostgreSQL
```

Supabase PostgreSQL is the planned initial managed database.

The application should communicate with PostgreSQL through the backend.

The frontend should not directly access the database.

---

# 23. ORM and Database Access

The backend should use a Python PostgreSQL ORM/data-access layer.

The selected ORM should support:

- Async or efficient database access
- Transactions
- Relationships
- Parameterized queries
- Migration integration
- PostgreSQL features

The exact ORM choice can be finalized during project foundation.

Alembic will manage migrations.

---

# 24. Database Connection Management

FastAPI must use controlled database connection pooling.

The application must not create a new database connection for every request without pooling.

Production configuration should include:

```text
connection pool size
maximum overflow
connection timeout
idle timeout
```

Exact values should be tuned based on deployment capacity.

---

# 25. File Storage Architecture

Images and uploaded files should not be stored inside PostgreSQL.

Architecture:

```text
Browser
   ↓
FastAPI
   ↓
Authorization + validation
   ↓
Object Storage
   ↓
File metadata → PostgreSQL
```

The database stores metadata such as:

```text
storage_key
filename
mime_type
file_size
event_id
idea_id
created_at
```

---

# 26. Object Storage

The initial implementation should use S3-compatible object storage.

Possible providers include:

```text
Supabase Storage
Cloudflare R2
AWS S3
```

The exact provider can be finalized during infrastructure setup.

The application should isolate storage access behind a storage service:

```text
StorageService
    │
    ├── upload()
    ├── delete()
    ├── generate_url()
    └── validate()
```

This prevents business logic from depending directly on a specific storage vendor.

---

# 27. File Upload Flow

```text
User selects image
       ↓
Frontend validation
       ↓
API request
       ↓
Authentication
       ↓
Event authorization
       ↓
File type/size validation
       ↓
Object storage
       ↓
Metadata saved to PostgreSQL
       ↓
Response
```

File names supplied by users should not be trusted as storage paths.

Generated storage keys should be used.

---

# 28. Subscription Architecture

Subscription status is controlled by the backend.

```text
User
  ↓
3-day trial
  ↓
Subscription checkout
  ↓
Payment Provider
  ↓
Webhook
  ↓
FastAPI
  ↓
PostgreSQL
  ↓
Subscription status
```

The frontend only displays the current subscription state.

---

# 29. Trial Architecture

Trial state should be calculated using server timestamps.

Example:

```text
trial_started_at
trial_ends_at
```

The frontend must not calculate whether a trial has expired and use that result as authorization.

Instead:

```text
Frontend
   ↓
API request
   ↓
Backend checks subscription/trial
   ↓
Allow / restrict
```

---

# 30. Payment Architecture

The initial payment provider will be selected between:

```text
Razorpay
Cashfree
```

The payment integration should use an abstraction.

```text
PaymentService
      │
      ├── RazorpayProvider
      │
      └── CashfreeProvider
```

This prevents the core subscription system from becoming tightly coupled to one provider.

---

# 31. Payment Webhooks

Payment status must be updated from verified provider webhooks.

```text
Payment Provider
       │
       │ HTTPS webhook
       ▼
FastAPI
       │
       ├── Verify signature
       ├── Validate payload
       ├── Check idempotency
       └── Update database
```

The system must not rely exclusively on the browser redirect after payment.

---

# 32. Webhook Idempotency

Payment webhooks can be delivered more than once.

The backend must safely handle duplicate events.

Example:

```text
provider_payment_id UNIQUE
```

or equivalent provider event identifiers should prevent duplicate processing.

---

# 33. Subscription Access Control

Every protected application request should be evaluated according to:

```text
Authentication
        ↓
Authorization
        ↓
Subscription access
```

However, subscription checks should be applied according to product policy rather than blindly blocking every endpoint.

For example:

```text
Account settings
Payment page
Subscription status
Logout
```

may remain accessible even after subscription expiration.

---

# 34. API Error Architecture

The backend should return consistent error responses.

Example:

```json
{
  "error": {
    "code": "EVENT_NOT_FOUND",
    "message": "The requested event could not be found."
  }
}
```

The frontend should use the machine-readable error code for behavior and the message for user-facing feedback where appropriate.

---

# 35. HTTP Status Codes

The API should use standard HTTP status codes.

| Status | Usage                                    |
| ------ | ---------------------------------------- |
| `200`  | Successful request                       |
| `201`  | Resource created                         |
| `204`  | Successful request with no response body |
| `400`  | Invalid request                          |
| `401`  | Unauthenticated                          |
| `403`  | Unauthorized                             |
| `404`  | Resource not found                       |
| `409`  | Conflict                                 |
| `422`  | Validation failure                       |
| `429`  | Rate limited                             |
| `500`  | Internal server error                    |
| `503`  | Temporary service unavailable            |

---

# 36. Request Validation

Validation should occur at the API boundary.

Examples:

```text
Mobile number
OTP
Event name
Event date
Guest count
Money values
File type
File size
```

Pydantic models should be used for FastAPI request/response validation.

---

# 37. Business Logic

Business rules must live in backend services rather than UI components.

Examples:

```text
Trial expiration
Subscription activation
Payment reconciliation
Event ownership
Task completion
Vendor payment totals
Guest attendance totals
```

The frontend may provide convenience validation, but it must not be the only validation layer.

---

# 38. Transaction Boundaries

Operations requiring multiple related database writes should use transactions.

Example:

```text
Create subscription
      ↓
Create payment
      ↓
Commit
```

If a required operation fails:

```text
Rollback
```

This prevents partially persisted business state.

---

# 39. External Service Boundaries

External services must be isolated behind internal service interfaces.

```text
Application
    │
    ├── OTPService
    │      └── Fast2SMS
    │
    ├── PaymentService
    │      └── Razorpay/Cashfree
    │
    └── StorageService
           └── Object Storage
```

The domain logic should not directly call provider SDKs throughout the codebase.

---

# 40. Notification Architecture

MVP notifications may initially be generated by backend application logic.

Potential notification events:

```text
Task due
Task overdue
Trial expiring
Payment failed
Subscription expired
```

Architecture:

```text
Business Event
      ↓
Notification Service
      ↓
notifications table
      ↓
Frontend
```

External push notifications or WhatsApp/SMS notifications can be introduced later.

---

# 41. Search and Filtering

The MVP does not require a dedicated search engine.

PostgreSQL queries are sufficient for:

```text
Task filtering
Vendor filtering
Guest filtering
Idea filtering
```

Examples:

```text
tasks.status
tasks.priority
tasks.category

vendors.category

guests.rsvp_status
guests.group_name

ideas.category
```

A dedicated search engine should only be introduced when PostgreSQL becomes insufficient.

---

# 42. Dashboard Architecture

The dashboard should aggregate information from existing domain entities.

```text
Dashboard API
      │
      ├── Event
      ├── Tasks
      ├── Vendors
      ├── Guests
      ├── Ideas
      └── Subscription
```

A dedicated dashboard database table is not required for MVP.

Example dashboard response:

```json
{
  "event": {},
  "progress": {},
  "tasks": {},
  "vendors": {},
  "guests": {},
  "subscription": {}
}
```

Exact response schema will be defined in the API specification.

---

# 43. Caching

Caching should be introduced only where it provides measurable value.

Initial MVP:

```text
No mandatory distributed cache
```

PostgreSQL and application-level optimization should be sufficient for the initial workload.

Potential future use:

```text
Redis
```

for:

- Rate limiting
- Temporary OTP state
- Caching
- Background jobs
- Distributed locks

Redis is not required for the initial architecture unless implementation requirements justify it.

---

# 44. Background Jobs

The MVP should avoid introducing a distributed job system unless necessary.

Potential future background jobs include:

```text
Reminder notifications
Email delivery
Image processing
Subscription reconciliation
Analytics processing
```

A background job architecture can be introduced later using a queue and worker system.

---

# 45. Observability

The application should provide:

## Application logs

Backend logs should capture:

- Request ID
- HTTP method
- Route
- Status code
- Duration
- Error information

Sensitive data must not be logged.

Do not log:

```text
OTP
passwords
session tokens
payment secrets
full authentication credentials
```

---

# 46. Health Checks

The API should expose a health endpoint.

Example:

```text
GET /health
```

Possible response:

```json
{
  "status": "ok"
}
```

A deeper readiness check may verify:

```text
API
Database
Critical dependencies
```

separately from the basic liveness check.

---

# 47. Error Monitoring

Production errors should be captured by an error monitoring platform.

Possible implementation:

```text
Sentry
```

or equivalent.

Monitoring should capture:

- Exceptions
- Stack traces
- Request context
- Environment
- Release/version

Sensitive information must be excluded.

---

# 48. Security Architecture

The security boundary is:

```text
Internet
   ↓
HTTPS
   ↓
Next.js
   ↓
HTTPS
   ↓
FastAPI
   ↓
Authentication
   ↓
Authorization
   ↓
Business logic
   ↓
PostgreSQL / Storage
```

Security requirements include:

- HTTPS
- Secure cookies
- Authentication
- Authorization
- Input validation
- Rate limiting
- CORS restrictions
- Secure secrets
- Database access restrictions
- File validation
- Webhook verification
- Audit logging

---

# 49. CORS

The API must explicitly allow only approved frontend origins.

Development:

```text
http://localhost:3000
```

Production:

```text
https://<production-domain>
```

Wildcard production CORS should not be used.

---

# 50. Secrets Management

Secrets must never be committed to Git.

Examples:

```text
DATABASE_URL
FAST2SMS_API_KEY
PAYMENT_API_KEY
PAYMENT_SECRET
STORAGE_ACCESS_KEY
STORAGE_SECRET_KEY
SESSION_SECRET
```

Development uses:

```text
.env.local
```

Production secrets should be stored in the hosting provider's secure environment-variable system.

---

# 51. Environment Architecture

Three environments are recommended:

```text
Development
     ↓
Staging
     ↓
Production
```

## Development

Used by developers for local work.

```text
localhost
local database / development database
test integrations
```

## Staging

Used for:

- Integration testing
- QA
- Payment testing
- Release verification

## Production

Used by real customers.

Production data must never be used casually during development.

---

# 52. Environment Variables

Frontend:

```text
NEXT_PUBLIC_API_URL
```

Backend:

```text
DATABASE_URL
SESSION_SECRET
FAST2SMS_API_KEY

PAYMENT_PROVIDER
PAYMENT_KEY_ID
PAYMENT_SECRET

STORAGE_ENDPOINT
STORAGE_BUCKET
STORAGE_ACCESS_KEY
STORAGE_SECRET_KEY

CORS_ORIGINS
```

Only variables explicitly intended for browser exposure may use the `NEXT_PUBLIC_` prefix.

Private secrets must never be exposed to the frontend.

---

# 53. Local Development Architecture

Recommended local setup:

```text
Developer Machine
│
├── Next.js
│     localhost:3000
│
├── FastAPI
│     localhost:8000
│
└── PostgreSQL
      localhost:5432
```

Optional Docker Compose can provide local infrastructure.

Example:

```text
docker-compose.yml
```

can run PostgreSQL and other development dependencies.

---

# 54. Production Architecture

Initial production deployment:

```text
                    Internet
                       │
                       ▼
              ┌─────────────────┐
              │     Vercel      │
              │    Next.js      │
              └────────┬────────┘
                       │
                     HTTPS
                       │
                       ▼
              ┌─────────────────┐
              │     Render      │
              │    FastAPI      │
              └───────┬─────────┘
                      │
             ┌────────┴─────────┐
             │                  │
             ▼                  ▼
      ┌──────────────┐   ┌───────────────┐
      │  Supabase    │   │ Object        │
      │ PostgreSQL   │   │ Storage       │
      └──────────────┘   └───────────────┘
             │
             │
      ┌──────┴───────────┐
      │                  │
      ▼                  ▼
   Fast2SMS       Razorpay/Cashfree
```

---

# 55. Deployment Strategy

Frontend:

```text
GitHub
   ↓
Vercel
   ↓
Build
   ↓
Deploy
```

Backend:

```text
GitHub
   ↓
Render
   ↓
Build
   ↓
Run migrations
   ↓
Deploy
```

Database:

```text
Supabase PostgreSQL
```

The production deployment process must ensure database migrations are applied safely.

---

# 56. CI/CD Architecture

GitHub Actions should automate:

```text
Pull Request
     ↓
Lint
     ↓
Type Check
     ↓
Unit Tests
     ↓
Build
     ↓
Integration Tests
```

Production deployment should happen only after required checks pass.

---

# 57. Branch Strategy

A simple Git workflow is recommended.

```text
main
 │
 ├── feature/*
 ├── fix/*
 └── chore/*
```

Production should deploy from the protected `main` branch.

Pull requests should be reviewed before merging.

---

# 58. API and Frontend Version Compatibility

The frontend and backend are independently deployable.

Therefore, API changes must consider compatibility.

Example:

```text
Frontend v1
     ↓
API v1
```

Breaking API changes should require:

```text
API v2
```

rather than silently changing the existing contract.

---

# 59. API Versioning

Initial API:

```text
/api/v1/
```

Future:

```text
/api/v2/
```

Versioning should be used for breaking contract changes.

Minor backwards-compatible changes do not necessarily require a new API version.

---

# 60. Scalability Strategy

The architecture should scale incrementally.

### Stage 1 — MVP

```text
Vercel
+
Render
+
Supabase
+
Object Storage
```

### Stage 2 — Growth

Potential additions:

```text
Redis
Background workers
CDN optimization
Database read replicas
Advanced monitoring
```

### Stage 3 — Large scale

Potential evolution:

```text
Dedicated services
Queue infrastructure
Dedicated databases
Advanced observability
Regional infrastructure
```

Microservices should not be introduced simply because the product grows. They should be introduced only when domain or scaling requirements justify them.

---

# 61. Failure Handling

The system must assume external services can fail.

Examples:

```text
Fast2SMS unavailable
Payment provider unavailable
Database temporarily unavailable
Object storage unavailable
Network timeout
```

The application should:

- Return clear errors
- Avoid corrupting database state
- Retry only safe operations
- Use idempotency for external operations
- Log failures
- Avoid exposing internal errors to users

---

# 62. External Service Failure Example

If OTP delivery fails:

```text
User
 ↓
FastAPI
 ↓
Fast2SMS
 ↓
Failure
```

The API should return a controlled response such as:

```text
Unable to send OTP. Please try again.
```

It must not expose:

```text
API credentials
provider response secrets
internal stack traces
```

---

# 63. Payment Failure Example

Payment flow:

```text
User
 ↓
Checkout
 ↓
Payment Provider
 ↓
Payment Failure
 ↓
Webhook / API confirmation
 ↓
Database
```

The subscription must not be marked `ACTIVE` merely because the frontend reached a success page.

The backend must verify payment status through trusted provider mechanisms.

---

# 64. Data Flow: Creating an Event

```text
User
  ↓
Create Event Form
  ↓
Next.js
  ↓
POST /api/v1/events
  ↓
FastAPI
  ↓
Authentication
  ↓
Validation
  ↓
Event Service
  ↓
PostgreSQL
  ↓
Event Created
  ↓
API Response
  ↓
Next.js
  ↓
Dashboard
```

---

# 65. Data Flow: Adding a Task

```text
Task Form
   ↓
Next.js
   ↓
POST /api/v1/events/{event_id}/tasks
   ↓
FastAPI
   ↓
Authenticate
   ↓
Authorize Event
   ↓
Validate Task
   ↓
Task Service
   ↓
PostgreSQL
   ↓
Response
   ↓
Task UI updated
```

---

# 66. Data Flow: Adding an Idea Image

```text
User selects image
        ↓
Next.js
        ↓
FastAPI
        ↓
Authentication
        ↓
Event authorization
        ↓
File validation
        ↓
Object Storage
        ↓
Save metadata
        ↓
PostgreSQL
        ↓
Return file reference
```

---

# 67. Data Flow: Subscription

```text
User
  ↓
Subscription Page
  ↓
Checkout
  ↓
Payment Provider
  ↓
Payment
  ↓
Webhook
  ↓
FastAPI
  ↓
Verify webhook
  ↓
Update payment
  ↓
Update subscription
  ↓
PostgreSQL
  ↓
API
  ↓
Frontend
```

---

# 68. Architectural Boundaries

The following boundaries must be maintained.

## Frontend must not:

- Access PostgreSQL directly
- Contain payment secrets
- Contain SMS provider secrets
- Determine subscription authorization
- Determine event ownership
- Trust client-side permission checks

## Backend must:

- Authenticate requests
- Authorize resources
- Validate business rules
- Manage database access
- Manage payment state
- Manage external service integrations

## Database must:

- Enforce referential integrity
- Enforce important constraints
- Persist authoritative application state

---

# 69. Anti-Patterns to Avoid

The project must avoid:

### Direct frontend-to-database access

```text
Next.js → PostgreSQL
```

### Business logic inside UI components

```text
React Component → subscription calculation
```

### Provider-specific logic everywhere

```text
TaskService → Fast2SMS
VendorService → Razorpay
```

Instead use dedicated service abstractions.

### Microservices too early

```text
auth-service
task-service
vendor-service
guest-service
idea-service
```

This creates unnecessary operational complexity for the MVP.

### Unversioned breaking APIs

Avoid changing existing contracts without versioning.

### Storing files in PostgreSQL

Use object storage.

### Trusting client-side authorization

Authorization must always be performed server-side.

---

# 70. Architectural Quality Requirements

The implementation should satisfy:

- Clear module boundaries
- Strong typing
- Explicit API contracts
- Automated testing
- Database migrations
- Centralized error handling
- Structured logging
- Secure secret management
- Environment separation
- Reliable payment webhook handling
- Secure file access
- Server-side authorization
- Mobile-responsive frontend
- Production-ready deployment

---

# 71. Architecture Definition of Done

The architecture is considered implemented when:

- [ ] Monorepo is created
- [ ] Next.js application is configured
- [ ] FastAPI application is configured
- [ ] PostgreSQL connection is implemented
- [ ] Alembic migrations are configured
- [ ] API versioning is implemented
- [ ] Authentication architecture is implemented
- [ ] Authorization architecture is implemented
- [ ] Event ownership checks are implemented
- [ ] OTP provider abstraction is implemented
- [ ] Payment provider abstraction is implemented
- [ ] Storage abstraction is implemented
- [ ] File upload flow is implemented
- [ ] Subscription enforcement is implemented
- [ ] Error handling is standardized
- [ ] Logging is implemented
- [ ] Health checks are implemented
- [ ] CORS is configured
- [ ] Environment variables are configured
- [ ] Development environment works
- [ ] Staging environment works
- [ ] Production deployment works
- [ ] CI/CD pipeline is configured
- [ ] Database backups are configured
- [ ] Monitoring is configured
- [ ] Security requirements are validated

---

# 72. Related Documentation

This document depends on:

```text
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/03-user-experience/user-flows.md
docs/03-user-experience/screen-inventory.md
```

This document is directly related to:

```text
docs/05-database/database-design.md
docs/06-api/api-specification.md
docs/08-security/security.md
docs/09-infrastructure/infrastructure.md
docs/10-development/development-guide.md
```

---

# 73. Final Architecture

The target MVP architecture is intentionally straightforward:

```text
                         ┌─────────────────────┐
                         │       USERS         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      Next.js       │
                         │      Frontend      │
                         │       Vercel       │
                         └──────────┬──────────┘
                                    │
                               HTTPS / JSON
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      FastAPI       │
                         │       Backend      │
                         │       Render       │
                         └───────┬─┬─┬────────┘
                                 │ │ │
                  ┌──────────────┘ │ └──────────────┐
                  │                │                │
                  ▼                ▼                ▼
           ┌────────────┐   ┌────────────┐   ┌──────────────┐
           │ PostgreSQL │   │   Object   │   │   External   │
           │  Supabase  │   │  Storage   │   │   Services   │
           └────────────┘   └────────────┘   └──────┬───────┘
                                                    │
                                           ┌────────┼────────┐
                                           │        │        │
                                           ▼        ▼        ▼
                                       Fast2SMS  Payment   Future
                                                 Provider  Services
```

The architectural principle is:

> **Keep the MVP modular and production-ready without introducing infrastructure complexity before the product actually needs it.**

The frontend owns presentation, the FastAPI backend owns business logic and authorization, PostgreSQL owns persistent state, object storage owns files, and external providers are isolated behind service abstractions.
