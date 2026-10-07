# Security Architecture and Requirements

**Document:** `docs/08-security/security.md`  
**Status:** Draft  
**Version:** 1.0  
**Application:** Event Planner  
**Backend:** FastAPI  
**Frontend:** Next.js  
**Database:** PostgreSQL  
**Authentication:** Mobile Number + OTP  
**Initial Market:** India

---

# 1. Purpose

This document defines the security architecture, requirements, controls and implementation standards for the Event Planner application.

The application will process personal and business-related information including:

- User names
- Mobile numbers
- Email addresses
- Guest names and contact information
- Vendor contact information
- Event information
- Uploaded images and files
- Subscription information
- Payment metadata
- Authentication information

Security must therefore be designed into the application from the beginning rather than added after development.

---

# 2. Security Objectives

The application must protect:

1. **Confidentiality**
   - Users must only access information they are authorized to access.

2. **Integrity**
   - Unauthorized users must not be able to modify application data.

3. **Availability**
   - Security controls must not unnecessarily prevent legitimate users from using the service.

4. **Authentication**
   - Users must be reliably identified.

5. **Authorization**
   - Every protected resource must enforce access control.

6. **Privacy**
   - Only required personal information should be collected and retained.

7. **Auditability**
   - Important security-sensitive actions should be traceable.

---

# 3. Security Principles

The application follows these principles:

### Least privilege

Users, services and administrators receive only the permissions they require.

### Defense in depth

Security should exist at multiple layers:

```text
Browser
   ↓
HTTPS
   ↓
Frontend
   ↓
API
   ↓
Authentication
   ↓
Authorization
   ↓
Business Logic
   ↓
Database
   ↓
Storage
```

### Never trust the client

The frontend is considered untrusted.

The backend must independently validate:

- Identity
- Authorization
- Input
- Ownership
- Subscription status
- Payment status
- File metadata

### Secure by default

New endpoints, resources and integrations should default to restricted access.

---

# 4. Threat Model

The application should account for the following threats.

| Threat             | Example                      | Primary Control              |
| ------------------ | ---------------------------- | ---------------------------- |
| Account takeover   | Attacker obtains OTP         | OTP controls + rate limiting |
| OTP brute force    | Repeated OTP attempts        | Attempt limits               |
| Session theft      | Stolen session cookie        | Secure HTTP-only cookies     |
| IDOR               | User changes `event_id`      | Server-side authorization    |
| Data leakage       | Unauthorized API response    | Access control               |
| SQL injection      | Malicious query input        | Parameterized queries/ORM    |
| XSS                | Malicious user content       | Output encoding/sanitization |
| CSRF               | Unauthorized browser request | SameSite/CSRF controls       |
| File upload attack | Malicious file upload        | Type/size validation         |
| Payment fraud      | Fake payment success         | Verified webhooks            |
| Webhook replay     | Repeated payment event       | Idempotency                  |
| Credential leakage | API key committed to Git     | Secret management            |
| Abuse              | OTP request flooding         | Rate limiting                |
| Data loss          | Database failure             | Backups                      |
| Supply-chain risk  | Vulnerable dependency        | Dependency scanning          |

---

# 5. Authentication

Authentication uses:

```text id="gq4s6e"
Mobile Number
      ↓
OTP
      ↓
Authenticated Session
```

No password is required for the initial MVP.

---

# 6. Mobile Number Normalization

Mobile numbers must be normalized before authentication operations.

Example canonical representation:

```text id="vdyvse"
+919876543210
```

The application must avoid treating different formatting variations as separate accounts.

Examples that should normalize to the same identity:

```text id="3vhz2s"
9876543210
+91 9876543210
+91-9876543210
```

---

# 7. OTP Security

OTP is a high-risk authentication component.

The following controls are mandatory:

