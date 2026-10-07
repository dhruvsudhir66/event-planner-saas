# API Specification

**Document:** `docs/06-api/api-specification.md`  
**Status:** Draft  
**Version:** 1.0  
**API Style:** REST / JSON  
**API Version:** `/api/v1`  
**Backend:** FastAPI  
**Database:** PostgreSQL  
**Authentication:** Mobile number + OTP  
**Primary Client:** Next.js web application

---

# 1. Purpose

This document defines the HTTP API contract for the Event Planner application.

It specifies:

- API conventions
- Authentication
- Authorization
- Endpoints
- Request schemas
- Response schemas
- Validation
- Pagination
- Filtering
- Error handling
- Subscription enforcement
- File upload behavior
- Payment webhooks
- HTTP status codes

The API is the contract between the Next.js frontend and FastAPI backend.

---

# 2. API Design Principles

The API must be:

1. **RESTful**
2. **Versioned**
3. **Consistent**
4. **Predictable**
5. **Secure**
6. **Stateless at the HTTP request level**
7. **Explicit about authorization**
8. **Idempotent where required**
9. **Easy to consume from TypeScript**
10. **Backward-compatible where possible**

---

# 3. Base URL

Development:

```text
http://localhost:8000/api/v1
```

Staging:

```text
https://api-staging.<domain>/api/v1
```

Production:

```text
https://api.<domain>/api/v1
```

The exact production domain will be finalized during infrastructure setup.

---

# 4. API Versioning

All application endpoints must be versioned.

```text
/api/v1/
```

Example:

```text
GET /api/v1/events
```

Breaking changes should introduce a new API version.

```text
/api/v2/
```

Non-breaking additions may remain within the current version.

---

# 5. Authentication Model

Authentication is based on:

```text
Mobile Number
       ↓
OTP
       ↓
Authenticated Session
```

The browser receives a secure authenticated session after successful OTP verification.

The preferred session mechanism is a secure HTTP-only cookie.

---

# 6. Authentication Flow

```text
User
 │
 │ mobile number
 ▼
POST /auth/request-otp
 │
 ▼
Fast2SMS
 │
 ▼
User receives OTP
 │
 │ OTP
 ▼
POST /auth/verify-otp
 │
 ▼
Create/find user
 │
 ▼
Create session
 │
 ▼
Authenticated
```

---

# 7. Authentication Endpoints

## 7.1 Request OTP

```http
POST /api/v1/auth/request-otp
```

### Purpose

Requests an OTP for registration or login.

### Request

```json
{
  "mobile_number": "+919876543210"
}
```

### Validation

- Mobile number is required.
- Mobile number must be valid.
- Mobile number must be normalized.
- Rate limits must be applied.
- Resend cooldown must be enforced.

### Response

```json
{
  "message": "OTP sent successfully.",
  "expires_in": 300,
  "resend_after": 30
}
```

### Status Codes

| Status | Meaning                  |
| ------ | ------------------------ |
| `200`  | OTP sent                 |
| `400`  | Invalid mobile number    |
| `429`  | Rate limit exceeded      |
| `503`  | SMS provider unavailable |

---

# 8. Verify OTP

```http
POST /api/v1/auth/verify-otp
```

### Purpose

Verifies the OTP and authenticates the user.

### Request

```json
{
  "mobile_number": "+919876543210",
  "otp": "123456"
}
```

### Response

```json
{
  "user": {
    "id": "uuid",
    "mobile_number": "+919876543210",
    "name": null
  },
  "is_new_user": true
}
```

A secure authenticated session is established through the response cookie.

### Status Codes

| Status | Meaning           |
| ------ | ----------------- |
| `200`  | OTP verified      |
| `400`  | Invalid OTP       |
| `410`  | OTP expired       |
| `429`  | Too many attempts |

---

# 9. Resend OTP

```http
POST /api/v1/auth/resend-otp
```

### Request

```json
{
  "mobile_number": "+919876543210"
}
```

### Response

```json
{
  "message": "OTP sent successfully.",
  "expires_in": 300,
  "resend_after": 30
}
```

---

# 10. Change Mobile Number

This endpoint is used during OTP authentication when the user wants to correct the entered number.

```http
POST /api/v1/auth/change-mobile
```

### Request

```json
{
  "mobile_number": "+919876543210"
}
```

The user must then complete OTP verification for the new number.

---

# 11. Current User

```http
GET /api/v1/auth/me
```

### Purpose

Returns the currently authenticated user.

