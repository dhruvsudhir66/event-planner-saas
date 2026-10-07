# Pricing Strategy and Plan Definition

**Document:** `docs/07-business/pricing.md`  
**Status:** Draft  
**Version:** 1.0  
**Application:** Event Planner  
**Market:** India  
**Business Model:** Free Trial + Paid Subscription

---

# 1. Purpose

This document defines the pricing strategy for Event Planner.

It establishes:

- Pricing principles
- Target customer
- Plan structure
- Trial strategy
- Feature packaging
- Billing intervals
- Pricing configuration
- Discounts
- Promotional pricing
- Price changes
- Upgrade/downgrade rules
- Refund considerations
- Pricing experimentation
- Commercial metrics

This document is separate from:

```text
docs/07-business/subscription.md
```

`subscription.md` defines how subscriptions technically operate.

`pricing.md` defines what customers are charged and what each plan includes.

---

# 2. Product Positioning

Event Planner is positioned as a centralized event-planning workspace.

The core value proposition is:

> **Plan your entire event in one place.**

The product replaces fragmented planning across:

- WhatsApp
- Excel
- Google Sheets
- Notes
- Phone contacts
- Pinterest
- Instagram
- Paper lists

The customer is paying for organization, convenience and reduced planning friction.

---

# 3. Target Customer

The initial customer is an individual planning an event.

Primary segment:

- Couples planning weddings
- Individuals planning engagements
- Birthday organizers
- Anniversary organizers
- Baby shower organizers
- Private event organizers

The initial market is India.

---

# 4. Pricing Principles

Pricing should follow these principles:

### 4.1 Simple

The customer should understand the price within seconds.

### 4.2 Transparent

The price displayed before checkout should match the amount used to create the payment.

### 4.3 Value-Based

Pricing should reflect the value of managing an entire event rather than the number of individual features.

### 4.4 Low Friction

The initial purchase should not require a complicated plan comparison.

### 4.5 Scalable

The pricing architecture must support additional plans later.

### 4.6 India-First

Initial pricing should be appropriate for Indian consumers while allowing future international pricing.

---

# 5. Initial Pricing Model

The recommended MVP model is:

```text
3-Day Free Trial
        ↓
One Paid Plan
        ↓
Monthly Subscription
```

An annual plan can be introduced after sufficient customer usage data is available.

---

# 6. Why One Paid Plan

The MVP should avoid excessive plan complexity.

Multiple plans introduce:

- Feature comparison complexity
- Pricing confusion
- More billing logic
- More support cases
- More upgrade/downgrade scenarios
- More testing requirements

The initial objective is product validation, not maximizing pricing complexity.

---

# 7. Recommended Initial Plan

Internal plan code:

```text
EVENT_PLANNER_PRO
```

Customer-facing name:

```text
Event Planner
```

or a final branded name selected before launch.

The internal `plan_code` must remain stable even if the display name changes.

---

# 8. Recommended MVP Price

The initial pricing recommendation is:

| Plan          | Billing | Recommended Price |
| ------------- | ------- | ----------------: |
| Free Trial    | 3 days  |                ₹0 |
| Event Planner | Monthly |        ₹299/month |

The final price should be validated against:

- Customer willingness to pay
- Competitor positioning
- Conversion rate
- Support costs
- Payment fees
- Retention
- Event duration

₹299/month is a starting hypothesis, not a permanent price.

---

# 9. Annual Plan

The annual plan should initially remain optional.

A potential future structure:

| Plan          | Monthly Equivalent | Annual Price |
| ------------- | -----------------: | -----------: |
| Event Planner |         ₹299/month |          TBD |

A reasonable future annual discount could be approximately:

```text
15%–25%
```

The exact amount should be determined after observing monthly conversion and retention.

---

# 10. Event Duration Consideration

Event planning is often temporary rather than permanent.

This is important for pricing.

A customer may need the application for:

```text
3–12 months
```

rather than several years.

Therefore, pricing should eventually consider a **planning-season model** in addition to conventional SaaS billing.

Possible future options:

