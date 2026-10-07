# Subscription and Billing Specification

**Document:** `docs/07-business/subscription.md`  
**Status:** Draft  
**Version:** 1.0  
**Application:** Event Planner  
**Business Model:** Free Trial → Paid Subscription  
**Initial Market:** India

---

# 1. Purpose

This document defines the subscription, trial, billing, payment, access-control and subscription-lifecycle requirements for the Event Planner SaaS.

It establishes the business rules that must be followed by:

- Frontend
- Backend
- Database
- Payment provider
- Webhooks
- Subscription UI
- Notifications
- Support and operations

The backend is the authoritative source for subscription status and access.

---

# 2. Business Model

The MVP uses a simple subscription model:

```text
New User
   ↓
3-Day Free Trial
   ↓
Trial Active
   ↓
Subscription Required
   ↓
Paid Subscription
   ↓
Continued Access
```

A user must subscribe after the free trial to continue using the application.

---

# 3. Commercial Objectives

The subscription system should:

1. Make the product easy to try.
2. Require minimal friction during onboarding.
3. Clearly communicate when the trial ends.
4. Clearly communicate pricing.
5. Prevent accidental loss of user data.
6. Restrict paid functionality server-side.
7. Support payment-provider changes.
8. Maintain accurate payment history.
9. Handle failed payments gracefully.
10. Provide a reliable audit trail.

---

# 4. MVP Subscription Model

The MVP initially supports:

- One free trial
- One paid subscription plan
- Recurring billing
- Subscription cancellation
- Payment history
- Subscription status
- Subscription expiry handling

The architecture should support multiple plans later.

---

# 5. Free Trial

The default trial duration is:

```text
3 calendar days
```

The exact trial start and end timestamps must be stored on the server.

Example:

```text
Trial starts:
2026-10-01 10:30 UTC

Trial ends:
2026-10-04 10:30 UTC
```

The backend must determine whether the trial is active.

The frontend must never calculate subscription eligibility as the authoritative source.

---

# 6. Trial Start

The trial begins when the user completes the required onboarding process and creates their first event.

Recommended flow:

```text
OTP Verification
      ↓
Create First Event
      ↓
Create Trial Subscription Record
      ↓
Trial Starts
      ↓
Dashboard
```

The exact business trigger should remain consistent across the application.

A user must not receive multiple free trials simply by creating multiple events.

---

# 7. One Trial Per User

A user is eligible for the free trial only once.

The system must prevent:

```text
User
 ↓
Trial
 ↓
Trial expires
 ↓
Delete event
 ↓
Create another event
 ↓
New trial
```

The trial belongs to the user account, not to an individual event.

---

# 8. Trial Status

The trial is considered active when:

```text
current_time < trial_ends_at
```

and the subscription has not transitioned to a paid subscription.

Recommended states:

```text
TRIAL
ACTIVE
PAST_DUE
CANCELLED
EXPIRED
```

---

# 9. Trial Countdown

The UI may display:

```text
3 days left
2 days left
1 day left
Trial ends today
```

The frontend may calculate display countdowns locally.

However, the backend remains authoritative for access.

---

# 10. Trial Expiration

When:

```text
current_time >= trial_ends_at
```

the trial is no longer active.

If the user has not subscribed:

```text
TRIAL
   ↓
EXPIRED
```

The user should no longer have normal access to paid application functionality.

---

# 11. Data Preservation After Trial

Trial expiration must **not automatically delete user data**.

The user's:

- Events
- Tasks
- Vendors
- Guests
- Ideas
- Files

should remain stored according to the application's retention policy.

The user should be able to subscribe later and regain access where business rules allow.

---

# 12. Subscription Plans

The database and API should support multiple plans even though the MVP may launch with only one.

Example structure:

| Field              | Description                   |
| ------------------ | ----------------------------- |
| `plan_code`        | Unique plan identifier        |
| `name`             | Display name                  |
| `billing_interval` | Monthly/yearly                |
| `price`            | Price                         |
| `currency`         | INR                           |
| `features`         | Included functionality        |
| `is_active`        | Whether plan can be purchased |

---