- OTP expiration
- Maximum verification attempts
- Resend cooldown
- Request rate limiting
- Verification rate limiting
- Mobile number normalization
- OTP invalidation after successful verification
- OTP invalidation after expiration
- Protection against OTP enumeration
- No plaintext OTP storage

---

# 8. OTP Expiration

OTP validity should be short.

Recommended initial validity:

```text id="0l5v9r"
5 minutes
```

The exact value should be configurable.

The backend must use server-side timestamps.

---

# 9. OTP Attempts

A maximum number of verification attempts must be enforced.

Example:

```text id="4v0c7s"
Maximum attempts: 5
```

After the limit is reached, the OTP should become invalid.

The exact limit should be configurable.

---

# 10. OTP Resend

OTP resend requests must have a cooldown.

Recommended initial value:

```text id="e8h7d6"
30 seconds
```

Repeated requests must be rate limited.

The user should receive a clear message without revealing unnecessary authentication details.

---

# 11. OTP Storage

If OTPs are stored in the database, the actual OTP value must never be stored in plaintext.

Instead:

```text id="jy1w68"
OTP
 ↓
Cryptographic hash
 ↓
Database
```

During verification:

```text id="7s90l7"
Submitted OTP
 ↓
Hash
 ↓
Compare
```

OTP records should contain:

```text id="8jgx0u"
otp_hash
expires_at
attempt_count
verified_at
purpose
```

---

# 12. OTP Provider

The initial provider is expected to be Fast2SMS.

The authentication system must not tightly couple the core authentication logic to the provider.

```text id="7n0gqk"
Authentication Service
        ↓
OTP Provider Interface
        ↓
Fast2SMS
```

This allows another SMS provider to be introduced later without redesigning authentication.

---

# 13. OTP Provider Secrets

The Fast2SMS API key must:

- Exist only on the backend
- Be stored as a secret environment variable
- Never be sent to the frontend
- Never be committed to Git
- Never appear in logs

Example:

```text id="g7w9ls"
FAST2SMS_API_KEY
```

---

# 14. Session Security

Authenticated sessions should use secure HTTP-only cookies.

Recommended properties:

```text id="6f0t0h"
HttpOnly
Secure
SameSite
```

Production cookies must only be transmitted over HTTPS.

---

# 15. Session Tokens

If server-side sessions are used, session tokens must not be stored in plaintext.

Recommended architecture:

```text id="s1g06r"
Browser
   ↓
Session token
   ↓
Hash
   ↓
Database
```

The database stores the token hash.

---

# 16. Session Expiration

Sessions must have an expiration policy.

The application should support:

- Session expiration
- Explicit logout
- Session revocation
- Revocation after security-sensitive events where appropriate

Exact session lifetime should be configurable.

---

# 17. Logout

Logout must invalidate the authenticated session on the backend.

The frontend clearing local UI state is not sufficient.

Flow:

```text id="h1gk9m"
POST /auth/logout
       ↓
Revoke session
       ↓
Clear authentication cookie
```

---

# 18. Authorization

Authentication determines:

```text id="1o8h6s"
Who are you?
```

Authorization determines:

```text id="f44h0p"
What are you allowed to access?
```

Every protected endpoint must enforce authorization server-side.

---

# 19. Event-Level Authorization

Event ownership is the primary authorization boundary.

```text id="f5t2se"
User
  ↓
Event Membership
  ↓
Event
  ↓
Event Resources
```

Event resources include:

```text id="9qkt44"
Tasks
Vendors
Vendor Payments
Guests
Ideas
Files
```

---

# 20. IDOR Protection

The application must protect against Insecure Direct Object Reference vulnerabilities.

Unsafe implementation:

```text id="l2o4tj"
GET /events/{event_id}/tasks
```

without checking whether the user owns the event.

Correct flow:

```text id="5r7q8a"
Authenticated User
       ↓
Find Event
       ↓
Verify Membership
       ↓
Verify Resource Belongs to Event
       ↓
Return Data
```

Changing a UUID in a request must never provide unauthorized access.

