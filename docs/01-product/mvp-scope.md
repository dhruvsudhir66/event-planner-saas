# MVP Scope

## 1. Purpose

This document defines the scope of the Minimum Viable Product (MVP) for the Event Planner application.

The purpose of the MVP is to validate the core product concept:

> **Can an event organizer manage the essential parts of planning an event from one simple application instead of relying on multiple disconnected tools?**

The MVP should provide enough functionality for a user to:

1. Create an account.
2. Create an event.
3. Manage event tasks.
4. Manage vendors.
5. Manage guests.
6. Save event ideas and inspiration.
7. Monitor event planning progress.
8. Use the application during a three-day free trial.
9. Subscribe to continue using the application after the trial.

The MVP should prioritize **simplicity, usability, reliability, and a complete end-to-end user journey** over the number of features.

---

# 2. Product Objective

The initial product objective is to create a centralized event planning workspace.

Instead of using:

```text
WhatsApp
+
Excel / Google Sheets
+
Phone Contacts
+
Notes
+
Instagram / Pinterest
+
Paper
```

the user should be able to manage the majority of their planning information in:

```text
                 EVENT PLANNER
                      │
       ┌──────────────┼──────────────┐
       │              │              │
     Tasks         Vendors         Guests
       │              │              │
       └──────────────┼──────────────┘
                      │
                    Ideas
                      │
                  Dashboard
```

---

# 3. MVP Target Users

The MVP targets individual event organizers.

### Primary users

- Couples planning weddings
- Individuals planning weddings
- People organizing engagements
- People organizing birthdays
- People organizing anniversaries
- People organizing baby showers
- People organizing private events

### Initial geographic market

India.

The architecture should not prevent future international expansion.

---

# 4. MVP Event Types

The initial release shall support:

- Wedding
- Engagement
- Birthday
- Anniversary
- Baby Shower
- Other

The event-type system should be configurable so additional event types can be added without major architectural changes.

---

# 5. MVP Core Features

The MVP consists of the following functional areas:

```text
Authentication
     ↓
Onboarding
     ↓
Event Management
     ↓
Dashboard
     ↓
Tasks
     ↓
Vendors
     ↓
Guests
     ↓
Ideas
     ↓
Subscription
     ↓
Settings
```

---

# 6. Authentication

## Included

The MVP shall include:

- Mobile number registration
- Mobile number login
- OTP verification
- OTP resend
- Change mobile number before verification
- Session management
- Logout
- Protected routes
- Backend authorization

### Authentication method

The initial authentication mechanism shall be:

> **Mobile number + OTP**

### SMS provider

The initial SMS provider is planned to be:

> **Fast2SMS**

The provider should be abstracted behind an internal OTP service so it can be replaced later without changing the application's authentication architecture.

---

# 7. User Onboarding

## Included

New users shall be guided through a simple onboarding process.

### Flow

```text
Register
   ↓
Verify OTP
   ↓
Account Created
   ↓
3-Day Trial Started
   ↓
Create First Event
   ↓
Dashboard
```

### Event creation information

Required:

- Event name
- Event type
- Event date

Optional:

- Location
- Event image

### UX objective

The user should reach a functional event dashboard as quickly as possible.

The onboarding process should not contain unnecessary questions.

---

# 8. Event Management

## Included

Users shall be able to:

- Create an event
- View an event
- Edit an event
- Delete an event

### Event information

```text
Event Name
Event Type
Event Date
Location
Event Image
Created Date
Updated Date
```

### MVP event model

The initial MVP may support one primary event workspace per user.

However, the database and API architecture should be designed so multiple events can be supported later.

---

# 9. Dashboard

The dashboard is the central screen of the application.

## Included

The dashboard shall display:

### Event information

- Event name
- Event date
- Countdown
- Event type
- Location

### Planning information

- Planning progress
- Pending tasks
- Upcoming tasks
- Overdue tasks

### Vendor information

- Number of vendors
- Payment summary

### Guest information

- Total guests
- RSVP summary

### Subscription information

- Trial status
- Trial days remaining
- Subscription status

### Quick actions

- Add task
- Add vendor
- Add guest
- Add idea

---

# 10. Task Management

## Included

Users shall be able to:

- Create tasks
- View tasks
- Edit tasks
- Complete tasks
- Reopen tasks
- Delete tasks
- Assign priority
- Assign category
- Set due dates

### Task fields

```text
Title
Description
Due Date
Priority
Category
Status
Created At
Updated At
```

### Priority

```text
Low
Medium
High
```

### Status

```text
Pending
Completed
```

### Initial categories

```text
Venue
Catering
Photography
Decoration
Invitations
Clothing
Beauty
Travel
Other
```