# 13. MVP Plan

The initial paid plan should be represented using a stable internal identifier.

Example:

```text
PLAN_CODE = EVENT_PLANNER_PRO
```

The exact commercial name and price are TBD.

The internal plan code should not depend on the payment provider's product ID.

---

# 14. Pricing

Pricing must be configurable rather than hardcoded throughout the frontend.

Example:

```text
Plan
├── plan_code
├── name
├── monthly_price
├── yearly_price
├── currency
└── features
```

The backend should provide active plan information through the API.

---

# 15. Currency

The initial market uses:

```text
INR
```

Money must be represented using exact decimal/numeric values.

Do not use floating-point values for monetary calculations.

---

# 16. Tax Handling

Tax requirements must be finalized before production billing.

The system should allow future support for:

- Base price
- Tax
- Total amount
- Invoice/reference information

Tax calculations should be performed server-side or by the payment/invoicing system where appropriate.

---

# 17. Subscription Lifecycle

The complete lifecycle is:

```text
TRIAL
  │
  ├── Payment successful
  │       ↓
  │    ACTIVE
  │
  └── Trial expires
          ↓
       EXPIRED

ACTIVE
  │
  ├── Renewal successful
  │       ↓
  │    ACTIVE
  │
  ├── Payment failure
  │       ↓
  │    PAST_DUE
  │
  └── Cancellation
          ↓
       CANCELLED
```

The exact provider-specific states must be mapped into the application's internal states.

---

# 18. Internal Subscription States

The application uses:

```text
TRIAL
ACTIVE
PAST_DUE
CANCELLED
EXPIRED
```

These states are provider-independent.

---

# 19. Provider Status Mapping

Payment-provider statuses must not be exposed directly throughout the application.

Instead:

```text
Payment Provider
      ↓
Provider Adapter
      ↓
Internal Subscription State
      ↓
Application
```

Example:

```text
Provider status
      ↓
ACTIVE
```

rather than allowing frontend code to depend on provider-specific values.

---

# 20. Subscription Record

The subscription database record should contain information such as:

```text
user_id
provider
provider_customer_id
provider_subscription_id
plan_code
status
trial_started_at
trial_ends_at
current_period_start
current_period_end
cancelled_at
created_at
updated_at
```

The database design is defined in:

```text
docs/05-database/database-design.md
```

---

# 21. One Active Subscription

A user should have at most one active subscription at a time.

Historical subscription records may be retained for billing history and auditing.

Business rule:

```text
One user
   ↓
Zero or one current active subscription
```

---

# 22. Subscription Ownership

Subscriptions belong to users, not individual events.

Example:

```text
User
 ├── Event A
 ├── Event B
 └── Subscription
```

One subscription can therefore support all events owned by that user, subject to the selected plan's limits.

---

# 23. Subscription Access

The backend should determine access using a centralized subscription service.

Example:

```text
Request
   ↓
Authentication
   ↓
Authorization
   ↓
Subscription Access Check
   ↓
Business Operation
```

---

# 24. Access Matrix

| Status    | Normal Application Access                                             |
| --------- | --------------------------------------------------------------------- |
| TRIAL     | Yes                                                                   |
| ACTIVE    | Yes                                                                   |
| PAST_DUE  | Grace period / restricted based on policy                             |
| CANCELLED | Until paid period ends or immediate, depending on cancellation policy |
| EXPIRED   | No                                                                    |

The exact PAST_DUE grace period must be finalized before launch.

---

# 25. Trial Access

During an active trial, users should have access to the complete MVP functionality.

The trial should not intentionally provide an artificially restricted version unless a separate free-plan strategy is introduced.

---

# 26. Expired Access

When the subscription expires:

```text
User
  ↓
Authentication still works
  ↓
Subscription access denied
  ↓
Subscription page shown
```

The user should still be able to:

- Log in
- View subscription status
- View billing information
- Start a subscription

They should not lose their account.

---

# 27. Subscription Paywall

When access is restricted, the application should clearly communicate:

- Why access is restricted
- Current subscription status
- Available plan
- Price
- Billing interval
- Call to action

Example:

```text
Your trial has ended.

Subscribe to continue planning your event.

[View subscription]
```

The final UI copy is TBD.

---

# 28. Paywall Behavior

The backend must reject protected operations when the user's subscription does not permit them.

Example:

```http
POST /api/v1/events/{event_id}/tasks
```

If subscription access is unavailable:

```http
403 Forbidden
```

with a machine-readable error code.

Example:

```json
{
  "error": {
    "code": "SUBSCRIPTION_REQUIRED",
    "message": "An active subscription is required.",
    "details": null
  }
}
```

---

# 29. Frontend Paywall

The frontend may proactively detect subscription state and redirect the user to the subscription page.

However:

```text
Frontend restriction ≠ Security
```

The backend must independently enforce the restriction.

---

# 30. Subscription API

The MVP API includes:

```text
GET  /api/v1/subscription
GET  /api/v1/subscription/plans
POST /api/v1/subscription/checkout
GET  /api/v1/subscription/access
POST /api/v1/subscription/cancel
GET  /api/v1/subscription/payments
```

Detailed contracts are defined in:

```text
docs/06-api/api-specification.md
```

---

# 31. Get Subscription

```http
GET /api/v1/subscription
```

Returns:

- Current status
- Plan
- Trial information
- Billing period
- Cancellation state
- Renewal information

Example:

```json
{
  "status": "TRIAL",
  "plan_code": null,
  "trial_ends_at": "2026-10-04T10:30:00Z",
  "current_period_end": null,
  "cancelled_at": null
}
```

---

# 32. Get Plans

```http
GET /api/v1/subscription/plans
```

Returns currently purchasable plans.

The frontend should use this endpoint rather than duplicating pricing configuration.

---

# 33. Checkout

```http
POST /api/v1/subscription/checkout
```

Request:

```json
{
  "plan_code": "EVENT_PLANNER_PRO",
  "billing_interval": "MONTHLY"
}
```

The backend should:

1. Validate the plan.
2. Validate the user's subscription state.
3. Create or retrieve the payment customer.
4. Create the payment order/subscription.
5. Return checkout information.

---

# 34. Payment Provider Abstraction

The backend should use a payment service interface.

Conceptually:

```python
class PaymentProvider:
    def create_customer(...)
    def create_subscription(...)
    def cancel_subscription(...)
    def verify_webhook(...)
    def get_subscription(...)
```

Provider-specific implementations should sit behind this abstraction.

Example:

```text
PaymentService
      │
      ├── RazorpayProvider
      │
      └── CashfreeProvider
```

---

# 35. Payment Initiation

The frontend must not directly create authoritative subscription records.

Correct flow:

```text
Frontend
   ↓
Backend
   ↓
Payment Provider
   ↓
Checkout
   ↓
Payment
   ↓
Webhook
   ↓
Backend
   ↓
Subscription ACTIVE
```

---

# 36. Payment Success

A successful checkout does not by itself mean the application should blindly trust the browser.

The backend must verify the payment through the payment provider and/or verified webhook.

Only then should:

```text
TRIAL
   ↓
ACTIVE
```

be established.

---

# 37. Payment Webhooks

Webhook endpoint:

```http
POST /api/v1/webhooks/payments/{provider}
```

The webhook handler must:

1. Validate the request.
2. Verify the provider signature.
3. Identify the event.
4. Check idempotency.
5. Update payment records.
6. Update subscription state.
7. Record the event.
8. Return an appropriate response.

---

# 38. Webhook Idempotency

The same webhook may be delivered more than once.

The system must safely handle duplicate events.

Use provider event IDs or equivalent unique identifiers.

Example:

```text
Webhook received
      ↓
Already processed?
   ├── Yes → Ignore safely
   └── No  → Process
```

---

# 39. Payment Records

Every payment should create a payment record.

Fields include:

```text
user_id
subscription_id
provider
provider_payment_id
provider_order_id
amount
currency
status
paid_at
created_at
```

---

# 40. Payment States

Internal payment states:

```text
PENDING
SUCCESS
FAILED
REFUNDED
```

Provider-specific payment states should be mapped into these internal states.