### Response

```json
{
  "id": "uuid",
  "mobile_number": "+919876543210",
  "name": "Aneena",
  "email": null,
  "profile_image_url": null
}
```

### Status Codes

```text
200
401
```

---

# 12. Logout

```http
POST /api/v1/auth/logout
```

### Response

```json
{
  "message": "Logged out successfully."
}
```

The server must revoke or invalidate the authenticated session.

---

# 13. Authentication Requirements

Protected endpoints must require an authenticated session.

Unauthenticated requests must return:

```http
401 Unauthorized
```

The API must never rely solely on frontend route protection.

---

# 14. Event API

Base resource:

```text
/events
```

---

# 15. List Events

```http
GET /api/v1/events
```

### Purpose

Returns events accessible to the authenticated user.

### Query Parameters

```text
page
page_size
```

Example:

```http
GET /api/v1/events?page=1&page_size=20
```

### Response

```json
{
  "items": [
    {
      "id": "uuid",
      "name": "Aneena & Loyed Wedding",
      "event_type": "WEDDING",
      "event_date": "2026-11-22",
      "location": "Kerala",
      "image_url": null,
      "currency": "INR"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 1,
    "total_pages": 1
  }
}
```

---

# 16. Create Event

```http
POST /api/v1/events
```

### Request

```json
{
  "name": "Aneena & Loyed Wedding",
  "event_type": "WEDDING",
  "event_date": "2026-11-22",
  "location": "Kerala",
  "description": "Wedding planning",
  "currency": "INR"
}
```

### Required

```text
name
event_type
event_date
```

### Response

```json
{
  "id": "uuid",
  "name": "Aneena & Loyed Wedding",
  "event_type": "WEDDING",
  "event_date": "2026-11-22",
  "location": "Kerala",
  "description": "Wedding planning",
  "currency": "INR",
  "created_at": "2026-10-07T10:00:00Z",
  "updated_at": "2026-10-07T10:00:00Z"
}
```

### Status

```text
201 Created
```

---

# 17. Get Event

```http
GET /api/v1/events/{event_id}
```

Returns an event accessible to the current user.

### Status Codes

```text
200
401
403
404
```

---

# 18. Update Event

```http
PATCH /api/v1/events/{event_id}
```

### Request

```json
{
  "name": "Aneena & Loyed Wedding",
  "location": "Kochi, Kerala"
}
```

Only supplied fields are updated.

### Status

```text
200 OK
```

---

# 19. Delete Event

```http
DELETE /api/v1/events/{event_id}
```

The default behavior is a controlled/soft deletion.

### Response

```text
204 No Content
```

The API must verify that the authenticated user has permission to delete the event.

---

# 20. Event Authorization

Every event endpoint must perform:

```text
Authenticated User
        ↓
Event Exists
        ↓
User Has Event Access
        ↓
Permission Check
        ↓
Operation
```

A user must not be able to access another user's event by modifying the `event_id`.

---

# 21. Dashboard API

```http
GET /api/v1/events/{event_id}/dashboard
```

### Purpose

Returns the information required by the main dashboard.

### Response

```json
{
  "event": {
    "id": "uuid",
    "name": "Aneena & Loyed Wedding",
    "event_type": "WEDDING",
    "event_date": "2026-11-22",
    "location": "Kerala"
  },
  "progress": {
    "total_tasks": 20,
    "completed_tasks": 12,
    "percentage": 60
  },
  "tasks": {
    "pending": 8,
    "upcoming": 4,
    "overdue": 2
  },
  "vendors": {
    "total": 8,
    "estimated_cost": 350000,
    "actual_cost": 320000,
    "paid": 180000,
    "pending": 140000
  },
  "guests": {
    "total": 120,
    "attending": 80,
    "not_attending": 10,
    "pending": 30
  },
  "subscription": {
    "status": "TRIAL",
    "trial_ends_at": "2026-10-10T10:00:00Z"
  }
}
```

Dashboard data should be derived from domain entities rather than stored in a separate dashboard table.

---

# 22. Task API

Base resource:

```text
/events/{event_id}/tasks
```

---

# 23. List Tasks

```http
GET /api/v1/events/{event_id}/tasks
```

### Query Parameters

```text
page
page_size
status
priority
category
```

Example:

```http
GET /api/v1/events/{event_id}/tasks?status=PENDING&priority=HIGH
```

---

# 24. Create Task

```http
POST /api/v1/events/{event_id}/tasks
```