```text
Monthly
Annual
Event Pass
```

The MVP should not implement all three simultaneously.

---

# 11. Future Event Pass

A future product could offer:

```text
Wedding / Event Pass
```

for customers who do not want recurring billing.

Example conceptual model:

```text
One Event
+
Fixed Planning Period
+
Full Access
=
Event Pass
```

This is intentionally out of the MVP until customer demand is validated.

---

# 12. Free Trial

The free trial is:

```text
3 days
```

The trial provides access to the full MVP experience.

The goal is to allow users to experience the product rather than artificially restricting functionality.

---

# 13. Trial Philosophy

The trial should allow the customer to reach an "aha moment".

The user should be able to:

```text
Create Event
    ↓
Add Tasks
    ↓
Add Vendors
    ↓
Add Guests
    ↓
Save Ideas
    ↓
See Dashboard
```

The product should demonstrate its value before asking for payment.

---

# 14. Trial Activation

The trial begins after the user creates their first event.

```text
Register
   ↓
Verify OTP
   ↓
Create Event
   ↓
Trial Begins
```

This avoids starting the trial before the user has actually experienced the product.

---

# 15. Trial Eligibility

A user may receive only one trial.

Eligibility belongs to the account:

```text
User
 ↓
Trial Used
```

not:

```text
Event
 ↓
Trial Used
```

Creating another event must not reset the trial.

---

# 16. Feature Packaging

The MVP paid plan should include the core planning functionality.

### Event Management

- Event creation
- Event details
- Event image

### Dashboard

- Countdown
- Planning progress
- Upcoming tasks
- Overdue tasks
- Vendor summary
- Guest summary
- Subscription status

### Tasks

- Create tasks
- Edit tasks
- Complete tasks
- Delete tasks
- Priorities
- Categories
- Due dates

### Vendors

- Vendor records
- Vendor contact information
- Estimated costs
- Actual costs
- Payments
- Payment status

### Guests

- Guest records
- RSVP status
- Attendee count
- Meal preference
- Groups

### Ideas

- Idea collection
- Images
- External links
- Categories

---

# 17. No Artificial Feature Gating

The MVP should not intentionally split basic functionality into several plans simply to force upgrades.

For example, avoid:

```text
Free:
10 guests

Paid:
Unlimited guests
```

during the initial product validation period unless infrastructure costs make such a limit necessary.

The primary monetization mechanism is the subscription itself.

---

# 18. Pricing Page

The pricing page should communicate:

1. What the product does
2. Trial duration
3. Paid price
4. Billing frequency
5. Included functionality
6. Cancellation policy
7. Primary CTA

Avoid excessive marketing language.

---

# 19. Pricing Page Structure

Recommended:

```text
Pricing

Plan your entire event in one place.

3-day free trial

₹299 / month

Everything you need to organize your event.

✓ Event planning
✓ Tasks
✓ Vendors
✓ Guest management
✓ Ideas
✓ Dashboard

[Start Free Trial]
```

The exact marketing copy is subject to UX review.

---

# 20. Pricing Transparency

Before checkout, the customer should know:

```text
Plan
Billing interval
Price
Currency
Applicable taxes
Renewal behavior
Cancellation terms
```

There should be no unexpected mandatory charge.

---

# 21. Price Configuration

Prices must not be hardcoded throughout the frontend.

Recommended backend representation:

```text
plan_code
name
billing_interval
price
currency
is_active
```

Example:

```json
{
  "plan_code": "EVENT_PLANNER_PRO",
  "name": "Event Planner",
  "billing_interval": "MONTHLY",
  "price": 299.0,
  "currency": "INR",
  "is_active": true
}
```

---

# 22. Price Source of Truth

The backend is the authoritative source for pricing.

Correct:

```text
Frontend
   ↓
GET /subscription/plans
   ↓
Backend
   ↓
Current price
```

Incorrect:

```text
Frontend
   ↓
Hardcoded ₹299
   ↓
Payment
```

---

# 23. Payment Amount Validation

The client should send:

```json
{
  "plan_code": "EVENT_PLANNER_PRO",
  "billing_interval": "MONTHLY"
}
```

The backend determines:

```text
₹299
```

The client must not send:

```json
{
  "amount": 299
}
```

as the authoritative payment amount.

---

# 24. Payment Provider Product IDs

Payment-provider identifiers should be stored separately from internal plan identifiers.

Example:

```text
Internal:
EVENT_PLANNER_PRO

Provider:
provider-specific-product-id
```

This allows the payment provider to change without changing product/business logic.

---

# 25. Pricing Configuration Model

A future pricing configuration may look like:

```text
PricingPlan
├── id
├── plan_code
├── display_name
├── description
├── billing_interval
├── amount
├── currency
├── provider_product_id
├── provider_price_id
├── is_active
├── created_at
└── updated_at
```

This can be introduced as a database table when multiple plans are required.

---

# 26. Pricing Versioning

Prices should be treated as versioned commercial data.

Example:

```text
EVENT_PLANNER_PRO
₹299
Version 1
```

Later:

```text
EVENT_PLANNER_PRO
₹349
Version 2
```

Historical payments must continue to reference the amount actually charged.

---

# 27. Existing Subscribers and Price Changes

When pricing changes, existing subscribers should not automatically be charged a different amount without an explicit migration strategy.

Possible policies:

### Grandfathering

Existing users retain their current price.

### Immediate Migration

Existing users move to the new price.

### Renewal Migration

Existing users move to the new price at their next renewal.

The recommended default is:

```text
Renewal Migration
```

with advance communication.

---

# 28. Price Increase Policy

Before a significant price increase:

1. Identify affected users.
2. Determine effective date.
3. Communicate the change.
4. Update billing configuration.
5. Ensure existing subscriptions are handled correctly.
6. Monitor cancellations.

The exact legal/notice requirements should be confirmed before implementation.

---

# 29. Promotional Pricing

Promotional pricing may be introduced later.

Possible promotions:

```text
First month discount
Annual discount
Launch offer
Referral discount
Coupon
```

Promotions should be implemented through a centralized pricing/discount system rather than hardcoded frontend logic.

---

# 30. Coupon System

Coupons are not required for MVP.

Future structure:

```text
Coupon
├── code
├── discount_type
├── discount_value
├── valid_from
├── valid_until
├── usage_limit
├── per_user_limit
└── is_active
```

---

# 31. Launch Pricing

A temporary launch price may be used.

Example:

```text
Standard Price: ₹349
Launch Price: ₹299
```

If used, the promotion must have clearly defined:

- Start date
- End date
- Eligibility
- Renewal behavior

---

# 32. Discounts

Discounts should never create ambiguity about the renewal price.

For example:

```text
First month: ₹199
Then: ₹299/month
```

must clearly communicate:

```text
₹199 for the first billing period.
₹299/month thereafter.
```

---

# 33. Cancellation

The pricing policy should align with the subscription policy.

Recommended:

```text
Cancel subscription
       ↓
No future renewal
       ↓
Access continues until paid period ends
```

The customer should clearly see the cancellation effective date.

---

# 34. Refund Policy

The refund policy is TBD before production.

The business must define:

- Whether refunds are offered
- Refund window
- Partial refunds
- Duplicate payment handling
- Failed payment handling
- Cancellation after payment

The final policy must be reflected in the Terms and subscription UI.

---

# 35. Failed Payments

Failed recurring payments should not immediately result in destructive actions.

Recommended:

```text
Payment fails
     ↓
PAST_DUE
     ↓
Recovery period
     ↓
Payment recovered → ACTIVE
     ↓
Payment not recovered → EXPIRED
```

The exact recovery period is TBD.

---

# 36. Billing Interval

MVP:

```text
MONTHLY
```

Future:

```text
YEARLY
```

Potential later:

```text
EVENT_PASS
```

---

# 37. Monthly Pricing Advantages

Monthly billing provides:

- Lower initial commitment
- Easier customer adoption
- Simple MVP implementation
- Faster pricing experimentation
- Easier cancellation

---