---

# 11. Vendor Management

## Included

Users shall be able to:

- Add vendors
- View vendors
- View vendor details
- Edit vendors
- Delete vendors
- Track vendor payments

### Vendor information

```text
Vendor Name
Category
Phone
Email
Estimated Cost
Actual Cost
Amount Paid
Payment Status
Notes
```

### Payment status

```text
Pending
Partially Paid
Paid
```

### Vendor summary

The system shall calculate:

```text
Total Estimated Cost
Total Actual Cost
Total Amount Paid
Total Amount Pending
```

---

# 12. Guest Management

## Included

Users shall be able to:

- Add guests
- View guests
- Edit guests
- Delete guests
- Track RSVP status
- Organize guests into groups

### Guest information

```text
Name
Phone
Group / Family
RSVP Status
Number of Attendees
Meal Preference
Notes
```

### RSVP status

```text
Pending
Attending
Not Attending
Maybe
```

### Guest summary

The system shall calculate:

```text
Total Guests
Expected Attendees
Attending
Pending
Not Attending
```

---

# 13. Ideas

The Ideas section is an important part of the MVP because event planning frequently involves collecting inspiration from different sources.

## Included

Users shall be able to:

- Create an idea
- Save text
- Save an image
- Save an external link
- Categorize an idea
- Edit an idea
- Delete an idea

### Initial categories

```text
Decor
Outfit
Photography
Invitation
Venue
Food
Makeup
Other
```

### Example use cases

A user can save:

```text
"Use white and sage flowers for the entrance."

Pinterest URL

Instagram URL

Screenshot of wedding decor

Wedding hairstyle image

Invitation design

Venue inspiration
```

### Product principle

The Ideas section should prioritize **fast capture over complex organization**.

---

# 14. File Uploads

## Included

The MVP shall support basic image uploads for:

- Event image
- Idea images

### Requirements

- File type validation
- File size validation
- Secure storage
- Event/resource association
- Authorization checks

### Storage

Images shall not be stored directly inside PostgreSQL.

An object-storage solution shall be used.

---

# 15. Subscription & Free Trial

## Included

The MVP shall implement a paid SaaS model.

### Trial

Every newly registered user receives:

> **3-day free trial**

### Trial lifecycle

```text
Account Created
      ↓
Trial Started
      ↓
Day 1
      ↓
Day 2
      ↓
Day 3
      ↓
Trial Expired
      ↓
Subscription Required
```

### Subscription states

The initial system shall support:

```text
TRIAL
ACTIVE
PAST_DUE
CANCELLED
EXPIRED
```

### Subscription flow

```text
Trial
  ↓
Trial Expired
  ↓
Subscription Screen
  ↓
Select Plan
  ↓
Payment
  ↓
Payment Verification
  ↓
Subscription Activated
  ↓
Application Access
```

### Payment provider

The initial payment provider shall be evaluated between:

- Razorpay
- Cashfree

The final provider will be documented in the infrastructure/business documentation before implementation.

---

# 16. Subscription Access Control

Subscription enforcement shall happen on the backend.

The frontend shall not determine whether a user is allowed to access the application.

### Access model

```text
                 User Request
                      │
                      ▼
                Authentication
                      │
                      ▼
                 Authorization
                      │
                      ▼
             Subscription Check
                      │
          ┌───────────┴───────────┐
          │                       │
       Active                  Inactive
          │                       │
          ▼                       ▼
      Allow Access          Subscription Page
```

The server shall be the source of truth for:

- Trial status
- Trial expiration
- Subscription status
- Payment status

---

# 17. Settings

## Included

The MVP shall provide:

- View profile
- Update supported profile information
- Notification preferences
- Subscription information
- Logout
- Delete account

---

# 18. Search and Filtering

Basic filtering shall be included where it significantly improves usability.

### Tasks

Filter by:

- Status
- Priority
- Category

### Guests

Filter by:

- RSVP status
- Group/family

### Vendors

Filter by:

- Category
- Payment status

Advanced global search is not required for the first MVP.

---

# 19. Notifications

The MVP will keep notifications intentionally limited.

## Included

The architecture should support future notifications.

The initial release may include:

- Trial expiry reminder
- Basic task reminder

### Not required for initial MVP

- Push notification infrastructure
- WhatsApp notifications
- Complex notification campaigns
- Marketing automation
- Email campaigns

---

# 20. Error and Empty States

All major MVP screens must include:

- Loading state
- Empty state
- Error state
- Success feedback where appropriate

Examples:

```text
No tasks yet.
Add your first task to start planning.
```

```text
No vendors added.
Keep all your vendor details in one place.
```