---

# 21. Cross-Event Authorization

A resource must belong to the event specified in the request.

Example:

```text id="3q1vzt"
EVENT-A
   └── TASK-A

EVENT-B
   └── TASK-B
```

A request such as:

```text id="xqz5ik"
GET /events/EVENT-A/tasks/TASK-B
```

must be rejected.

---

# 22. Authorization Failure

Unauthorized access should return:

```http id="0q4hjm"
403 Forbidden
```

Resource existence should not be unnecessarily disclosed.

In some cases, returning `404 Not Found` for inaccessible resources may be preferable to avoid resource enumeration.

The exact behavior should be standardized by endpoint type.

---

# 23. Subscription Authorization

Subscription status is determined by the backend.

The frontend must never be trusted to determine whether a user has access.

```text id="sv4fji"
User Request
     ↓
Authentication
     ↓
Authorization
     ↓
Subscription Check
     ↓
Business Operation
```

---

# 24. Trial Security

Trial dates must be calculated from server-controlled timestamps.

Example:

```text id="89fb7u"
trial_started_at
trial_ends_at
```

The client must not be able to modify:

```text id="5z13uc"
trial_ends_at
subscription status
payment status
```

---

# 25. Payment Security

The browser must never be the source of truth for successful payment.

Unsafe:

```text id="y8tq1h"
Frontend says:
"Payment successful"
        ↓
Activate subscription
```

Correct:

```text id="c91x3z"
Payment Provider
       ↓
Verified Webhook
       ↓
FastAPI
       ↓
Database
       ↓
Subscription ACTIVE
```

---

# 26. Payment Provider Secrets

Payment secrets must remain backend-only.

Examples:

```text id="h4d2b9"
RAZORPAY_KEY_ID
RAZORPAY_KEY_SECRET
```

or equivalent Cashfree credentials.

Only public client-side keys may be exposed to the browser when explicitly required by the provider.

---

# 27. Payment Webhook Verification

Every payment webhook must be verified.

The backend must validate:

1. Provider signature
2. Event structure
3. Provider transaction identifier
4. Expected payment/subscription relationship
5. Idempotency

Unverified webhook requests must be rejected.

---

# 28. Payment Webhook Replay Protection

Payment providers may retry webhook delivery.

The backend must handle duplicate events safely.

Use a unique provider event/payment identifier.

Example:

```text id="wd4f7x"
provider_payment_id UNIQUE
```

Repeated processing must not:

- Create duplicate payments
- Extend subscriptions incorrectly
- Charge the customer again
- Create duplicate records

---

# 29. Financial Data Integrity

Financial values must use:

```sql id="u0j9fq"
NUMERIC(12,2)
```

Never use floating-point values for money.

The backend must validate:

```text id="j4f0cs"
amount >= 0
```

---

# 30. Database Security

The database must not be directly accessible from the public internet unless specifically required by the managed provider architecture.

Application access should be:

```text id="h5m0n9"
Next.js
   ↓
FastAPI
   ↓
PostgreSQL
```

The frontend must never contain database credentials.

---

# 31. Database Credentials

Database credentials must:

- Exist only in backend configuration
- Be stored in secret management
- Never be committed to Git
- Never be logged
- Be rotated when required

Example:

```text id="8v0jv3"
DATABASE_URL
```

---

# 32. SQL Injection Protection

All database operations must use:

- Parameterized queries
- ORM query mechanisms
- Safe query builders

Never construct SQL using raw user input.

Unsafe:

```python id="q9t4qk"
query = f"SELECT * FROM guests WHERE name = '{name}'"
```

Safe approaches should use bound parameters.

---

# 33. Input Validation

All API input must be validated.

Validation must cover:

- Data type
- Length
- Format
- Range
- Allowed values
- Required fields

FastAPI/Pydantic schemas should be used for API validation.

---

# 34. Output Validation

API responses should only expose fields required by the client.