### Request

```json
{
  "title": "Book wedding photographer",
  "description": "Compare shortlisted photographers.",
  "due_date": "2026-10-20",
  "priority": "HIGH",
  "category": "PHOTOGRAPHY"
}
```

### Response

```json
{
  "id": "uuid",
  "event_id": "uuid",
  "title": "Book wedding photographer",
  "description": "Compare shortlisted photographers.",
  "due_date": "2026-10-20",
  "priority": "HIGH",
  "category": "PHOTOGRAPHY",
  "status": "PENDING",
  "completed_at": null,
  "created_at": "2026-10-07T10:00:00Z",
  "updated_at": "2026-10-07T10:00:00Z"
}
```

### Status

```text
201 Created
```

---

# 25. Get Task

```http
GET /api/v1/events/{event_id}/tasks/{task_id}
```

---

# 26. Update Task

```http
PATCH /api/v1/events/{event_id}/tasks/{task_id}
```

Example:

```json
{
  "title": "Book wedding photographer",
  "priority": "MEDIUM"
}
```

---

# 27. Complete Task

```http
POST /api/v1/events/{event_id}/tasks/{task_id}/complete
```

### Response

```json
{
  "id": "uuid",
  "status": "COMPLETED",
  "completed_at": "2026-10-07T12:30:00Z"
}
```

---

# 28. Reopen Task

```http
POST /api/v1/events/{event_id}/tasks/{task_id}/reopen
```

### Response

```json
{
  "id": "uuid",
  "status": "PENDING",
  "completed_at": null
}
```

---

# 29. Delete Task

```http
DELETE /api/v1/events/{event_id}/tasks/{task_id}
```

Default behavior:

```text
204 No Content
```

---

# 30. Vendor API

Base resource:

```text
/events/{event_id}/vendors
```

---

# 31. List Vendors

```http
GET /api/v1/events/{event_id}/vendors
```

### Query Parameters

```text
page
page_size
category
payment_status
```

---

# 32. Create Vendor

```http
POST /api/v1/events/{event_id}/vendors
```

### Request

```json
{
  "name": "Example Photography",
  "category": "PHOTOGRAPHY",
  "phone": "+919876543210",
  "email": "hello@example.com",
  "estimated_cost": 75000,
  "actual_cost": null,
  "notes": "Full-day coverage"
}
```

### Response

```json
{
  "id": "uuid",
  "event_id": "uuid",
  "name": "Example Photography",
  "category": "PHOTOGRAPHY",
  "phone": "+919876543210",
  "email": "hello@example.com",
  "estimated_cost": 75000,
  "actual_cost": null,
  "total_paid": 0,
  "pending_amount": 75000,
  "payment_status": "PENDING",
  "notes": "Full-day coverage",
  "created_at": "2026-10-07T10:00:00Z",
  "updated_at": "2026-10-07T10:00:00Z"
}
```

---

# 33. Get Vendor

```http
GET /api/v1/events/{event_id}/vendors/{vendor_id}
```

---

# 34. Update Vendor

```http
PATCH /api/v1/events/{event_id}/vendors/{vendor_id}
```

Example:

```json
{
  "actual_cost": 70000,
  "notes": "Final negotiated amount"
}
```

---

# 35. Delete Vendor

```http
DELETE /api/v1/events/{event_id}/vendors/{vendor_id}
```

---

# 36. Vendor Payments

## List Payments

```http
GET /api/v1/events/{event_id}/vendors/{vendor_id}/payments
```

---

## Add Payment

```http
POST /api/v1/events/{event_id}/vendors/{vendor_id}/payments
```

### Request

```json
{
  "amount": 25000,
  "payment_date": "2026-10-07",
  "payment_method": "UPI",
  "reference": "UPI123456",
  "notes": "Advance payment"
}
```

### Response

```json
{
  "id": "uuid",
  "vendor_id": "uuid",
  "amount": 25000,
  "payment_date": "2026-10-07",
  "payment_method": "UPI",
  "reference": "UPI123456",
  "notes": "Advance payment",
  "created_at": "2026-10-07T10:00:00Z"
}
```

---

# 37. Delete Vendor Payment

```http
DELETE /api/v1/events/{event_id}/vendors/{vendor_id}/payments/{payment_id}
```

The backend must verify that the payment belongs to the specified vendor and event.

---

# 38. Guest API

Base resource:

```text
/events/{event_id}/guests
```

---

# 39. List Guests