```text
Something went wrong.
Please try again.
```

The application must not expose raw backend errors or stack traces.

---

# 21. Security Scope

Security is part of the MVP and is not considered a future feature.

The MVP shall include:

- Secure OTP handling
- OTP expiration
- OTP rate limiting
- Authentication
- Backend authorization
- Event-level authorization
- Secure API endpoints
- Input validation
- File validation
- Secure file access
- Secrets management
- HTTPS
- Payment webhook verification
- Protection against unauthorized resource access

---

# 22. Data Model Scope

The initial database should support at minimum:

```text
users
events
tasks
vendors
guests
ideas
idea_attachments
subscriptions
payments
```

Additional supporting tables may be introduced where required for:

- OTP handling
- Sessions
- Notifications
- Audit records
- File metadata

The exact database schema will be defined in:

```text
docs/05-database/database-design.md
```

---

# 23. API Scope

The MVP backend shall provide APIs for:

```text
Authentication
Events
Dashboard
Tasks
Vendors
Guests
Ideas
Files
Subscriptions
Settings
```

The API shall follow a versioned structure:

```text
/api/v1/
```

Example:

```text
POST /api/v1/auth/request-otp
POST /api/v1/auth/verify-otp

GET  /api/v1/events
POST /api/v1/events

GET  /api/v1/events/{event_id}/tasks
POST /api/v1/events/{event_id}/tasks

GET  /api/v1/events/{event_id}/vendors
POST /api/v1/events/{event_id}/vendors

GET  /api/v1/events/{event_id}/guests
POST /api/v1/events/{event_id}/guests

GET  /api/v1/events/{event_id}/ideas
POST /api/v1/events/{event_id}/ideas
```

The complete API contract will be defined separately.

---

# 24. Frontend Scope

The frontend shall be built using:

- Next.js
- TypeScript
- Tailwind CSS

### Initial screens

```text
SCR-001 Landing
SCR-002 Login
SCR-003 OTP Verification
SCR-004 Create Event
SCR-005 Dashboard
SCR-006 Tasks
SCR-007 Vendors
SCR-008 Guests
SCR-009 Ideas
SCR-010 Subscription
SCR-011 Settings
```

The frontend shall provide responsive layouts with **mobile-first design**.

---

# 25. Backend Scope

The backend shall be built using:

- Python
- FastAPI
- PostgreSQL
- SQLAlchemy
- Alembic

The backend shall be responsible for:

- Authentication
- Authorization
- Business logic
- Data persistence
- Subscription enforcement
- Payment verification
- OTP integration
- File access control
- API validation

---

# 26. MVP Infrastructure

The initial infrastructure target is:

```text
Frontend
Next.js
     ↓
Vercel

Backend
FastAPI
     ↓
Render

Database
PostgreSQL
     ↓
Supabase

SMS
Fast2SMS

Payments
Razorpay / Cashfree

File Storage
Object Storage
```

The final providers shall be documented and confirmed before production deployment.

---

# 27. Environments

The MVP should have at least:

```text
Development
     ↓
Staging
     ↓
Production
```

### Development

Used for local development.

### Staging

Used for:

- Integration testing
- Payment testing
- SMS testing
- Pre-production validation

### Production

Used by actual customers.

Production credentials and data must never be used for local development.

---

# 28. MVP Out of Scope

The following features shall **not** be implemented in the initial MVP.

## Collaboration

- Multiple event members
- Couple/shared accounts
- Roles and permissions
- Comments
- Activity feed

## Guest Communication

- Digital invitations
- Guest RSVP portal
- Email invitations
- SMS invitations
- WhatsApp invitations
- Automated guest reminders

## Vendor Marketplace

- Vendor discovery
- Vendor recommendations
- Vendor reviews
- Vendor ratings
- Vendor booking
- Vendor marketplace

## Advanced Finance

- Full budget management
- Expense tracking
- Expense receipts
- Financial reports
- Payment scheduling
- Accounting integration

## Calendar

- Google Calendar integration
- Outlook integration
- Calendar synchronization
- Advanced event timeline

## AI

- AI planning assistant
- AI-generated checklists
- AI recommendations
- AI budget planning
- AI vendor recommendations
- AI image analysis

## Advanced Content

- Document management
- Contract management
- PDF generation
- Invitation generation
- Seating charts

## Analytics

- Advanced analytics dashboard
- Event performance analytics
- User behavior analytics
- Financial analytics

These features may be considered for future releases based on user feedback.

---

# 29. MVP Non-Goals

The MVP is **not** intended to:

- Replace professional event planners.
- Provide a vendor marketplace.
- Handle all event communication.
- Become a complete accounting application.
- Become a social network for events.
- Provide advanced AI functionality.
- Support enterprise event management.

The goal is to provide a **simple personal planning workspace**.

---

# 30. MVP Success Criteria

The MVP should be considered successful if a new user can complete the following journey without assistance:

```text
Landing
   ↓
Register
   ↓
Verify Mobile
   ↓
Create Event
   ↓
Reach Dashboard
   ↓
Create Task
   ↓
Add Vendor
   ↓
Add Guest
   ↓
Save Idea
   ↓
View Planning Progress
   ↓
Use Application During Trial
   ↓
Subscribe
   ↓
Continue Using Application
```

### Usability goal

A first-time user should be able to understand the application's basic functionality without requiring documentation or a tutorial.

---

# 31. MVP Quality Requirements

The MVP shall not be considered complete merely because all screens have been implemented.

The following must also be functional:

### UX

- Responsive design
- Mobile-first layouts
- Clear navigation
- Loading states
- Empty states
- Error states
- Confirmation dialogs for destructive actions

### Backend

- Input validation
- Authentication
- Authorization
- Error handling
- Database transactions where required
- Secure API design

### Security

- OTP protection
- Rate limiting
- Secure sessions
- Resource-level authorization
- Secure file access
- Payment webhook verification

### Reliability

- Database backups
- Error logging
- Health checks
- Production monitoring

---

# 32. MVP Release Criteria

The MVP may be released to users only when:

- All P0 functionality is implemented.
- Critical user journeys work end-to-end.
- Authentication has been tested.
- OTP delivery has been tested.
- Database migrations are stable.
- Subscription flow has been tested.
- Payment webhooks have been tested.
- Trial expiration has been tested.
- Authorization has been tested.
- File uploads have been tested.
- Mobile responsiveness has been verified.
- Production environment is configured.
- Secrets are not exposed.
- Error monitoring is enabled.
- Backup strategy is established.
- Privacy policy is available.
- Terms and conditions are available.

---

# 33. MVP Development Priority

Development should follow this order:

```text
1. Project Foundation
        ↓
2. Authentication
        ↓
3. Event Creation
        ↓
4. Dashboard
        ↓
5. Tasks
        ↓
6. Vendors
        ↓
7. Guests
        ↓
8. Ideas
        ↓
9. Subscription
        ↓
10. Settings
        ↓
11. Security Hardening
        ↓
12. Testing
        ↓
13. Staging Deployment
        ↓
14. Production Deployment
```

---

# 34. MVP Definition of Done

The MVP is considered complete when:

1. A new user can register using mobile OTP.
2. A user can create an event.
3. A user can access the event dashboard.
4. A user can create and manage tasks.
5. A user can create and manage vendors.
6. A user can create and manage guests.
7. A user can save and manage ideas.
8. The dashboard provides meaningful planning information.
9. A three-day trial is correctly enforced.
10. A user can subscribe after the trial.
11. Payment status is verified by the backend.
12. Unauthorized users cannot access protected data.
13. The application works on mobile and desktop.
14. Errors and empty states are handled properly.
15. Production monitoring and backups are configured.
16. Required legal pages are available.
17. Critical end-to-end tests pass.

---

# 35. Scope Change Policy

Any feature not explicitly included in this document shall be considered **outside the MVP scope**.

A feature may be added to the MVP only after evaluating:

- User value
- Development effort
- Security impact
- Infrastructure impact
- Product complexity
- Launch timeline

Adding a feature to the MVP must not compromise the core user journey.

---

# 36. Future Product Direction

After validating the MVP, the product may evolve toward a more comprehensive event-planning platform.

Potential future capabilities include:

```text
MVP
 │
 ├── Collaboration
 │
 ├── Budget Management
 │
 ├── Calendar
 │
 ├── Invitations
 │
 ├── Guest Communication
 │
 ├── Vendor Marketplace
 │
 ├── AI Planning
 │
 └── Event Automation
```

These capabilities should be driven by actual user behavior and feedback rather than being assumed as MVP requirements.

---

# 37. Related Documentation

This document should be used together with:

- `docs/01-product/product-definition.md`
- `docs/02-requirements/functional-requirements.md`
- `docs/02-requirements/user-stories.md`
- `docs/03-user-experience/user-flows.md`
- `docs/03-user-experience/screen-inventory.md`
- `docs/04-architecture/architecture.md`
- `docs/05-database/database-design.md`
- `docs/06-api/api-specification.md`
- `docs/07-business/subscription.md`
- `docs/08-security/security.md`
- `docs/09-infrastructure/infrastructure.md`
- `docs/10-development/roadmap.md`