---

# 41. Failed Payment

When a renewal payment fails:

```text
ACTIVE
   ↓
PAST_DUE
```

The user should be notified.

The exact grace-period and retry policy must be configured before production launch.

---

# 42. Past Due

The `PAST_DUE` state represents a billing problem rather than immediate data deletion.

During this state the system may:

- Allow temporary access
- Show payment warnings
- Request payment recovery

The exact grace period is a business decision.

Recommended MVP approach:

```text
PAST_DUE
   ↓
Short grace period
   ↓
EXPIRED
```

---

# 43. Cancellation

The user may cancel their subscription through:

```http
POST /api/v1/subscription/cancel
```

Cancellation behavior must be explicitly defined.

Recommended SaaS behavior:

```text
Cancel subscription
       ↓
Disable future renewal
       ↓
Continue access until current paid period ends
       ↓
EXPIRED
```

This avoids unexpectedly removing paid access.

---

# 44. Cancellation Timestamp

The system should record:

```text
cancelled_at
```

The provider subscription should also be cancelled according to the provider's API.

---

# 45. Re-Subscription

An expired or cancelled user may subscribe again.

Flow:

```text
EXPIRED
   ↓
Choose plan
   ↓
Checkout
   ↓
Payment successful
   ↓
ACTIVE
```

Historical payment/subscription records should remain intact.

---

# 46. Payment History

Users should be able to view their payment history.

Example fields:

```text
Date
Description
Amount
Status
Payment reference
```

Endpoint:

```http
GET /api/v1/subscription/payments
```

---

# 47. Invoices

Invoice requirements are not part of the initial MVP unless required by the payment provider/business setup.

The architecture should allow invoices to be introduced later.

Future support may include:

- Invoice number
- Invoice date
- Tax information
- Billing address
- Downloadable invoice

---

# 48. Refunds

Refund functionality is not a primary MVP workflow.

However, payment records must support:

```text
REFUNDED
```

The backend should reconcile refund events from the payment provider.

---

# 49. Subscription Notifications

Important subscription events should generate notifications.

Examples:

```text
Trial ending
Trial expired
Payment successful
Payment failed
Subscription cancelled
Subscription renewed
```

The initial notification mechanism may be in-app.

SMS/email/WhatsApp notifications can be added later.

---

# 50. Trial Reminder Schedule

Recommended reminders:

```text
Trial started
      ↓
Trial ending soon
      ↓
Trial ends
```

Exact reminder timing should be finalized based on product requirements.

The notification system must avoid duplicate notifications.

---

# 51. Trial Expiration Job

Trial expiration may be evaluated lazily whenever the user accesses the system.

A scheduled background job can later be introduced to proactively transition records and send notifications.

For MVP:

```text
Server timestamp check
+
Optional scheduled processing
```

is sufficient.

---

# 52. Subscription State Source of Truth

The source of truth is:

```text
Backend + PostgreSQL
```

Not:

```text
Frontend local state
Browser storage
Payment success URL
Client-side countdown
```

---

# 53. Subscription Security

Subscription operations must require authentication.

Users must only access their own:

- Subscription
- Payment records
- Billing information

Payment webhook endpoints use provider signature verification rather than user authentication.

---

# 54. Subscription Abuse Prevention

The system must prevent:

- Multiple free trials
- Checkout manipulation
- Price manipulation
- Plan-code manipulation
- Unauthorized subscription access
- Fake payment-success requests
- Webhook replay
- Cross-user payment access

---

# 55. Price Integrity

The client must never be trusted to provide the final amount.

The backend should resolve:

```text
plan_code
+
billing_interval
      ↓
Authoritative price
```

The payment order/subscription must be created using the server-side price.

---

# 56. Example

Incorrect:

```json
{
  "plan_code": "PRO",
  "amount": 1
}
```

The backend should never trust the client-provided amount.

Correct:

```json
{
  "plan_code": "EVENT_PLANNER_PRO",
  "billing_interval": "MONTHLY"
}
```

The backend determines the price.

---

# 57. Subscription Audit Events

Important subscription actions should be recorded in audit logs.

Examples:

```text
TRIAL_CREATED
CHECKOUT_CREATED
PAYMENT_SUCCESS
PAYMENT_FAILED
SUBSCRIPTION_ACTIVATED
SUBSCRIPTION_CANCELLED
SUBSCRIPTION_EXPIRED
REFUND_RECEIVED
```

---

# 58. Business Metrics

The business should eventually track:

### Acquisition

- Trial registrations
- Trial activation rate

### Trial

- Trial-to-paid conversion
- Average time to subscription

### Revenue

- Monthly recurring revenue
- Annual recurring revenue
- Average revenue per user

### Retention

- Churn
- Renewal rate
- Reactivation rate

### Billing

- Payment success rate
- Failed payment rate

These analytics are not required to block MVP launch.

---

# 59. Core Subscription Metrics

The most important initial metrics are:

```text
Trial → Paid Conversion
Monthly Recurring Revenue
Churn
Payment Success Rate
Active Subscribers
```

---

# 60. Subscription Business Rules

The following rules are mandatory:

1. A user receives only one free trial.
2. Trial duration is three days.
3. Trial status is determined server-side.
4. Subscription belongs to the user.
5. A user has at most one active subscription.
6. Pricing is resolved server-side.
7. Payment status is verified server-side.
8. Webhooks are signature-verified.
9. Webhooks are idempotent.
10. Subscription access is enforced server-side.
11. Expired subscriptions do not delete user data.
12. Cancellation should normally stop renewal rather than immediately remove paid access.
13. Payment history is retained.
14. Provider-specific states are abstracted.
15. Payment credentials never reach the frontend.

---

# 61. Subscription State Machine

```text
                    ┌──────────────┐
                    │    TRIAL     │
                    └──────┬───────┘
                           │
                Payment    │    Trial expires
                succeeds   │
                           │
                           ▼
                    ┌──────────────┐
                    │    ACTIVE    │
                    └──────┬───────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
          Payment fails       User cancels
                 │                   │
                 ▼                   ▼
          ┌────────────┐      ┌────────────┐
          │  PAST_DUE  │      │  CANCELLED │
          └─────┬──────┘      └──────┬─────┘
                │                    │
          Payment fixed        Period ends
                │                    │
                ▼                    ▼
          ┌──────────────┐    ┌──────────────┐
          │    ACTIVE    │    │    EXPIRED   │
          └──────────────┘    └──────┬───────┘
                                     │
                                  Resubscribe
                                     │
                                     ▼
                                ┌────────────┐
                                │   ACTIVE   │
                                └────────────┘
```

---

# 62. Subscription UI Requirements

The subscription screen should clearly show:

- Current status
- Current plan
- Trial remaining time, if applicable
- Plan price
- Billing interval
- Renewal date
- Cancellation state
- Payment history
- Primary subscription action

Avoid unnecessary promotional UI.

The interface should prioritize clarity over marketing.

---

# 63. Subscription UX States

The UI must support:

```text
Trial Active
Trial Ending
Trial Expired
Payment Pending
Active
Past Due
Cancellation Scheduled
Cancelled
Expired
Payment Failed
```

Each state should have:

- Clear status
- Explanation
- Relevant action

---

# 64. Empty States

If payment history is empty:

```text
No payments yet.
```

If the user has no active subscription:

```text
You are currently using your free trial.
```

Exact copy is subject to UX review.

---

# 65. Loading States

Subscription pages must show loading states while retrieving:

- Subscription
- Plans
- Payment history

The user should never see a misleading "expired" state simply because the API has not finished loading.

---

# 66. Error States

The UI should distinguish:

```text
Network failure
Payment failure
Subscription unavailable
Checkout creation failure
Webhook pending
```

The user should receive a clear next action.

---

# 67. Checkout Failure

If checkout creation fails:

```text
Do not change subscription state.
```

The user should be able to retry.

---

# 68. Payment Pending

If payment has been initiated but confirmation has not yet arrived:

```text
Payment status: Pending
```

The frontend should not assume payment failure immediately.

The backend should reconcile the final state through provider confirmation/webhooks.

---

# 69. Webhook Failure Recovery