```http
GET /api/v1/events/{event_id}/guests
```

### Query Parameters

```text
page
page_size
rsvp_status
group_name
```

Example:

```http
GET /api/v1/events/{event_id}/guests?rsvp_status=ATTENDING
```

---

# 40. Create Guest

```http
POST /api/v1/events/{event_id}/guests
```

### Request

```json
{
  "name": "John Mathew",
  "phone": "+919876543210",
  "group_name": "Bride Family",
  "rsvp_status": "PENDING",
  "attendee_count": 1,
  "meal_preference": "VEGETARIAN",
  "notes": null
}
```

### Response

```json
{
  "id": "uuid",
  "event_id": "uuid",
  "name": "John Mathew",
  "phone": "+919876543210",
  "group_name": "Bride Family",
  "rsvp_status": "PENDING",
  "attendee_count": 1,
  "meal_preference": "VEGETARIAN",
  "notes": null,
  "created_at": "2026-10-07T10:00:00Z",
  "updated_at": "2026-10-07T10:00:00Z"
}
```

---

# 41. Get Guest

```http
GET /api/v1/events/{event_id}/guests/{guest_id}
```

---

# 42. Update Guest

```http
PATCH /api/v1/events/{event_id}/guests/{guest_id}
```

Example:

```json
{
  "rsvp_status": "ATTENDING",
  "attendee_count": 2
}
```

---

# 43. Delete Guest

```http
DELETE /api/v1/events/{event_id}/guests/{guest_id}
```

---

# 44. Guest Summary

```http
GET /api/v1/events/{event_id}/guests/summary
```

### Response

```json
{
  "total": 120,
  "pending": 30,
  "attending": 80,
  "not_attending": 10,
  "maybe": 0,
  "total_attendees": 95
}
```

The exact calculation rules will be finalized during implementation.

---

# 45. Idea API

Base resource:

```text
/events/{event_id}/ideas
```

---

# 46. List Ideas

```http
GET /api/v1/events/{event_id}/ideas
```

### Query Parameters

```text
page
page_size
category
```

---

# 47. Create Idea

```http
POST /api/v1/events/{event_id}/ideas
```

### Request

```json
{
  "title": "Outdoor ceremony setup",
  "description": "Use white flowers with greenery.",
  "category": "DECOR",
  "external_url": "https://example.com/inspiration"
}
```

### Response

```json
{
  "id": "uuid",
  "event_id": "uuid",
  "title": "Outdoor ceremony setup",
  "description": "Use white flowers with greenery.",
  "category": "DECOR",
  "external_url": "https://example.com/inspiration",
  "attachments": [],
  "created_at": "2026-10-07T10:00:00Z",
  "updated_at": "2026-10-07T10:00:00Z"
}
```

---

# 48. Get Idea

```http
GET /api/v1/events/{event_id}/ideas/{idea_id}
```

---

# 49. Update Idea

```http
PATCH /api/v1/events/{event_id}/ideas/{idea_id}
```

---

# 50. Delete Idea

```http
DELETE /api/v1/events/{event_id}/ideas/{idea_id}
```

---

# 51. Idea Attachment Upload

```http
POST /api/v1/events/{event_id}/ideas/{idea_id}/attachments
```

Content type:

```text
multipart/form-data
```

Field:

```text
file
```

### Supported Initial File Types

```text
image/jpeg
image/png
image/webp
```

Maximum file size must be enforced by the backend.

The exact limit will be finalized in the infrastructure/security configuration.

---

# 52. List Idea Attachments

```http
GET /api/v1/events/{event_id}/ideas/{idea_id}/attachments
```

### Response

```json
{
  "items": [
    {
      "id": "uuid",
      "original_filename": "decor.jpg",
      "mime_type": "image/jpeg",
      "file_size": 245678,
      "url": "https://storage.example.com/...",
      "created_at": "2026-10-07T10:00:00Z"
    }
  ]
}
```

URLs should preferably be short-lived signed URLs when the storage is private.

---

# 53. Delete Idea Attachment

```http
DELETE /api/v1/events/{event_id}/ideas/{idea_id}/attachments/{attachment_id}
```

The backend must remove or invalidate the corresponding object-storage file.

---

# 54. Event Image

The event cover image can use:

```http
POST /api/v1/events/{event_id}/image
```

Content type:

```text
multipart/form-data
```

The backend must:

1. Authenticate the user.
2. Verify event access.
3. Validate file.
4. Upload to object storage.
5. Update the event's image reference.