The API must not accidentally return:

- OTP hashes
- Session token hashes
- Payment secrets
- Internal database credentials
- Internal provider credentials
- Sensitive audit metadata

---

# 35. Mass Assignment Protection

The API must explicitly define fields that can be modified.

Clients must not be able to submit arbitrary database fields.

For example, a user update request must not allow:

```json id="35j2n6"
{
  "is_admin": true,
  "subscription_status": "ACTIVE"
}
```

unless those fields are explicitly supported by an authorized backend operation.

---

# 36. API Rate Limiting

Rate limiting must be implemented for abuse-sensitive endpoints.

Mandatory targets:

```text id="bjc0v7"
/auth/request-otp
/auth/resend-otp
/auth/verify-otp
```

Additional targets:

```text id="39mmq0"
/subscription/checkout
/webhooks/*
file uploads
```

Exact limits should be configurable.

---

# 37. Rate Limiting Strategy

Rate limiting may use multiple dimensions:

```text id="7v0zqf"
IP address
+
Mobile number
+
User account
+
Endpoint
```

This prevents attackers from bypassing limits simply by changing one identifier.

---

# 38. Brute-Force Protection

Authentication endpoints must protect against:

- OTP guessing
- Automated OTP requests
- Account enumeration
- Credential stuffing
- Distributed abuse

Repeated failures may trigger temporary blocking or progressively stricter rate limits.

---

# 39. Account Enumeration Protection

Authentication responses should avoid revealing unnecessary information.

For example, OTP request responses should generally not reveal whether a mobile number already belongs to an account.

The system should use consistent messaging where practical.

---

# 40. CORS Security

Production CORS must explicitly allow trusted origins.

Example:

```text id="j1uxjj"
https://app.example.com
```

Avoid:

```text id="x5hl6w"
Access-Control-Allow-Origin: *
```

for authenticated production APIs.

---

# 41. HTTPS

All production communication must use HTTPS.

Required:

```text id="3o6lzw"
Browser → Next.js
Browser → FastAPI
FastAPI → External Providers
FastAPI → Database
```

HTTP should redirect to HTTPS where applicable.

---

# 42. Transport Security

Sensitive information must never be transmitted through insecure channels.

Never send:

- OTP
- Session credentials
- Payment credentials
- API secrets

over plain HTTP in production.

---

# 43. CSRF Protection

Because authentication uses browser cookies, CSRF protection must be considered.

Controls should include:

```text id="0x1t1s"
SameSite cookies
+
Origin validation
+
CSRF protection where required
```

State-changing requests must be protected according to the final cookie/session architecture.

---

# 44. XSS Protection

User-generated content includes:

- Event names
- Task descriptions
- Vendor notes
- Guest notes
- Idea descriptions

The frontend must render user-controlled text safely.

React's default escaping should be preserved.

Avoid unsafe HTML rendering unless explicitly required.

---

# 45. HTML Sanitization

If future features allow rich text or HTML content, the application must sanitize HTML using a trusted sanitizer before rendering.

The MVP should avoid arbitrary HTML input wherever possible.

---

# 46. Open Redirect Protection

The application must not redirect users to arbitrary URLs based solely on untrusted query parameters.

External URLs saved as idea references must be treated as data, not authentication redirects.

---

# 47. File Upload Security

File uploads are a significant security boundary.

The backend must validate:

- File size
- MIME type
- Extension
- File content where practical
- Ownership
- Storage destination

---

# 48. Allowed File Types

Initial image uploads should support:

```text id="y4os4j"
image/jpeg
image/png
image/webp
```

Additional file types require explicit approval.

---

# 49. File Size Limits

A maximum upload size must be enforced server-side.

Example initial policy:

```text id="i7n5f2"
Maximum image size: 10 MB
```

The exact limit should be configurable.

The frontend may enforce the same limit for user experience, but the backend remains authoritative.

---

# 50. File Storage Security