# 38. Annual Pricing Advantages

Annual pricing can provide:

- Higher upfront revenue
- Lower churn
- Better retention
- Lower payment frequency
- Better economics for longer planning periods

Annual billing should be introduced once there is evidence of users needing the product for longer periods.

---

# 39. Pricing Experimentation

Pricing should be treated as an experiment.

Potential variables:

```text
₹199
₹249
₹299
₹349
₹399
```

Testing should not be performed recklessly across existing subscribers.

Experiments should measure:

- Trial activation
- Trial completion
- Trial-to-paid conversion
- Revenue per visitor
- Revenue per trial
- Churn
- Refunds
- Support requests

---

# 40. Primary Pricing Metric

The most important early metric is:

```text
Trial-to-Paid Conversion Rate
```

Formula:

```text
Paid Users
─────────────── × 100
Eligible Trial Users
```

---

# 41. Revenue Per Trial

Another useful metric:

```text
Revenue Per Trial
=
Subscription Revenue
÷
Number of Trial Users
```

This combines conversion and pricing into one business metric.

---

# 42. Customer Acquisition Cost

Once paid acquisition begins:

```text
CAC =
Total Acquisition Spend
÷
New Paying Customers
```

CAC should eventually be compared with customer lifetime value.

---

# 43. Customer Lifetime Value

A simplified early model:

```text
LTV
=
Average Monthly Revenue
×
Average Customer Lifetime
```

A more complete model should account for:

- Payment fees
- Discounts
- Refunds
- Support
- Acquisition cost

---

# 44. Churn

Monthly customer churn:

```text
Customers Lost During Period
───────────────────────────── × 100
Customers at Start of Period
```

High churn may indicate that customers only need the product temporarily.

This is especially relevant for event-planning software.

---

# 45. Event-Based Product Economics

Unlike traditional productivity SaaS, Event Planner may have naturally limited customer lifetimes.

A wedding customer may use the product for:

```text
3–12 months
```

and then leave because the event is complete.

Therefore:

```text
Churn ≠ necessarily product failure
```

The business must distinguish:

```text
Event completed
```

from:

```text
Customer dissatisfied
```

---

# 46. Event Completion

A future product feature may allow users to mark:

```text
Event Completed
```

This can provide valuable business analytics.

For example:

```text
Subscription ended because event completed
```

is different from:

```text
Subscription cancelled because product was unsatisfactory
```

This distinction is not required for MVP.

---

# 47. Recommended Customer Lifecycle

```text
Visitor
   ↓
Trial User
   ↓
Active Subscriber
   ↓
Event Planning
   ↓
Event Completed
   ↓
Subscription Ends
```

Potential future lifecycle:

```text
Completed Event
   ↓
New Event
   ↓
Reactivation
```

This creates an opportunity for customers who plan multiple events.

---

# 48. Multi-Event Pricing

The MVP subscription should allow the account to manage multiple events according to the plan's limits.

The pricing model should not assume:

```text
One subscription = one event
```

unless the commercial strategy explicitly changes.

The current architecture treats subscription ownership at the user level.

---

# 49. Future Plan Expansion

Future plans could potentially be:

```text
Free Trial
     ↓
Essential
     ↓
Pro
     ↓
Business
```

However, these should only be introduced when there are meaningful differences in customer needs.

---

# 50. Potential Future Plans

Example:

| Plan      | Target                         |
| --------- | ------------------------------ |
| Essential | Individuals with simple events |
| Pro       | Larger/complex events          |
| Business  | Professional event planners    |

The Business plan is not part of the MVP.

---

# 51. Professional Event Planner Opportunity

A future B2B segment could include:

- Wedding planners
- Event coordinators
- Decorators
- Event agencies

Their requirements may justify:

- Multiple events
- Team members
- Client management
- Collaboration
- Advanced reporting
- Branding
- Vendor management

These features are outside MVP scope.

---

# 52. Pricing and Product Scope

Pricing must not drive premature feature development.

Do not introduce features solely to justify a higher plan before customers actually need them.

The product should first establish:

```text
Problem
   ↓
Usage
   ↓
Value
   ↓
Willingness to pay
```

---

# 53. Pricing Analytics

The application should eventually track:

### Acquisition

- Landing-page visitors
- Trial registrations
- Trial activation

### Conversion

- Checkout initiation
- Successful payment
- Trial-to-paid conversion

### Revenue

- MRR
- ARR
- ARPU

### Retention

- Churn
- Renewal
- Reactivation

### Billing

- Payment success
- Payment failure
- Refunds

---

# 54. Pricing Dashboard

An internal business dashboard may eventually display:

```text
Active Subscribers
Trial Users
Trial Conversion
MRR
ARR
Churn
Payment Success Rate
Refund Rate
```

This is an internal/admin capability and is not part of the MVP user dashboard.

---

# 55. Pricing Data Privacy

Business analytics must not expose unnecessary personal information.

Analytics should preferably use:

- Aggregated counts
- Anonymous identifiers where appropriate
- Internal user IDs rather than phone numbers

Payment credentials and sensitive billing information must never be included in analytics events.

---

# 56. Pricing Configuration Rules

Pricing configuration must satisfy:

1. No hardcoded prices across UI components.
2. Backend is authoritative.
3. Historical payment amounts are immutable.
4. Provider price IDs are separate from internal plan IDs.
5. Inactive plans cannot be newly purchased.
6. Existing subscriptions remain traceable.
7. Currency is explicit.
8. Billing interval is explicit.
9. Discounts are explicit.
10. Promotional periods have start and end dates.

---

# 57. Pricing Acceptance Criteria

Pricing functionality is considered complete when:

- [ ] MVP price is configured centrally
- [ ] Frontend retrieves plan information from API
- [ ] Backend determines payment amount
- [ ] Currency is explicit
- [ ] Billing interval is explicit
- [ ] Provider price identifiers are separated from internal plan codes
- [ ] Trial pricing is correctly displayed
- [ ] Checkout displays correct pricing
- [ ] Renewal pricing is clear
- [ ] Payment history records actual charged amounts
- [ ] Cancellation terms are displayed
- [ ] Price changes can be introduced without frontend-wide changes
- [ ] Historical pricing remains auditable

---

# 58. Initial Pricing Decision

For MVP planning purposes:

```text
Product:
Event Planner

Trial:
3 days

Paid plan:
Event Planner

Billing:
Monthly

Initial price hypothesis:
₹299/month

Annual:
Not initially required

Coupons:
Not initially required

Multiple paid tiers:
Not initially required

Event Pass:
Future consideration
```

---

# 59. Pricing Decision Framework

Before changing the price, evaluate:

```text
Conversion
+
Retention
+
Customer feedback
+
Usage
+
Support cost
+
Payment cost
+
Acquisition cost
=
Pricing decision
```

Pricing should be changed based on evidence rather than intuition alone.

---

# 60. Final Pricing Strategy

The MVP should launch with **one clear paid plan at a simple monthly price**, following a three-day full-feature trial.

The commercial architecture should remain flexible enough to introduce:

- Annual billing
- Event passes
- Discounts
- Multiple plans
- Professional/event-planner plans

without rewriting the subscription system.

The initial goal is not to optimize every possible revenue stream.

The initial goal is to validate:

```text
Do users find enough value in Event Planner
to pay for continued access?
```

---

# 61. Related Documentation

```text
docs/01-product/product-definition.md
docs/01-product/mvp-scope.md

docs/02-requirements/functional-requirements.md
docs/02-requirements/user-stories.md

docs/04-architecture/architecture.md
docs/05-database/database-design.md
docs/06-api/api-specification.md

docs/07-business/subscription.md

docs/08-security/security.md
docs/09-infrastructure/infrastructure.md
```

**Source of truth:**

```text
pricing.md
    ↓
Defines commercial pricing

subscription.md
    ↓
Defines subscription lifecycle

database-design.md
    ↓
Defines persistence

api-specification.md
    ↓
Defines billing APIs

security.md
    ↓
Defines payment/security controls
```