---

# 55. Subscription API

Base resource:

```text
/subscription
```

---

# 56. Get Subscription

```http
GET /api/v1/subscription
```

### Response

```json
{
  "id": "uuid",
  "plan_code": "MONTHLY",
  "status": "TRIAL",
  "trial_started_at": "2026-10-07T10:00:00Z",
  "trial_ends_at": "2026-10-10T10:00:00Z",
  "current_period_start": null,
  "current_period_end": null
}
```

---

# 57. Get Available Plans

```http
GET /api/v1/subscription/plans
```

### Response

```json
{
  "items": [
    {
      "code": "MONTHLY",
      "name": "Monthly",
      "amount": 499,
      "currency": "INR",
      "interval": "MONTH"
    }
  ]
}
```

Actual pricing will be finalized in the business documentation.

---

# 58. Create Checkout

```http
POST /api/v1/subscription/checkout
```

### Request

```json
{
  "plan_code": "MONTHLY"
}
```

### Response

```json
{
  "checkout_id": "provider-order-id",
  "provider": "RAZORPAY",
  "amount": 499,
  "currency": "INR"
}
```

The frontend uses the returned information to initiate the provider's checkout process.

---

# 59. Subscription Access Check

```http
GET /api/v1/subscription/access
```

### Response

```json
{
  "has_access": true,
  "status": "TRIAL",
  "reason": null,
  "trial_ends_at": "2026-10-10T10:00:00Z"
}
```

Possible reasons:

```text
TRIAL_ACTIVE
SUBSCRIPTION_ACTIVE
TRIAL_EXPIRED
SUBSCRIPTION_EXPIRED
PAYMENT_FAILED
SUBSCRIPTION_CANCELLED
```

---

# 60. Cancel Subscription

```http
POST /api/v1/subscription/cancel
```

### Response

```json
{
  "status": "CANCELLED",
  "cancelled_at": "2026-10-07T10:00:00Z"
}
```

The exact cancellation behavior depends on the payment provider and billing policy.

---

# 61. Payment History

```http
GET /api/v1/subscription/payments
```

### Response

```json
{
  "items": [
    {
      "id": "uuid",
      "amount": 499,
      "currency": "INR",
      "status": "SUCCESS",
      "paid_at": "2026-10-07T10:00:00Z"
    }
  ]
}
```

---

# 62. Payment Webhook

Payment providers call a backend webhook endpoint.

```http
POST /api/v1/webhooks/payments/{provider}
```

Example:

```text
POST /api/v1/webhooks/payments/razorpay
```

or:

```text
POST /api/v1/webhooks/payments/cashfree
```

---

# 63. Payment Webhook Requirements

The webhook handler must:

1. Validate request authenticity.
2. Verify provider signature.
3. Parse event.
4. Check idempotency.
5. Locate relevant payment/subscription.
6. Update database.
7. Return successful acknowledgement.

The handler must not trust payment status supplied by the frontend.

---

# 64. Webhook Idempotency

Provider transaction/event identifiers must be unique.

Repeated webhook delivery must not create duplicate payments or subscriptions.

Example:

```text
provider_payment_id UNIQUE
```

---

# 65. Settings API

---

# 66. Get Profile

```http
GET /api/v1/settings/profile
```

### Response

```json
{
  "id": "uuid",
  "name": "Aneena",
  "mobile_number": "+919876543210",
  "email": null,
  "profile_image_url": null
}
```

---

# 67. Update Profile

```http
PATCH /api/v1/settings/profile
```

### Request

```json
{
  "name": "Aneena Vincent",
  "email": "aneena@example.com"
}
```

---

# 68. Notification Preferences

```http
GET /api/v1/settings/notifications
```

### Response

```json
{
  "task_reminders": true,
  "subscription_alerts": true
}
```

---

# 69. Update Notification Preferences

```http
PATCH /api/v1/settings/notifications
```

### Request

```json
{
  "task_reminders": false,
  "subscription_alerts": true
}
```

---

# 70. Delete Account

```http
DELETE /api/v1/settings/account
```

The endpoint must use a controlled account deletion workflow.

The exact retention and anonymization rules are defined in the security/privacy documentation.

---

# 71. Notifications API

## List Notifications

```http
GET /api/v1/notifications
```

### Query Parameters

```text
page
page_size
unread_only
```

---

# 72. Mark Notification Read

```http
POST /api/v1/notifications/{notification_id}/read
```

### Response