Files should be stored using generated storage keys.

Example:

```text id="p9g5m0"
events/{event_id}/ideas/{idea_id}/{uuid}.jpg
```

Do not use:

```text id="p7h4d6"
events/{event_id}/{original_filename}
```

as the primary storage path.

---

# 51. Private Object Storage

User-uploaded files should preferably use private object storage.

Access should be granted through:

- Authorized API requests
- Short-lived signed URLs

rather than exposing permanent public storage URLs.

---

# 52. File Authorization

Before generating an access URL, the backend must verify:

```text id="f8m0nd"
User
 ↓
Event access
 ↓
Idea/event file ownership
 ↓
Generate URL
```

A user must not be able to access another user's uploaded image by guessing a storage key.

---

# 53. Malware Considerations

For MVP image-only uploads, strict type/size validation reduces risk.

If general document uploads are introduced later, the system should consider:

- Malware scanning
- Content inspection
- File quarantine
- Safe file processing

---

# 54. Dependency Security

The project will contain:

- Python packages
- Node.js packages
- Build tools
- Framework dependencies

Dependencies must be reviewed regularly.

The project should use:

```text id="0g7wy4"
Lock files
Dependency scanning
Security updates
```

Examples:

```text id="n9o4zj"
package-lock.json / pnpm-lock.yaml / yarn.lock
requirements lock / uv lock / equivalent
```

Only one package manager strategy should be selected for the project.

---

# 55. Dependency Updates

Dependency updates should be tested before production deployment.

Security-critical updates should be prioritized.

Major framework upgrades must be treated as planned engineering changes rather than automatic upgrades.

---

# 56. Environment Security

The application must have separate:

```text id="k3js44"
Development
Staging
Production
```

credentials and databases.

Production secrets must never be copied into local development environments.

---

# 57. Secret Management

Secrets include:

```text id="2o6f9v"
DATABASE_URL
SESSION_SECRET
FAST2SMS_API_KEY
PAYMENT_SECRET
STORAGE_ACCESS_KEY
STORAGE_SECRET_KEY
```

Secrets must be stored using:

- Vercel environment variables
- Render environment variables
- Supabase secret/configuration facilities
- Appropriate secret-management systems as the infrastructure grows

---

# 58. `.env` Rules

Local environment files must not be committed.

The repository should contain:

```text id="a6oxgn"
.env.example
```

but not:

```text id="y1k7iq"
.env
.env.local
.env.production
```

with real credentials.

---

# 59. Git Security

Before merging code:

- Scan for secrets
- Review configuration changes
- Review dependency changes
- Review authentication changes
- Review authorization changes

Secret-scanning tooling should be enabled where practical.

---

# 60. Logging Security

Logs must never contain:

```text id="q1l7gy"
OTP values
Session tokens
Passwords
API secrets
Payment secrets
Full card details
```

Logs may contain safe identifiers such as:

```text id="f2j8e1"
user_id
event_id
request_id
payment_id
```

provided they do not expose sensitive information.

---

# 61. Structured Logging

Backend logs should include:

```text id="j4w2ne"
timestamp
request_id
method
route
status_code
duration
environment
error_code
```

Sensitive request/response bodies should not automatically be logged.

---

# 62. Audit Logging

Security-sensitive operations should create audit records.

Examples:

```text id="x0v8f4"
USER_LOGIN
USER_LOGOUT
EVENT_CREATED
EVENT_DELETED
ACCOUNT_DELETED
SUBSCRIPTION_CREATED
SUBSCRIPTION_CANCELLED
PAYMENT_COMPLETED
PAYMENT_FAILED
```

---

# 63. Audit Log Integrity

Audit logs should not be editable by normal application users.

The application should not provide an endpoint allowing users to modify audit records.

Retention requirements will be defined separately.

---

# 64. Personal Data Minimization

The application should only collect data required for product functionality.

For example, guest records may require:

```text id="x8a7w2"
name
phone
RSVP
attendee count
meal preference
notes
```

The application should not require unnecessary information.

---

# 65. Guest Data

Guest information belongs to the event.

Therefore:

```text id="w3g7l8"
User
 ↓
Event authorization
 ↓
Guest data
```

Guests must not be globally searchable across users.

---

# 66. Vendor Data

Vendor information belongs to an event.

A vendor added to Event A must not automatically become visible to Event B.

Future vendor marketplace functionality will require a separate data model.

---

# 67. Data Isolation

The primary isolation boundary is:

```text id="l6b2n3"
Event
```

All event-owned data must contain an appropriate relationship to the event.

This makes authorization explicit and reduces accidental cross-event access.

---

# 68. Database Integrity

Database constraints must enforce important relationships.

Examples:

```text id="gr5h6f"
Foreign keys
NOT NULL
UNIQUE
CHECK constraints
```

Business logic should not rely exclusively on application code.

---

# 69. Database Encryption

The managed PostgreSQL provider should provide encryption at rest and encrypted connections.

The exact implementation depends on the selected managed PostgreSQL configuration.

Application-level encryption should only be introduced for particularly sensitive fields where there is a clear requirement.

---

# 70. Data in Transit

Sensitive traffic must use TLS.

This includes:

```text id="1cld7h"
Frontend ↔ API
API ↔ PostgreSQL
API ↔ Fast2SMS
API ↔ Payment Provider
API ↔ Object Storage
```

---

# 71. Backup Security

Database backups must:

- Be access controlled
- Use encryption provided by the managed service
- Have defined retention
- Not be publicly accessible
- Be periodically tested

Backups contain user data and must be treated as sensitive.

---

# 72. Account Deletion

Account deletion must follow a controlled workflow.

The process should determine:

```text id="w7f8v3"
User record
Events
Tasks
Vendors
Guests
Ideas
Files
Subscriptions
Payment history
Audit records
```

which data should be:

- Deleted
- Anonymized
- Retained for legal/accounting purposes

The final policy must be documented in the privacy policy and data-retention documentation.

---

# 73. Event Deletion

Deleting an event should not immediately leave orphaned data.

The deletion process must account for:

```text id="9t7f6k"
Tasks
Vendors
Vendor payments
Guests
Ideas
Attachments
Files
Notifications
```

The exact cleanup strategy should follow the database deletion policy.

---

# 74. Privacy by Design

The application should follow privacy-by-design principles:

- Minimize collection
- Restrict access
- Protect stored information
- Avoid unnecessary duplication
- Define retention
- Provide account deletion controls
- Avoid unnecessary tracking
- Avoid exposing private event data

---

# 75. Third-Party Services

The application initially depends on external services such as:

```text id="r3q8mv"
Fast2SMS
Razorpay/Cashfree
Object Storage
Vercel
Render
Supabase
```

Each integration should be evaluated for:

- Data transmitted
- Authentication mechanism
- Security controls
- Failure behavior
- Logging
- Data retention
- Provider availability

---

# 76. Third-Party Data Minimization

Only required data should be sent to external providers.

For example:

### SMS provider

Required:

```text id="w4w0j8"
mobile number
OTP/message
```

Do not send unnecessary event or guest information.

### Payment provider

Send only the information required for payment processing.

---

# 77. Webhook Endpoint Security

Webhook endpoints must:

- Accept only supported providers
- Validate signatures
- Validate payload structure
- Enforce idempotency
- Log safely
- Reject invalid requests
- Avoid exposing detailed failure information

---

# 78. Security Headers

The frontend/API deployment should use appropriate security headers.

Potential headers include:

```text id="5u9m2w"
Content-Security-Policy
Strict-Transport-Security
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
```

Exact CSP configuration must be tested carefully because overly restrictive policies can break legitimate application functionality.

---

# 79. Content Security Policy

A Content Security Policy should be introduced once the external resources required by the application are known.

