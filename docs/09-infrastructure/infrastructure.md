# Infrastructure and Deployment

**Document:** `docs/09-infrastructure/infrastructure.md`  
**Status:** Draft  
**Version:** 1.0  
**Application:** Event Planner  
**Frontend:** Next.js + TypeScript  
**Backend:** FastAPI + Python  
**Database:** PostgreSQL  
**Initial Deployment:** Vercel + Render + Supabase  
**Source Control:** GitHub  
**Initial Market:** India

---

# 1. Purpose

This document defines the infrastructure, deployment, environment, CI/CD, networking, storage, monitoring, backup and operational requirements for the Event Planner application.

It translates the system architecture into a practical deployment model.

This document covers:

- Hosting
- Environments
- Domains
- DNS
- Frontend deployment
- Backend deployment
- PostgreSQL
- Object storage
- SMS integration
- Payment integration
- Environment variables
- Docker
- CI/CD
- Database migrations
- Backups
- Monitoring
- Logging
- Health checks
- Scaling
- Disaster recovery
- Production release process

---

# 2. Infrastructure Principles

The infrastructure must prioritize:

1. Simplicity
2. Reliability
3. Security
4. Low operational overhead
5. Independent deployment
6. Environment isolation
7. Automated deployment
8. Backup and recovery
9. Observability
10. Incremental scalability

The MVP should not introduce infrastructure that is not justified by the current product scale.

---

# 3. Initial Infrastructure Stack

The planned MVP stack is:

| Component        | Service                          | Purpose               |
| ---------------- | -------------------------------- | --------------------- |
| Source control   | GitHub                           | Code repository       |
| Frontend         | Vercel                           | Next.js hosting       |
| Backend          | Render                           | FastAPI hosting       |
| Database         | Supabase PostgreSQL              | PostgreSQL            |
| Object storage   | Supabase Storage / S3-compatible | Images/files          |
| OTP              | Fast2SMS                         | SMS OTP               |
| Payments         | Razorpay / Cashfree              | Subscription payments |
| CI/CD            | GitHub Actions                   | Testing/automation    |
| Error monitoring | Sentry or equivalent             | Application errors    |
| DNS              | Domain provider / Cloudflare     | DNS and domain        |
| SSL              | Vercel/Render managed TLS        | HTTPS                 |

The final provider selection for payment and object storage can be confirmed before implementation.

---

# 4. High-Level Infrastructure

```text id="f2z5q1"
                         ┌──────────────────┐
                         │      Users       │
                         └────────┬─────────┘
                                  │
                                HTTPS
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      Vercel      │
                         │     Next.js      │
                         └────────┬─────────┘
                                  │
                              HTTPS / API
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      Render      │
                         │     FastAPI      │
                         └─────┬────┬───────┘
                               │    │
                    ┌──────────┘    └──────────────┐
                    │                              │
                    ▼                              ▼
             ┌──────────────┐              ┌──────────────┐
             │   Supabase   │              │    Object    │
             │ PostgreSQL   │              │   Storage    │
             └──────────────┘              └──────────────┘
                    │
                    │
             ┌──────┴───────────┐
             │                  │
             ▼                  ▼
         Fast2SMS        Razorpay/Cashfree
```

---

# 5. Environment Architecture

Three environments are required:

```text id="2k6h1q"
Development
      ↓
Staging
      ↓
Production
```

Each environment must have separate configuration.

---

# 6. Development Environment

Purpose:

- Local development
- Feature development
- Unit testing
- Integration testing

Typical setup:

```text id="9k4n5s"
Next.js
localhost:3000

FastAPI
localhost:8000

PostgreSQL
localhost:5432
```

Development services should never use production credentials.

---

# 7. Staging Environment

Purpose:

- QA
- Integration testing
- Release validation
- Payment sandbox testing
- Production-like testing

Example:

```text id="q6v5x8"
Frontend:
https://staging.<domain>

API:
https://api-staging.<domain>
```