```json
{
  "id": "uuid",
  "is_read": true,
  "read_at": "2026-10-07T10:00:00Z"
}
```

---

# 73. Mark All Notifications Read

```http
POST /api/v1/notifications/read-all
```

### Response

```json
{
  "updated_count": 5
}
```

---

# 74. Pagination

All collection endpoints should use a consistent pagination format.

Query parameters:

```text
page
page_size
```

Default:

```text
page = 1
page_size = 20
```

Maximum:

```text
page_size = 100
```

Example:

```http
GET /api/v1/events/{event_id}/guests?page=2&page_size=20
```

---

# 75. Pagination Response

```json
{
  "items": [],
  "pagination": {
    "page": 2,
    "page_size": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

The exact pagination strategy may evolve to cursor-based pagination if required at scale.

---

# 76. Filtering

Filtering uses query parameters.

Example:

```http
GET /api/v1/events/{event_id}/tasks?status=PENDING&priority=HIGH
```

Multiple filters should be combined using:

```text
AND
```

unless explicitly documented otherwise.

---

# 77. Sorting

Collection endpoints may support sorting.

Example:

```text
sort_by=due_date
sort_order=asc
```

Example:

```http
GET /api/v1/events/{event_id}/tasks?sort_by=due_date&sort_order=asc
```

Only explicitly supported sort fields should be accepted.

The API must reject arbitrary database column names supplied by clients.

---

# 78. Search

MVP search can use PostgreSQL-backed filtering.

Example:

```http
GET /api/v1/events/{event_id}/guests?search=john
```

Search behavior should be documented separately for each endpoint.

The backend must sanitize and safely parameterize search values.

---

# 79. Standard Success Response

Single resources should normally return the resource directly.

Example:

```json
{
  "id": "uuid",
  "name": "Wedding",
  "event_date": "2026-11-22"
}
```

Collections use:

```json
{
  "items": [],
  "pagination": {}
}
```

---

# 80. Standard Error Response

All API errors should follow a consistent structure.

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request could not be processed.",
    "details": null
  }
}
```

---

# 81. Validation Error

Example:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields are invalid.",
    "details": {
      "event_date": "Event date is required."
    }
  }
}
```

---

# 82. Authentication Error

```json
{
  "error": {
    "code": "AUTHENTICATION_REQUIRED",
    "message": "Authentication is required."
  }
}
```

Status:

```text
401
```

---

# 83. Authorization Error

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to access this resource."
  }
}
```

Status:

```text
403
```

---

# 84. Resource Not Found

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found."
  }
}
```

Status:

```text
404
```

The API should avoid revealing whether another user's resource exists.

---

# 85. Conflict Error

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "The requested operation conflicts with the current state."
  }
}
```

Status:

```text
409
```

---

# 86. Rate Limit Error

```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests. Please try again later."
  }
}
```

Status:

```text
429
```

---

# 87. Internal Error

Production responses must not expose stack traces.

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Something went wrong. Please try again later."
  }
}
```

Status:

```text
500
```

The complete exception should be logged internally.

---

# 88. Request IDs

Each API request should have a request/correlation ID.

Example header:

```text
X-Request-ID: 8b5e3a7c-...
```

If the client provides a valid request ID, the backend may propagate it.

Otherwise the backend should generate one.

The request ID should appear in logs and error diagnostics.

---

# 89. API Authentication Headers

Cookie-based sessions are preferred for the web application.

If token-based authentication is introduced later, the standard format is:

```http
Authorization: Bearer <token>
```

The API should not support multiple authentication mechanisms unnecessarily during MVP.

---

# 90. Content Types

JSON requests:

```http
Content-Type: application/json
```

File uploads:

```http
Content-Type: multipart/form-data
```

Webhook payload handling must follow the payment provider's requirements.

---

# 91. API Security Requirements

The API must implement:

- Authentication
- Authorization
- Input validation
- Rate limiting
- CORS restrictions
- Secure cookies
- CSRF protection where applicable
- Parameterized database queries
- File validation
- Webhook signature validation
- Request logging
- Secret management

---

# 92. Rate Limiting

Rate limiting is especially important for:

```text
/auth/request-otp
/auth/resend-otp
/auth/verify-otp
```

It should also apply to:

```text
Payment endpoints
File uploads
Public-facing endpoints
```

The exact rate limits will be defined in security/infrastructure documentation.

---

# 93. Subscription Enforcement

Protected business APIs must verify subscription access according to product rules.

Example:

```text
Authenticated
     ↓