The policy should restrict:

- Script sources
- Frame sources
- Image sources
- Connect sources
- Font sources

Only required origins should be allowed.

---

# 80. Clickjacking Protection

The application should prevent unauthorized framing where appropriate.

Use appropriate:

```text id="j7j8e0"
Content-Security-Policy: frame-ancestors
```

or equivalent platform configuration.

---

# 81. API Documentation Security

Interactive API documentation may expose endpoint information.

Production `/docs` access should be:

- Disabled, or
- Protected, or
- Restricted according to deployment requirements

OpenAPI JSON exposure should be reviewed similarly.

---

# 82. Error Handling

Production errors must not expose:

- Stack traces
- SQL queries
- File system paths
- Environment variables
- Provider secrets
- Internal architecture details

Users should receive safe error messages.

Internal logs should contain sufficient information for debugging.

---

# 83. Security Monitoring

Production monitoring should detect:

- Authentication failures
- OTP abuse
- Rate-limit events
- Payment webhook failures
- Repeated authorization failures
- Server errors
- Database failures
- Storage failures

---

# 84. Suspicious Activity

The system should be capable of identifying unusual patterns such as:

```text id="z3e9q4"
Hundreds of OTP requests
Repeated failed OTP verification
Repeated access-denied requests
Unusual file upload volume
Repeated payment failures
Webhook signature failures
```

The initial response may be rate limiting and logging.

More advanced fraud detection can be introduced later.

---

# 85. Security Testing

Security testing must cover:

### Authentication

- Invalid OTP
- Expired OTP
- OTP brute force
- OTP resend abuse
- Session expiration
- Logout

### Authorization

- Cross-user access
- Cross-event access
- Unauthorized update
- Unauthorized deletion
- Resource enumeration

### Input

- Invalid types
- Oversized values
- Malformed JSON
- SQL injection attempts
- XSS payloads

### Files

- Unsupported MIME types
- Oversized files
- Invalid extensions
- Unauthorized file access

### Payments

- Invalid webhook signatures
- Duplicate webhooks
- Fake payment success
- Subscription state manipulation

---

# 86. Security Testing Example

For every event-owned endpoint, test:

```text id="5u5xq2"
User A owns Event A
User B owns Event B

User A → Event A → ALLOW
User B → Event B → ALLOW

User A → Event B → DENY
User B → Event A → DENY
```

This test pattern should be reused across:

```text id="6f2k6z"
Tasks
Vendors
Payments
Guests
Ideas
Files
Dashboard
```

---

# 87. Production Security Checklist

Before production release:

- [ ] HTTPS enabled
- [ ] Secure cookies configured
- [ ] CORS restricted
- [ ] Authentication implemented
- [ ] Authorization implemented
- [ ] Event ownership verified
- [ ] OTP rate limits enabled
- [ ] OTP expiration enabled
- [ ] OTP attempts limited
- [ ] OTP values never logged
- [ ] Session tokens protected
- [ ] Payment webhooks verified
- [ ] Payment idempotency implemented
- [ ] File validation implemented
- [ ] Private storage configured
- [ ] Signed file URLs implemented where required
- [ ] Database credentials secured
- [ ] Secrets removed from source control
- [ ] Production error messages sanitized
- [ ] Security headers configured
- [ ] Dependency vulnerabilities reviewed
- [ ] Database backups enabled
- [ ] Backup restoration tested
- [ ] Monitoring enabled
- [ ] Audit logging enabled
- [ ] Account deletion flow tested

---

# 88. Security Definition of Done

The security implementation is complete when:

- [ ] Authentication is secure
- [ ] OTP lifecycle is protected
- [ ] Sessions are secure
- [ ] Authorization is enforced server-side
- [ ] IDOR protections are tested
- [ ] Subscription enforcement is backend-controlled
- [ ] Payment webhooks are verified
- [ ] Payment operations are idempotent
- [ ] API rate limiting exists
- [ ] CORS is restricted
- [ ] HTTPS is enforced
- [ ] Input validation exists
- [ ] Database queries are parameterized
- [ ] File uploads are validated
- [ ] Object storage is protected
- [ ] Secrets are securely managed
- [ ] Logs do not contain secrets
- [ ] Audit logs are implemented
- [ ] Security headers are configured
- [ ] Dependencies are scanned
- [ ] Backups are configured
- [ ] Account deletion behavior is defined
- [ ] Security tests pass
- [ ] Production security review is completed

---

# 89. Security Incident Response

If a security incident occurs:

```text id="3n3h0a"
Detect
  ↓
Contain
  ↓
Investigate
  ↓
Remediate
  ↓
Recover
  ↓
Review
```

Potential incidents include:

- Compromised API credentials
- Database exposure
- Account takeover
- Payment fraud
- Storage exposure
- Vulnerable dependency
- Unauthorized data access

Credentials should be rotated immediately when compromise is suspected.

---

# 90. Secret Rotation

The application must support rotation of:

```text id="8x6d7c"
Database credentials
SMS provider credentials
Payment credentials
Storage credentials
Session secrets
```

Rotation procedures should be documented in infrastructure operations.

---

# 91. Security Responsibility Model

| Layer            | Primary Responsibility           |
| ---------------- | -------------------------------- |
| Browser          | Secure interaction               |
| Next.js          | UI security and safe rendering   |
| FastAPI          | Authentication and authorization |
| Service layer    | Business security rules          |
| PostgreSQL       | Data integrity                   |
| Object Storage   | File security                    |
| Payment Provider | Payment processing               |
| SMS Provider     | OTP delivery                     |
| Hosting Provider | Infrastructure security          |

Security remains a shared responsibility across all layers.

---

# 92. Security Architecture

```text id="p0k6yc"
                         INTERNET
                            │
                         HTTPS
                            │
                            ▼
                  ┌──────────────────┐
                  │     Next.js      │
                  │    Frontend      │
                  └────────┬─────────┘
                           │
                         HTTPS
                           │
                           ▼
                  ┌──────────────────┐
                  │     FastAPI      │
                  │      API         │
                  └────────┬─────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
        Authentication  Authorization  Validation
              │            │            │
              └────────────┼────────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   Service Layer  │
                  └───────┬───┬──────┘
                          │   │
                ┌─────────┘   └─────────┐
                ▼                       ▼
        ┌──────────────┐        ┌──────────────┐
        │  PostgreSQL  │        │ Object Store │
        └──────────────┘        └──────────────┘

                    External Providers
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           Fast2SMS            Payment Provider
```

---

# 93. Security Priorities

Security implementation should follow this order:

```text id="d5xx0s"
1. Authentication
2. Authorization
3. Session security
4. Database protection
5. API validation
6. Payment security
7. File security
8. Rate limiting
9. Secret management
10. Logging and monitoring
11. Backup and recovery
12. Security testing
```

---

# 94. Final Security Principle

The most important security rule for the application is:

> **Never trust anything coming from the client.**

The frontend can request an operation, but only the backend can determine:

```text id="t7b8o1"
Who the user is
        ↓
What event they can access
        ↓
What operation they can perform
        ↓
Whether their subscription permits it
        ↓
Whether the data is valid
        ↓
Whether the operation should be persisted
```

Security must therefore be enforced at the **FastAPI + PostgreSQL + storage boundaries**, not merely through Next.js UI restrictions.

---

# 95. Related Documentation

This document is based on:

```text id="q7z8we"
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/03-user-experience/user-flows.md
docs/03-user-experience/screen-inventory.md

docs/04-architecture/architecture.md
docs/05-database/database-design.md
docs/06-api/api-specification.md
```

It should be implemented together with:

```text id="2e3j1r"
docs/09-infrastructure/infrastructure.md
docs/10-development/development-guide.md
```