Staging should use:

- Separate database
- Separate storage bucket
- Test SMS/payment credentials where available
- Separate secrets

---

# 8. Production Environment

Production serves real users.

Example:

```text id="m6c8s4"
Frontend:
https://app.<domain>

API:
https://api.<domain>
```

Production must use:

- Production database
- Production storage
- Production payment credentials
- Production SMS credentials
- Production secrets
- Production monitoring
- Automated backups

---

# 9. Environment Isolation

The following must never be shared casually between environments:

```text id="3v5k3n"
Database
Storage buckets
Payment credentials
SMS credentials
Session secrets
API secrets
```

Production data must not be used in development.

---

# 10. Repository Structure

The infrastructure assumes the following repository:

```text id="k4m3n7"
event-planner/
│
├── apps/
│   ├── web/
│   └── api/
│
├── docs/
│
├── infrastructure/
│   ├── docker/
│   └── deployment/
│
├── .github/
│   └── workflows/
│
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

---

# 11. GitHub

GitHub is the source-control system.

Responsibilities:

- Repository hosting
- Pull requests
- Code review
- Branch protection
- CI/CD
- Release history
- Security scanning

The `main` branch should be protected.

---

# 12. Branch Strategy

Recommended:

```text id="6o5y3z"
main
 │
 ├── feature/*
 ├── fix/*
 ├── refactor/*
 └── chore/*
```

Feature branches should be merged through pull requests.

Direct pushes to `main` should be restricted.

---

# 13. Pull Request Requirements

Before merging:

- Code review completed
- Tests pass
- Lint passes
- Type checks pass
- Build passes
- Security checks pass where applicable

Database migrations must be reviewed separately.

---

# 14. Frontend Deployment

The Next.js application will be deployed to Vercel.

Deployment root:

```text id="0u7v9d"
apps/web
```

Vercel should be configured with the monorepo root directory appropriately.

---

# 15. Frontend Build

Typical build:

```text id="h6v4p8"
npm install
npm run build
```

The exact package manager will be finalized during project initialization.

The build must fail if:

- Type checking fails
- Production compilation fails
- Required environment configuration is missing

---

# 16. Frontend Environment Variables

Public variables:

```text id="j4d7w8"
NEXT_PUBLIC_API_URL
```

Only values that are safe for browser exposure may use `NEXT_PUBLIC_`.

Private secrets must never be included in frontend environment variables.

---

# 17. Backend Deployment

The FastAPI application will be deployed to Render.

Deployment root:

```text id="h2s9d4"
apps/api
```

Render should run the FastAPI application using a production ASGI server.

Example:

```text id="9v3j5f"
uvicorn app.main:app
```

The exact production worker configuration will be finalized during performance testing.

---

# 18. Backend Build

The backend deployment should:

```text id="9l6n5k"
Install dependencies
       ↓
Run checks
       ↓
Build application
       ↓
Run migrations
       ↓
Start FastAPI
```

Migrations must be handled carefully to avoid application/database version mismatches.

---

# 19. Production ASGI Server

FastAPI should run through a production-grade ASGI server.

Possible configuration:

```text id="l5v0mz"
Gunicorn
+
Uvicorn workers
```

or an equivalent supported deployment configuration.

Worker count should be determined based on available CPU/memory rather than arbitrarily increasing workers.

---

# 20. Backend Environment Variables

Production backend configuration may include:

```text id="9h8n3x"
APP_ENV
DATABASE_URL

SESSION_SECRET

CORS_ORIGINS

FAST2SMS_API_KEY

PAYMENT_PROVIDER
PAYMENT_KEY_ID
PAYMENT_SECRET
PAYMENT_WEBHOOK_SECRET

STORAGE_ENDPOINT
STORAGE_BUCKET
STORAGE_ACCESS_KEY
STORAGE_SECRET_KEY

SENTRY_DSN
```

Only required variables should be configured.

---

# 21. Secret Management

Secrets must be stored in the hosting platform's encrypted environment-variable system.

Never commit:

```text id="z6z1f4"
.env
.env.local
.env.production
```

containing real credentials.

The repository should contain:

```text id="c5x9t3"
.env.example
```

with placeholders only.

---

# 22. Example `.env.example`

```text id="v4g8s2"
APP_ENV=development

DATABASE_URL=

SESSION_SECRET=

CORS_ORIGINS=http://localhost:3000

FAST2SMS_API_KEY=

PAYMENT_PROVIDER=

PAYMENT_KEY_ID=
PAYMENT_SECRET=
PAYMENT_WEBHOOK_SECRET=

STORAGE_ENDPOINT=
STORAGE_BUCKET=
STORAGE_ACCESS_KEY=
STORAGE_SECRET_KEY=

SENTRY_DSN=
```

No real credentials should ever appear in this file.

---

# 23. Database Infrastructure

PostgreSQL will initially be provided by Supabase.

The database is the authoritative persistence layer.

Architecture:

```text id="s7k4j2"
FastAPI
   │
   │ secure database connection
   ▼
Supabase PostgreSQL
```

---

# 24. Database Environments

Recommended:

```text id="x5w7r9"
Development → Development database
Staging     → Staging database
Production  → Production database
```

At minimum, production must be isolated from development.

---

# 25. Database Connection

The backend should use a connection pool.

Configuration should account for:

- Maximum connections
- Connection timeout
- Idle connections
- Query timeout where appropriate

Connection limits must respect the managed database plan.

---

# 26. Database Migrations

Alembic will manage schema changes.

Migration files should live within:

```text id="f3d2g8"
apps/api/migrations/
```

or the finalized backend migration directory.

---

# 27. Migration Workflow

```text id="q9s5h7"
Modify Model
      ↓
Create Alembic Migration
      ↓
Review Migration
      ↓
Run Locally
      ↓
Run Tests
      ↓
Apply to Staging
      ↓
Validate
      ↓
Apply to Production
```

Production migrations must never be generated blindly during deployment.

Migration files should be committed to Git.

---

# 28. Database Migration Safety

Migrations should avoid unnecessary downtime.

Prefer:

```text id="8c3v5n"
Add nullable column
      ↓
Deploy application
      ↓
Backfill data
      ↓
Add constraint
```

over potentially destructive changes in a single deployment.

Destructive migrations require explicit review.

---

# 29. Database Backups

Production PostgreSQL must have automated backups enabled.

The backup strategy must define:

- Frequency
- Retention
- Recovery procedure
- Restore testing
- Access control

The managed Supabase backup capabilities should be used initially.

---

# 30. Backup Testing

Backups must periodically be restored into a non-production environment.

A backup should not be considered reliable until restoration has been tested.

Recommended process:

```text id="2f8k3n"
Production Backup
      ↓
Restore to isolated environment
      ↓
Verify schema
      ↓
Verify representative records
      ↓
Document result
```

---

# 31. Object Storage

Object storage is used for:

- Event images
- Idea images
- Future uploaded documents

The initial implementation may use Supabase Storage or another S3-compatible provider.

---

# 32. Storage Structure

Recommended logical structure:

```text id="h7x3m8"
events/
  {event_id}/
    event-image/
    ideas/
      {idea_id}/
        {file_id}
```

Generated UUIDs should be used for object names.

---

# 33. Storage Buckets

Separate buckets may be used for different environments:

```text id="7r6c4z"
event-planner-dev
event-planner-staging
event-planner-production
```

Production files must not be stored in development buckets.

---

# 34. Storage Access

Private user files should use private buckets where supported.

Access should be granted using:

```text id="6v8m2q"
Authorized API
        ↓
Short-lived signed URL
        ↓
Object Storage
```

Permanent public URLs should not be used for private event content.

---

# 35. Storage Limits

The backend must enforce:

- Maximum file size
- Allowed MIME types
- Maximum number of attachments where required

The frontend may validate the same rules for user experience.

Backend validation remains authoritative.

---

# 36. Domain Architecture

Recommended production domains:

```text id="k5v7q2"
app.<domain>
api.<domain>
```

Staging:

```text id="j4m8n1"
staging.<domain>
api-staging.<domain>
```

The exact domain will be finalized before launch.

---

# 37. DNS

DNS records should point to the appropriate hosting providers.

Conceptually:

```text id="f9k2m7"
app.<domain>
      ↓
Vercel

api.<domain>
      ↓
Render
```

DNS should be managed through the selected domain/DNS provider.

Cloudflare may be used if additional DNS, security or CDN controls are required.

---

# 38. SSL/TLS

HTTPS must be enabled for:

```text id="y7x3n4"
app.<domain>
api.<domain>
staging.<domain>
api-staging.<domain>
```

Vercel and Render managed certificates can be used initially.

Certificates must be automatically renewed.

---

# 39. API Networking

The frontend communicates with the backend over HTTPS.

```text id="s4g8n0"
Browser
   │
   │ HTTPS
   ▼
Vercel
   │
   │ HTTPS
   ▼
Render FastAPI
```

The API must not expose database ports directly to the browser.

---

# 40. CORS Configuration

Development:

```text id="1c9x4v"
http://localhost:3000
```

Staging:

```text id="0m7s5k"
https://staging.<domain>
```

Production:

```text id="w3q8f1"
https://app.<domain>
```

Only required origins should be allowed.

---

# 41. Health Checks

The backend must expose:

```http id="v6y4r2"
GET /health
```

Basic response:

```json id="2f8j3v"
{
  "status": "ok"
}
```

A readiness endpoint may additionally verify database connectivity.

Example:

```http id="p8j1s7"
GET /health/ready
```

---

# 42. Liveness vs Readiness

## Liveness

Answers:

```text id="7s0m5x"
Is the application process running?
```

## Readiness

Answers:

```text id="h3n9c6"
Can the application serve traffic?
```

Readiness may check:

- Database connection
- Required configuration
- Critical dependencies

---

# 43. Monitoring

Production monitoring should cover:

### Application

- Error rate
- Response time
- Request volume
- HTTP status codes

### Database

- Connection usage
- Query performance
- Storage usage
- Availability

### Infrastructure

- CPU
- Memory
- Instance health

### External services

- OTP failures
- Payment failures
- Storage failures

---

# 44. Error Monitoring

Sentry or an equivalent service should be used for application exceptions.

The monitoring system should capture:

```text id="x3p9v7"
Exception
Stack trace
Environment
Release
Request ID
Relevant safe context
```

Sensitive values must be scrubbed.

---

# 45. Logging

Backend logs should be structured.

Example fields:

```text id="j5k2r8"
timestamp
level
request_id
route
method
status_code
duration
environment
user_id where safe
error_code
```

Never log:

```text id="x8p1d4"
OTP
session tokens
API keys
payment secrets
passwords
```

---

# 46. Log Retention

Log retention should be long enough to investigate incidents while avoiding unnecessary retention.

The exact retention period will be determined based on:

- Provider capabilities
- Cost
- Operational requirements
- Privacy requirements

---

# 47. CI/CD Architecture

GitHub Actions should automate quality checks.

```text id="v6p8s2"
Pull Request
     ↓
Install dependencies
     ↓
Lint
     ↓
Type checking
     ↓
Unit tests
     ↓
Build
     ↓
Security checks
```

Deployment should occur only after required checks pass.

---

# 48. Frontend CI

Frontend pipeline:

```text id="x8m4k1"
Checkout
   ↓
Install dependencies
   ↓
Lint
   ↓
Typecheck
   ↓
Test
   ↓
Build
```

---

# 49. Backend CI

Backend pipeline:

```text id="m3v7c2"
Checkout
   ↓
Install dependencies
   ↓
Lint
   ↓
Type check
   ↓
Unit tests
   ↓
Integration tests
   ↓
Build
```

---

# 50. Database CI

Database tests should verify:

- Migrations run successfully
- Fresh database can be created
- Constraints work
- Seed data works where applicable

Example:

```text id="k2n5r9"
Empty PostgreSQL
      ↓
Alembic upgrade head
      ↓
Run integration tests
```

---

# 51. Deployment Pipeline

Production deployment:

```text id="g5f9m2"
Developer
    ↓
Pull Request
    ↓
CI
    ↓
Code Review
    ↓
Merge to main
    ↓
Production Build
    ↓
Database Migration
    ↓
Deployment
    ↓
Health Check
    ↓
Release Complete
```

---

# 52. Frontend Deployment

Vercel should deploy the frontend from `main`.

Preview deployments may be generated for pull requests.

Recommended:

```text id="w8h3p6"
Pull Request
     ↓
Vercel Preview
```

This allows UI testing before merging.

---

# 53. Backend Deployment

Render should deploy the backend from the protected production branch.

Staging should deploy from an appropriate staging branch or controlled deployment process.

---

# 54. Deployment Ordering

For backward-compatible changes:

```text id="x6q9r3"
1. Database migration
2. Backend deployment
3. Frontend deployment
```

However, migration ordering must depend on compatibility.

For schema changes that require both old and new application versions to work temporarily:

```text id="r5k7v1"
Expand
  ↓
Deploy
  ↓
Migrate data
  ↓
Switch usage
  ↓
Contract
```

---

# 55. Rollback Strategy

Application rollback:

```text id="n3f6w8"
Previous frontend deployment
Previous backend deployment
```

Database rollback requires greater care.

Avoid relying on destructive down-migrations as the primary recovery mechanism.

Prefer forward-fix migrations where possible.

---

# 56. Release Strategy

The MVP can use:

```text id="d4m8x2"
Continuous deployment after main-branch merge
```

provided:

- CI passes
- Review is complete
- Production checks pass

Major releases should have explicit release notes.

---

# 57. Zero-Downtime Considerations

The application should be designed so normal deployments do not require extended downtime.

Requirements:

- Backward-compatible API changes
- Safe migrations
- Health checks
- Graceful process shutdown
- Proper connection handling

---

# 58. Graceful Shutdown

FastAPI must gracefully handle shutdown signals.

The application should:

- Stop accepting new work
- Complete active requests where possible
- Close database connections
- Close external clients
- Exit cleanly

---

# 59. Dependency Health

Critical external dependencies include:

```text id="r5n9x1"
PostgreSQL
Object Storage
Fast2SMS
Payment Provider
```

The application should distinguish between:

```text id="7p3k5m"
Application failure
Dependency failure
```

so incidents can be diagnosed accurately.

---

# 60. External Service Configuration

External providers should be configurable.

Example:

```text id="8n2m7v"
OTP_PROVIDER=fast2sms
PAYMENT_PROVIDER=razorpay
STORAGE_PROVIDER=supabase
```

This allows provider replacement without rewriting application architecture.

---

# 61. Fast2SMS Infrastructure

Fast2SMS is used for OTP delivery.

Backend:

```text id="h6x9m4"
FastAPI
   ↓
Fast2SMS API
```

The API key remains backend-only.

The system should handle provider failures gracefully.

---

# 62. Payment Infrastructure

Payment provider will initially be either:

```text id="v8c2m5"
Razorpay
```

or:

```text id="x4j7n3"
Cashfree
```

Production selection must be finalized before payment implementation.

The selected provider must support:

- Subscription billing
- Webhooks
- Payment status verification
- Refund handling where required

---

# 63. Payment Environment Separation

Use provider test/sandbox credentials for:

```text id="n7m4x2"
Development
Staging
```

Production credentials only belong in production.

Never test real payments casually in production.

---

# 64. Payment Webhook Infrastructure

Webhook endpoint:

```text id="w4k8p3"
POST /api/v1/webhooks/payments/{provider}
```

The endpoint must be reachable from the payment provider.

It must use HTTPS.

---

# 65. Webhook Monitoring

Monitor:

- Webhook success
- Signature failures
- Duplicate events
- Processing errors
- Delayed processing

Payment webhook failures should be treated as high-priority operational events.

---

# 66. Storage Monitoring

Monitor:

- Storage usage
- Upload failures
- File access failures
- Unexpected growth

Future lifecycle policies may automatically remove orphaned files.

---

# 67. Orphaned File Handling

An uploaded file may exist in object storage while its database record fails to be created.

The storage system should therefore have a cleanup strategy.

Possible future process:

```text id="r3f8v6"
Scheduled cleanup
      ↓
Find unreferenced objects
      ↓
Verify age
      ↓
Delete safely
```

This should not be implemented destructively until the reference model is stable.

---

# 68. Database Performance Monitoring

Monitor:

- Slow queries
- Connection utilization
- Query frequency
- Database CPU
- Storage
- Lock contention

Initial optimization should focus on:

```text id="g4v7n2"
Indexes
Query efficiency
Pagination
Connection pooling
```

---

# 69. Scaling Strategy

The MVP should scale vertically before introducing complex distributed infrastructure.

Initial approach:

```text id="p6r8v1"
Increase backend resources
Increase database resources
Optimize queries
```

Only later consider:

```text id="k2x5m9"
Redis
Workers
Read replicas
Dedicated services
```

---

# 70. Backend Scaling

If API traffic increases:

```text id="n8m3v5"
Render instance
      ↓
Increase CPU/memory
      ↓
Multiple instances if supported/required
```

The backend must remain stateless enough to support horizontal scaling.

Session state should not depend on local process memory.

---

# 71. Frontend Scaling

Next.js static and cached content can be served through Vercel's infrastructure.

The frontend should avoid unnecessary server-side workload.

Large assets should use optimized image delivery.

---

# 72. Database Scaling

When database load increases:

1. Optimize queries
2. Add appropriate indexes
3. Review connection pooling
4. Increase database resources
5. Consider read replicas
6. Consider architectural changes only when justified

---

# 73. CDN

Vercel provides CDN capabilities for frontend assets.

Object storage/CDN optimization can be introduced for uploaded media as usage grows.

---

# 74. Image Optimization

User-uploaded images may require:

- Size validation
- Compression
- Thumbnail generation
- Responsive versions

For MVP, the system may store validated images without complex processing.

Image optimization can be introduced when storage or performance requirements justify it.

---

# 75. Local Development with Docker

Docker Compose may provide local infrastructure.

Example:

```text id="u3f8q1"
docker-compose.yml
```

Potential services:

```text id="8m4v2x"
postgres
```

Additional services should only be added when required.

The frontend and backend can still run directly on the host during development.

---

# 76. Example Local Architecture

```text id="p8w4r6"
Browser
   │
   ▼
Next.js :3000
   │
   ▼
FastAPI :8000
   │
   ▼
PostgreSQL :5432
```

---

# 77. Local Service Configuration

Development should use:

```text id="y7m3v9"
DATABASE_URL=postgresql://...
CORS_ORIGINS=http://localhost:3000
APP_ENV=development
```

External integrations should use test credentials where available.

---

# 78. Infrastructure as Code

The initial MVP does not require a complex infrastructure-as-code platform.

However, infrastructure configuration should be documented and version-controlled where possible.

Future options:

```text id="m5x8q2"
Terraform
Pulumi
Provider-specific configuration
```

These should be introduced when infrastructure complexity justifies them.

---

# 79. Production Access

Production infrastructure access must be restricted.

Only authorized project administrators should have access to:

- Production database
- Production storage
- Production secrets
- Payment dashboard
- SMS provider dashboard
- Hosting dashboards

---

# 80. Database Access

Developers should not routinely use direct production database access.

When production access is required:

- Use controlled credentials
- Record the purpose
- Avoid destructive queries
- Prefer read-only access when possible

---

# 81. Production Data Safety

Before running manual production operations:

```text id="k7n3w4"
Verify environment
      ↓
Verify target
      ↓
Backup if required
      ↓
Execute controlled operation
      ↓
Verify result
```

Never assume a local command is safe to run against production.

---

# 82. Monitoring Alerts

Critical alerts should include:

### Application

- High 5xx rate
- Application unavailable
- High latency

### Database

- Database unavailable
- High connection utilization
- Storage approaching limit

### Payments

- Webhook failures
- Payment processing errors

### Authentication

- Abnormal OTP traffic
- Repeated verification failures

---

# 83. Alert Severity

Suggested levels:

```text id="6x3k8m"
CRITICAL
HIGH
MEDIUM
LOW
```

Example:

```text id="5g9r2n"
Database unavailable → CRITICAL
Payment webhook failures → HIGH
Elevated API latency → MEDIUM
Single failed request → LOW
```

---

# 84. Disaster Recovery

Disaster recovery must address:

```text id="v3n7m1"
Database loss
Application deployment failure
Storage failure
Credential compromise
Hosting outage
Payment provider outage
```

---

# 85. Recovery Strategy

If the application fails:

```text id="c6f8q2"
Identify failure
      ↓
Contain
      ↓
Restore application
      ↓
Restore database if required
      ↓
Verify health
      ↓
Verify critical flows
      ↓
Resume service
```

---

# 86. Recovery Objectives

The project should define:

### RPO

Maximum acceptable amount of data loss.

### RTO

Maximum acceptable time to restore service.

Initial target values should be selected based on:

- Product stage
- Hosting capabilities
- Budget
- Customer requirements

Exact production RPO/RTO values should be finalized before launch.

---

# 87. Domain and DNS Recovery

Domain access must not depend on a single individual's account.

The business should maintain secure access to:

- Domain registrar
- DNS provider
- Vercel
- Render
- Supabase
- Payment provider
- SMS provider

Recovery credentials must be stored securely.

---

# 88. Account Ownership

Production infrastructure accounts should use company-controlled identities rather than personal accounts wherever possible.

At minimum, critical services should have:

- More than one authorized administrator
- Recovery email
- MFA
- Secure password management

---

# 89. Multi-Factor Authentication for Infrastructure

MFA should be enabled for:

```text id="s8f2m4"
GitHub
Vercel
Render
Supabase
Payment provider
SMS provider
DNS/domain provider
Monitoring provider
```

---

# 90. Infrastructure Security Checklist

Before production:

- [ ] GitHub repository protected
- [ ] Branch protection enabled
- [ ] MFA enabled on infrastructure accounts
- [ ] Production secrets configured
- [ ] Development/staging/production separated
- [ ] HTTPS enabled
- [ ] DNS configured
- [ ] CORS restricted
- [ ] Database backups enabled
- [ ] Storage configured
- [ ] Payment production credentials configured securely
- [ ] SMS production credentials configured securely
- [ ] Health checks configured
- [ ] Error monitoring configured
- [ ] Logs configured
- [ ] Alerts configured
- [ ] Database migrations tested
- [ ] Rollback strategy documented
- [ ] Recovery procedure documented

---

# 91. Production Deployment Checklist

Before first production release:

### Application

- [ ] Frontend production build succeeds
- [ ] Backend production build succeeds
- [ ] API health check succeeds
- [ ] Frontend can communicate with API

### Database

- [ ] Production database created
- [ ] Migrations applied
- [ ] Constraints verified
- [ ] Backups enabled

### Authentication

- [ ] OTP provider configured
- [ ] OTP delivery tested
- [ ] Session security verified
- [ ] Rate limiting verified

### Payments

- [ ] Production provider selected
- [ ] Production credentials configured
- [ ] Webhook URL configured
- [ ] Webhook signature verification tested
- [ ] Payment flow tested

### Storage

- [ ] Production bucket configured
- [ ] Private access configured
- [ ] Upload flow tested
- [ ] Signed URLs tested

### Operations

- [ ] Monitoring configured
- [ ] Alerts configured
- [ ] Logs accessible
- [ ] Recovery procedure documented

---

# 92. Release Validation

After deployment:

```text id="r8f4m2"
Open application
      ↓
Login
      ↓
Verify OTP
      ↓
Create event
      ↓
Create task
      ↓
Create vendor
      ↓
Add vendor payment
      ↓
Add guest
      ↓
Add idea
      ↓
Upload image
      ↓
Verify dashboard
      ↓
Verify subscription state
```

This should be part of the production smoke test.

---

# 93. Smoke Tests

Critical production smoke tests:

```text id="e4j8n1"
GET /health
GET /auth/me
Create event
Create task
Create vendor
Create guest
Create idea
Upload image
Read dashboard
Read subscription
```

Payment tests should use the provider's approved production verification process.

---

# 94. Operational Documentation

The repository should contain operational instructions for:

```text id="j5m7x3"
Deploying frontend
Deploying backend
Running migrations
Rolling back application
Restoring database
Rotating secrets
Updating payment credentials
Updating SMS credentials
Managing domains
Responding to incidents
```

---

# 95. Infrastructure Definition of Done

Infrastructure is considered ready when:

- [ ] GitHub repository configured
- [ ] Branch protection configured
- [ ] Vercel project configured
- [ ] Render project configured
- [ ] Supabase database configured
- [ ] Object storage configured
- [ ] Development environment works
- [ ] Staging environment works
- [ ] Production environment works
- [ ] Environment variables configured
- [ ] DNS configured
- [ ] HTTPS configured
- [ ] CORS configured
- [ ] Database migrations configured
- [ ] Database backups enabled
- [ ] CI/CD configured
- [ ] Health checks configured
- [ ] Error monitoring configured
- [ ] Logging configured
- [ ] Alerts configured
- [ ] OTP provider configured
- [ ] Payment provider configured
- [ ] Payment webhooks configured
- [ ] File uploads tested
- [ ] Production smoke tests pass
- [ ] Recovery process documented

---

# 96. Final Infrastructure Architecture

The target MVP infrastructure is:

```text id="m4q7x8"
                         ┌───────────────────┐
                         │       GitHub      │
                         │   Source + CI/CD  │
                         └─────────┬─────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
             ┌──────────────┐              ┌──────────────┐
             │    Vercel    │              │    Render    │
             │   Next.js    │              │   FastAPI    │
             └──────┬───────┘              └──────┬───────┘
                    │                             │
                    │ HTTPS                       │
                    └─────────────┐      ┌────────┘
                                  │      │
                                  ▼      ▼
                              ┌──────────────┐
                              │   Supabase   │
                              │  PostgreSQL  │
                              └──────────────┘

                              ┌──────────────┐
                              │    Object    │
                              │   Storage    │
                              └──────────────┘

                    External Services
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
          Fast2SMS              Razorpay/
                                Cashfree
```

---

# 97. Final Infrastructure Principle

The infrastructure should remain **managed, secure and intentionally simple** for the MVP.

The initial production system should therefore prefer:

```text id="x7c2m4"
Managed PostgreSQL
+
Managed frontend hosting
+
Managed backend hosting
+
Managed object storage
+
Managed payment provider
+
Managed SMS provider
```

rather than operating custom servers, Kubernetes clusters, self-hosted databases or unnecessary distributed infrastructure.

The infrastructure should evolve only when actual traffic, reliability, cost or product requirements justify the additional complexity.

---

# 98. Related Documentation

This document should be implemented alongside:

```text id="q2m6v8"
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/03-user-experience/user-flows.md
docs/03-user-experience/screen-inventory.md

docs/04-architecture/architecture.md
docs/05-database/database-design.md
docs/06-api/api-specification.md
docs/08-security/security.md
```