Subscription Access
     ↓
Allowed
     ↓
Business Operation
```

If access is expired:

```json
{
  "error": {
    "code": "SUBSCRIPTION_REQUIRED",
    "message": "An active subscription is required to continue."
  }
}
```

Status:

```text
403
```

Exceptions such as subscription management and account settings must remain accessible where required.

---

# 94. API Authorization Matrix

| Resource        | Auth Required |  Event Access Required | Subscription Required |
| --------------- | ------------: | ---------------------: | --------------------: |
| OTP             |            No |                     No |                    No |
| Current user    |           Yes |                     No |                    No |
| Events          |           Yes | Yes for existing event |                   Yes |
| Tasks           |           Yes |                    Yes |                   Yes |
| Vendors         |           Yes |                    Yes |                   Yes |
| Vendor payments |           Yes |                    Yes |                   Yes |
| Guests          |           Yes |                    Yes |                   Yes |
| Ideas           |           Yes |                    Yes |                   Yes |
| Files           |           Yes |                    Yes |                   Yes |
| Dashboard       |           Yes |                    Yes |                   Yes |
| Subscription    |           Yes |                     No |                    No |
| Payment history |           Yes |                     No |                    No |
| Settings        |           Yes |                     No |                    No |
| Notifications   |           Yes |                     No |                    No |
| Payment webhook | Provider auth |                     No |                    No |

---

# 95. Resource Authorization Rule

For any endpoint of the form:

```text
/events/{event_id}/...
```

the backend must verify:

```text
current_user
      ↓
event_members
      ↓
event_id
```

before accessing the child resource.

The child resource must also belong to the same event.

---

# 96. Example Authorization Failure

Request:

```http
GET /api/v1/events/EVENT-A/tasks/TASK-B
```

If `TASK-B` belongs to `EVENT-B`, the API must reject the request even if the user has access to `EVENT-A`.

This prevents cross-event resource access.

---

# 97. API Transaction Rules

The service layer must use database transactions when an operation modifies multiple related entities.

Examples:

```text
Create subscription + payment
Process payment webhook
Delete event + related resources
Create vendor + initial payment
```

Transactions must preserve database consistency.

---

# 98. Idempotency

Operations involving external providers or retryable requests must support idempotency.

Payment webhooks are mandatory candidates.

Future payment creation endpoints may support:

```http
Idempotency-Key: <unique-key>
```

The backend should document which endpoints support idempotency.

---

# 99. API Documentation

FastAPI should automatically expose OpenAPI documentation.

Development:

```text
/docs
```

Alternative schema:

```text
/openapi.json
```

Production access to interactive API documentation may be restricted depending on security requirements.

---

# 100. Type Generation

The API should eventually support generation of TypeScript types from the OpenAPI schema.

Conceptually:

```text
FastAPI
   ↓
OpenAPI
   ↓
TypeScript types
   ↓
Next.js
```

This reduces frontend/backend schema mismatches.

---

# 101. API Testing

The API must have automated tests covering:

### Authentication

- OTP request
- OTP verification
- Invalid OTP
- Expired OTP
- Rate limiting
- Logout

### Events

- Create
- Read
- Update
- Delete
- Unauthorized access

### Tasks

- CRUD
- Completion
- Reopening
- Filtering

### Vendors

- CRUD
- Payments
- Payment calculations

### Guests

- CRUD
- RSVP
- Filtering

### Ideas

- CRUD
- Attachments

### Subscription

- Trial
- Checkout
- Payment success
- Payment failure
- Expiration
- Webhooks

### Security

- Cross-user access
- Cross-event access
- Invalid sessions
- Missing authorization

---

# 102. API Test Layers

Testing should include:

```text
Unit Tests
     ↓
Service Tests
     ↓
API Integration Tests
     ↓
Database Tests
     ↓
End-to-End Tests
```

Critical authorization and subscription behavior must have integration coverage.

---

# 103. API Endpoint Summary

## Authentication

```text
POST   /auth/request-otp
POST   /auth/verify-otp
POST   /auth/resend-otp
POST   /auth/change-mobile
GET    /auth/me
POST   /auth/logout
```

## Events

```text
GET    /events
POST   /events
GET    /events/{event_id}
PATCH  /events/{event_id}
DELETE /events/{event_id}
```

## Dashboard

```text
GET    /events/{event_id}/dashboard
```

## Tasks

```text
GET    /events/{event_id}/tasks
POST   /events/{event_id}/tasks
GET    /events/{event_id}/tasks/{task_id}
PATCH  /events/{event_id}/tasks/{task_id}
DELETE /events/{event_id}/tasks/{task_id}
POST   /events/{event_id}/tasks/{task_id}/complete
POST   /events/{event_id}/tasks/{task_id}/reopen
```

## Vendors

```text
GET    /events/{event_id}/vendors
POST   /events/{event_id}/vendors
GET    /events/{event_id}/vendors/{vendor_id}
PATCH  /events/{event_id}/vendors/{vendor_id}
DELETE /events/{event_id}/vendors/{vendor_id}