If a payment succeeds but the webhook is delayed:

```text
Payment Provider
      ↓
Payment successful
      ↓
Webhook delayed
      ↓
Subscription temporarily pending
      ↓
Webhook received
      ↓
Subscription ACTIVE
```

The system should provide a reconciliation mechanism where required.

---

# 70. Provider Abstraction

The business logic must not contain code such as:

```python
if razorpay_status == "...":
```

throughout the application.

Instead:

```text
Payment Provider
       ↓
Adapter
       ↓
Normalized Payment Event
       ↓
Subscription Service
```

This allows the provider to change without rewriting business logic.

---

# 71. Configuration

Provider configuration should be environment-based:

```text
PAYMENT_PROVIDER=razorpay
```

or:

```text
PAYMENT_PROVIDER=cashfree
```

Provider credentials remain server-side.

---

# 72. Subscription Testing

Automated tests must cover:

### Trial

- Trial creation
- Trial expiry
- One-trial-only rule

### Checkout

- Valid plan
- Invalid plan
- Duplicate active subscription

### Payment

- Success
- Failure
- Pending
- Refund

### Webhooks

- Valid signature
- Invalid signature
- Duplicate event
- Unknown event

### Cancellation

- Cancel active subscription
- Cancel already cancelled subscription
- Re-subscribe

### Access

- Trial access
- Active access
- Past-due access
- Expired access

---

# 73. Acceptance Criteria

Subscription functionality is complete when:

- [ ] New users receive one three-day trial
- [ ] Trial start/end timestamps are stored
- [ ] Trial status is server-controlled
- [ ] Trial cannot be restarted
- [ ] Plans are returned by API
- [ ] Pricing is server-controlled
- [ ] Checkout can be created
- [ ] Payment provider is abstracted
- [ ] Payment success is verified
- [ ] Webhooks are signature-verified
- [ ] Webhooks are idempotent
- [ ] Subscription state updates correctly
- [ ] Payment records are persisted
- [ ] Cancellation works
- [ ] Expired subscriptions are restricted
- [ ] User data remains preserved
- [ ] Payment history is accessible
- [ ] Subscription errors are handled
- [ ] Audit events are recorded
- [ ] Subscription tests pass

---

# 74. MVP Commercial Decisions Still Required

The following decisions must be finalized before production billing:

| Decision                     | Status                          |
| ---------------------------- | ------------------------------- |
| Paid plan name               | TBD                             |
| Monthly price                | TBD                             |
| Annual plan                  | TBD                             |
| Annual discount              | TBD                             |
| GST/tax handling             | TBD                             |
| PAST_DUE grace period        | TBD                             |
| Cancellation policy          | Recommended: end of paid period |
| Refund policy                | TBD                             |
| Invoice requirements         | TBD                             |
| Payment provider             | Razorpay / Cashfree             |
| Trial reminder schedule      | TBD                             |
| Payment failure retry policy | TBD                             |

These values should be centralized and documented before payment implementation.

---

# 75. Recommended MVP Commercial Model

Unless business validation indicates otherwise, the simplest initial model is:

```text
3-Day Free Trial
       ↓
One Paid Plan
       ↓
Monthly Subscription
       ↓
Optional Annual Plan Later
```

This minimizes pricing complexity while the product is being validated.

---

# 76. Final Business Flow

```text
User Registers
      ↓
OTP Verification
      ↓
Creates First Event
      ↓
3-Day Trial Starts
      ↓
Uses Event Planner
      ↓
Trial Reminder
      ↓
Trial Expires
      ↓
Subscription Required
      ↓
Select Plan
      ↓
Checkout
      ↓
Payment Verification
      ↓
Subscription ACTIVE
      ↓
Continued Access
      ↓
Renewal
      ↓
ACTIVE
```

---

# 77. Related Documentation

This document should remain aligned with:

```text
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/04-architecture/architecture.md
docs/05-database/database-design.md
docs/06-api/api-specification.md

docs/08-security/security.md
docs/09-infrastructure/infrastructure.md
```

The subscription rules defined here are the business source of truth for the subscription implementation.