GET    /events/{event_id}/vendors/{vendor_id}/payments
POST   /events/{event_id}/vendors/{vendor_id}/payments
DELETE /events/{event_id}/vendors/{vendor_id}/payments/{payment_id}
```

## Guests

```text
GET    /events/{event_id}/guests
POST   /events/{event_id}/guests
GET    /events/{event_id}/guests/{guest_id}
PATCH  /events/{event_id}/guests/{guest_id}
DELETE /events/{event_id}/guests/{guest_id}
GET    /events/{event_id}/guests/summary
```

## Ideas

```text
GET    /events/{event_id}/ideas
POST   /events/{event_id}/ideas
GET    /events/{event_id}/ideas/{idea_id}
PATCH  /events/{event_id}/ideas/{idea_id}
DELETE /events/{event_id}/ideas/{idea_id}

POST   /events/{event_id}/ideas/{idea_id}/attachments
GET    /events/{event_id}/ideas/{idea_id}/attachments
DELETE /events/{event_id}/ideas/{idea_id}/attachments/{attachment_id}
```

## Files

```text
POST   /events/{event_id}/image
```

## Subscription

```text
GET    /subscription
GET    /subscription/plans
POST   /subscription/checkout
GET    /subscription/access
POST   /subscription/cancel
GET    /subscription/payments
```

## Settings

```text
GET    /settings/profile
PATCH  /settings/profile

GET    /settings/notifications
PATCH  /settings/notifications

DELETE /settings/account
```

## Notifications

```text
GET    /notifications
POST   /notifications/{notification_id}/read
POST   /notifications/read-all
```

## Webhooks

```text
POST   /webhooks/payments/{provider}
```

---

# 104. API Definition of Done

The API implementation is considered complete when:

- [ ] `/api/v1` is implemented
- [ ] Authentication endpoints work
- [ ] Secure sessions are implemented
- [ ] Authorization is implemented
- [ ] Event CRUD is implemented
- [ ] Dashboard endpoint is implemented
- [ ] Task APIs are implemented
- [ ] Vendor APIs are implemented
- [ ] Vendor payment APIs are implemented
- [ ] Guest APIs are implemented
- [ ] Idea APIs are implemented
- [ ] File upload APIs are implemented
- [ ] Subscription APIs are implemented
- [ ] Payment webhook is implemented
- [ ] Settings APIs are implemented
- [ ] Notification APIs are implemented
- [ ] Pagination is standardized
- [ ] Filtering is standardized
- [ ] Error responses are standardized
- [ ] Request IDs are implemented
- [ ] Rate limiting is implemented
- [ ] API authorization tests exist
- [ ] Cross-event access tests exist
- [ ] Subscription enforcement tests exist
- [ ] OpenAPI documentation is generated
- [ ] Type generation strategy is established
- [ ] Integration tests pass
- [ ] Production security checks pass

---

# 105. Related Documentation

This document depends on:

```text
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/03-user-experience/user-flows.md
docs/03-user-experience/screen-inventory.md

docs/04-architecture/architecture.md
docs/05-database/database-design.md
```

It should be implemented alongside:

```text
docs/08-security/security.md
docs/09-infrastructure/infrastructure.md
docs/10-development/development-guide.md
```

---

# 106. Final API Principle

The API must provide a clear boundary between the Next.js application and the application's business logic.

```text
Next.js
   │
   │ HTTP / JSON
   ▼
FastAPI
   │
   ├── Authentication
   ├── Authorization
   ├── Validation
   ├── Business Logic
   ├── Subscription Enforcement
   ├── External Integrations
   │
   ▼
PostgreSQL / Object Storage
```

The most important rule is:

> **The frontend requests an operation; the backend decides whether that operation is valid and authorized.**

The API should remain predictable, versioned, secure and sufficiently simple for the MVP while providing a stable foundation for future capabilities.
