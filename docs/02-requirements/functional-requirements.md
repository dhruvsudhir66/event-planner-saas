# Functional Requirements

## 1. Purpose

This document defines the functional requirements for the initial MVP of the Event Planner application.

Each requirement describes a specific system behavior and is assigned a unique identifier that can be referenced during:

- UX design
- Frontend development
- Backend development
- API design
- Database design
- Testing
- Acceptance testing
- Future product planning

---

# 2. Requirement ID Convention

Requirements follow this format:

```text
FR-[MODULE]-[NUMBER]
```

Examples:

```text
FR-AUTH-001
FR-EVENT-001
FR-TASK-001
FR-VENDOR-001
FR-GUEST-001
FR-IDEA-001
FR-SUB-001
```

### Module Codes

| Code   | Module            |
| :----- | :---------------- |
| AUTH   | Authentication    |
| ONB    | Onboarding        |
| EVENT  | Event Management  |
| DASH   | Dashboard         |
| TASK   | Task Management   |
| VENDOR | Vendor Management |
| GUEST  | Guest Management  |
| IDEA   | Ideas             |
| SUB    | Subscription      |
| SET    | Settings          |
| NOTIF  | Notifications     |
| SYS    | System            |

---

# 3. Authentication Requirements

## FR-AUTH-001 — User Registration

The system shall allow a new user to register using a valid mobile number.

### Preconditions

- The user does not already have an account.
- The application is available.
- The SMS service is available.

### Flow

1. User enters their mobile number.
2. System validates the mobile number.
3. System checks whether the mobile number is already registered.
4. System generates an OTP.
5. System stores the OTP securely with an expiration time.
6. System sends the OTP through the configured SMS provider.
7. User enters the OTP.
8. System validates the OTP.
9. System creates the user account.
10. System creates the user's trial period.
11. System authenticates the user.
12. System redirects the user to event onboarding.

### Validation

- Mobile number is mandatory.
- Mobile number must follow the supported format.
- Mobile number must not contain invalid characters.
- Mobile number must not already belong to an existing account.
- OTP must expire after a configured period.
- Maximum OTP verification attempts must be enforced.
- OTP resend requests must be rate limited.

### Success

- User account is created.
- User is authenticated.
- Three-day trial is initialized.
- User is redirected to event onboarding.

### Failure

The system shall display an appropriate user-friendly error message.

---

## FR-AUTH-002 — Existing User Login

The system shall allow an existing user to authenticate using their registered mobile number and OTP.

### Preconditions

- User has an existing account.
- User's account is not permanently deleted or disabled.

### Flow

1. User enters their registered mobile number.
2. System validates the mobile number.
3. System generates an OTP.
4. System sends the OTP.
5. User enters the OTP.
6. System validates the OTP.
7. System creates or refreshes the user's authenticated session.
8. System checks account and subscription status.
9. System redirects the user to the appropriate destination.

### Success

The authenticated user is redirected to:

- Dashboard if an event exists and access is active.
- Event creation if no event exists.
- Subscription screen if access has expired.

### Failure

The system displays an appropriate authentication error.

---

## FR-AUTH-003 — OTP Verification

The system shall verify an OTP submitted by the user.

### Validation

- OTP must match the currently active OTP.
- OTP must not be expired.
- OTP verification attempts must not exceed the configured limit.
- OTP must only be usable once.

### Success

The OTP is marked as used and authentication continues.

### Failure

The system displays an appropriate error and allows the user to retry if attempts remain.

---

## FR-AUTH-004 — Resend OTP

The system shall allow users to request a new OTP.

### Rules

- Resend requests must be rate limited.
- A configurable cooldown period must exist between resend requests.
- A maximum number of resend requests must be enforced.
- Previous OTPs must be invalidated when appropriate.
- The newest valid OTP shall be used for verification.

### Failure

If the resend limit is exceeded, the system shall temporarily prevent additional requests.

---

## FR-AUTH-005 — Change Mobile Number

The system shall allow a user on the OTP screen to change the mobile number entered before verification.

### Flow

```text
OTP Screen
   ↓
Change Number
   ↓
Enter New Number
   ↓
Request OTP
   ↓
Verify New OTP
```

---

## FR-AUTH-006 — Session Management

The system shall maintain an authenticated session after successful login or registration.

### Requirements

- Authentication credentials must be securely stored.
- Session expiration must be configurable.
- Expired sessions must not provide access to protected resources.
- Refresh mechanisms must be securely implemented where applicable.
- Logout must invalidate the active session.

---

## FR-AUTH-007 — Logout

The system shall allow an authenticated user to log out.

### Flow

```text
Settings
   ↓
Logout
   ↓
Invalidate Session
   ↓
Return to Login / Landing
```

---

## FR-AUTH-008 — Protected Resource Access

The system shall prevent unauthenticated users from accessing protected application resources.

### Rule

Requests to protected APIs without valid authentication shall return an unauthorized response.

The frontend shall redirect unauthenticated users to the login flow.

---

# 4. Onboarding Requirements

## FR-ONB-001 — Create First Event

The system shall guide a newly registered user through creating their first event.

### Required information

- Event name
- Event type
- Event date

### Optional information

- Location
- Event image

### Flow

```text
Account Created
   ↓
Create Your Event
   ↓
Enter Event Information
   ↓
Create Event
   ↓
Dashboard
```

---

## FR-ONB-002 — Event Type Selection

The system shall allow users to select an event type.

### Initial event types

- Wedding
- Engagement
- Birthday
- Anniversary
- Baby Shower
- Other

The event type list should be configurable so that additional event types can be introduced later.

---

## FR-ONB-003 — Complete Onboarding

The system shall consider onboarding complete when the user has successfully created their first event.

The user shall subsequently be redirected to the event dashboard.

---

# 5. Event Management Requirements

## FR-EVENT-001 — Create Event

The system shall allow an authenticated user to create an event.

### Required fields

- Event name
- Event type
- Event date

### Optional fields

- Location
- Event image

### Validation

- Event name is required.
- Event type is required.
- Event date is required.
- Event date must be valid.
- Input lengths must be validated.
- User must be authenticated.

### Success

The event is created and associated with the authenticated user.

---

## FR-EVENT-002 — View Event

The system shall allow the user to view event details.

### Event information

- Event name
- Event type
- Event date
- Location
- Event image
- Created date

---

## FR-EVENT-003 — Edit Event

The system shall allow the user to update event information.

### Editable information

- Event name
- Event type
- Event date
- Location
- Event image

### Success

Updated information is persisted and reflected throughout the application.

---

## FR-EVENT-004 — Delete Event

The system shall allow an authorized user to delete an event.

### Flow

```text
Event Settings
   ↓
Delete Event
   ↓
Confirmation
   ↓
Confirm
   ↓
Delete Event
```

### Requirements

- Destructive action must require confirmation.
- The system must verify that the user has permission to delete the event.
- Associated data must follow the application's defined deletion policy.

---

## FR-EVENT-005 — Event Ownership

The system shall ensure that users can only access events they are authorized to access.

A user must not be able to retrieve, modify, or delete another user's event by manipulating an event ID.

---

# 6. Dashboard Requirements

## FR-DASH-001 — Display Event Overview

The dashboard shall display a summary of the user's active event.

### Information

- Event name
- Event date
- Countdown
- Event type
- Location where available

---

## FR-DASH-002 — Display Planning Progress

The system shall calculate and display event planning progress.

The initial calculation may be based on completed tasks versus total tasks.

Example:

```text
Completed Tasks / Total Tasks × 100
```

If no tasks exist, the dashboard should display an appropriate empty state rather than displaying misleading progress.

---

## FR-DASH-003 — Display Upcoming Tasks

The dashboard shall display relevant upcoming tasks.

The system should prioritize:

1. Overdue tasks
2. Tasks due today
3. Upcoming tasks

---

## FR-DASH-004 — Display Vendor Summary

The dashboard shall display a summary of the event's vendors.

Example information:

- Total vendors
- Vendors with pending payments
- Total vendor expenditure

---

## FR-DASH-005 — Display Guest Summary

The dashboard shall display a summary of the event's guests.

Example information:

- Total guests
- Confirmed guests
- Pending RSVPs
- Declined guests

---

## FR-DASH-006 — Quick Actions

The dashboard shall provide quick actions for frequently used operations.

Initial actions:

- Add task
- Add vendor
- Add guest
- Add idea

---

## FR-DASH-007 — Subscription Status

The dashboard shall display relevant trial or subscription information.

Examples:

```text
Trial — 2 days remaining
```

or:

```text
Subscription active
```

The subscription status shall be determined by the backend.

---

# 7. Task Management Requirements

## FR-TASK-001 — Create Task

The system shall allow users to create event-related tasks.

### Fields

- Title
- Description
- Due date
- Priority
- Category

### Validation

- Task title is required.
- User must have access to the event.
- Due date must be valid when provided.

---

## FR-TASK-002 — View Tasks

The system shall display tasks associated with the selected event.

Users shall be able to distinguish between:

- Pending tasks
- Completed tasks
- Overdue tasks

---

## FR-TASK-003 — Update Task

The system shall allow users to modify task information.

---

## FR-TASK-004 — Complete Task

The system shall allow users to mark a task as completed.

### Success

- Task status changes to completed.
- Planning progress is recalculated.
- Dashboard information is updated.

---

## FR-TASK-005 — Reopen Task

The system shall allow users to move a completed task back to an active state.

---

## FR-TASK-006 — Delete Task

The system shall allow users to delete a task after confirmation.

---

## FR-TASK-007 — Task Priority

The system shall support task priorities.

Initial priorities:

- Low
- Medium
- High

---

## FR-TASK-008 — Task Categories

The system shall support task categories.

Initial categories may include:

- Venue
- Catering
- Photography
- Decoration
- Invitations
- Clothing
- Beauty
- Travel
- Other

The category list should be configurable.

---

# 8. Vendor Management Requirements

## FR-VENDOR-001 — Add Vendor

The system shall allow users to add a vendor to an event.

### Fields

- Vendor name
- Category
- Phone
- Email
- Estimated cost
- Actual cost
- Amount paid
- Payment status
- Notes

### Validation

- Vendor name is required.
- User must have access to the event.
- Monetary values must be valid.
- Phone/email must be validated when provided.

---

## FR-VENDOR-002 — View Vendors

The system shall display all vendors associated with an event.

Users should be able to identify:

- Vendor category
- Cost
- Payment status

---

## FR-VENDOR-003 — View Vendor Details

The system shall provide a detailed vendor view.

The view shall include:

- Contact information
- Financial information
- Payment status
- Notes

---

## FR-VENDOR-004 — Update Vendor

The system shall allow users to update vendor information.

---

## FR-VENDOR-005 — Delete Vendor

The system shall allow users to delete a vendor after confirmation.

---

## FR-VENDOR-006 — Vendor Payment Status

The system shall support:

- Pending
- Partially Paid
- Paid

Payment status should be derived from the payment information where appropriate.

---

## FR-VENDOR-007 — Vendor Financial Summary

The system shall calculate:

- Total estimated vendor cost
- Total actual vendor cost
- Total amount paid
- Total amount pending

---

# 9. Guest Management Requirements

## FR-GUEST-001 — Add Guest

The system shall allow users to add a guest.

### Fields

- Name
- Phone number
- Group/family
- RSVP status
- Number of attendees
- Meal preference
- Notes

### Validation

- Guest name is required.
- Number of attendees must be a valid positive number.
- RSVP status must use a supported value.

---

## FR-GUEST-002 — View Guests

The system shall display guests associated with an event.

---

## FR-GUEST-003 — Update Guest

The system shall allow users to update guest information.

---

## FR-GUEST-004 — Delete Guest

The system shall allow users to delete a guest after confirmation.

---

## FR-GUEST-005 — RSVP Status

The system shall support the following initial RSVP states:

- Pending
- Attending
- Not Attending
- Maybe

---

## FR-GUEST-006 — Guest Groups

The system shall allow guests to be associated with a group or family.

This will support future filtering and guest-management functionality.

---

## FR-GUEST-007 — Guest Summary

The system shall calculate:

- Total guests
- Total expected attendees
- Attending guests
- Pending RSVPs
- Not attending guests

---

# 10. Ideas Management Requirements

## FR-IDEA-001 — Create Idea

The system shall allow users to save an event idea.

### Supported content

- Text
- Image
- URL/link

### Optional information

- Title
- Description
- Category

---

## FR-IDEA-002 — View Ideas

The system shall display ideas associated with the selected event.

---

## FR-IDEA-003 — Update Idea

The system shall allow users to modify saved ideas.

---

## FR-IDEA-004 — Delete Idea

The system shall allow users to delete an idea after confirmation.

---

## FR-IDEA-005 — Categorize Idea

The system shall allow ideas to be categorized.

Initial categories:

- Decor
- Outfit
- Photography
- Invitation
- Venue
- Food
- Makeup
- Other

---

## FR-IDEA-006 — Upload Idea Image

The system shall allow users to upload images associated with ideas.

### Requirements

- Supported file types must be configurable.
- Maximum file size must be enforced.
- Uploaded files must be validated.
- Files must be stored outside the relational database.
- Unauthorized users must not be able to access private event files.

---

## FR-IDEA-007 — Save External Link

The system shall allow users to save an external URL as an idea.

The system shall validate that the submitted value is a valid URL.

---

# 11. Subscription Requirements

## FR-SUB-001 — Start Trial

The system shall automatically start a three-day trial when a new user successfully registers.

### Trial information

The system shall record:

- Trial start timestamp
- Trial end timestamp
- Trial status

---

## FR-SUB-002 — Trial Access

Users with an active trial shall have access to all MVP functionality.

The backend shall determine whether the trial is currently active.

---

## FR-SUB-003 — Display Trial Status

The application shall display relevant trial information.

Example:

```text
Free trial
2 days remaining
```

---

## FR-SUB-004 — Trial Expiration

When the trial period ends and the user does not have an active subscription, the system shall restrict access to protected application functionality according to the defined subscription policy.

The user shall be redirected to the subscription screen.

---

## FR-SUB-005 — Subscription Plan

The system shall provide at least one paid subscription plan for the initial release.

The pricing and billing interval shall be configurable.

---

## FR-SUB-006 — Create Subscription

The system shall allow an eligible user to initiate a subscription through the configured payment provider.

---

## FR-SUB-007 — Payment Processing

The system shall redirect or otherwise integrate with the configured payment provider to process subscription payments.

Payment credentials must not be stored directly by the application unless explicitly required and compliant with applicable payment-security requirements.

---

## FR-SUB-008 — Payment Verification

The backend shall verify payment/subscription status using trusted payment-provider responses and webhooks.

The frontend shall not be considered the source of truth for payment status.

---

## FR-SUB-009 — Active Subscription

Users with an active subscription shall have access to the application's protected functionality.

---

## FR-SUB-010 — Failed Payment

The system shall handle failed subscription payments.

Possible states:

- Payment failed
- Past due
- Grace period
- Subscription expired

Exact behavior shall be determined by the final payment-provider configuration.

---

## FR-SUB-011 — Subscription Cancellation

The system shall allow users to cancel their subscription according to the configured billing rules.

---

## FR-SUB-012 — Subscription Status

The system shall maintain a subscription status.

Initial statuses:

```text
TRIAL
ACTIVE
PAST_DUE
CANCELLED
EXPIRED
```

---

# 12. Settings Requirements

## FR-SET-001 — View Profile

The system shall allow users to view their account information.

---

## FR-SET-002 — Update Profile

The system shall allow users to update supported profile information.

Initial information may include:

- Name
- Profile image
- Mobile number, subject to authentication rules

---

## FR-SET-003 — Notification Preferences

The system shall allow users to manage supported notification preferences.

The initial release may provide only a basic notification preference model.

---

## FR-SET-004 — View Subscription

The system shall allow users to access their subscription information through Settings.

---

## FR-SET-005 — Delete Account

The system shall allow a user to request account deletion.

### Requirements

- The user must explicitly confirm the action.
- The system must explain the consequences before deletion.
- Associated data must follow the defined data-retention policy.
- The user's active session must be terminated after deletion.

---

# 13. Authorization Requirements

## FR-SYS-001 — Event-Level Authorization

The system shall verify that the authenticated user has permission to access the requested event.

This authorization check must occur on the backend.

---

## FR-SYS-002 — Resource-Level Authorization

The system shall ensure that users cannot access or modify resources belonging to another event.

This includes:

- Tasks
- Vendors
- Guests
- Ideas
- Files
- Event settings

---

## FR-SYS-003 — API Authorization

Every protected API endpoint shall validate authentication and authorization before returning or modifying protected data.

---

# 14. File Management Requirements

## FR-SYS-004 — File Upload

The system shall support file uploads where required by the application.

Initial use case:

- Idea images
- Event image

### Requirements

- Validate file type.
- Validate file size.
- Generate secure storage identifiers.
- Prevent executable file uploads.
- Store files using private access controls where appropriate.
- Associate uploaded files with the correct event/resource.

---

## FR-SYS-005 — File Access

Users shall only be able to access files belonging to events they are authorized to access.

---

# 15. Error Handling Requirements

## FR-SYS-006 — Validation Errors

The system shall provide clear validation feedback when user input is invalid.

Examples:

```text
Event name is required.
Please enter a valid mobile number.
Please select an event date.
```

---

## FR-SYS-007 — API Errors

The backend shall return structured API errors.

The frontend shall convert technical errors into user-friendly messages.

Raw stack traces and internal system information must never be displayed to users.

---

## FR-SYS-008 — Network Failure

The frontend shall provide an appropriate error state when a network request fails.

Where appropriate, users should be able to retry the operation.

---

## FR-SYS-009 — Payment Failure

Payment failures shall provide a clear explanation and allow the user to retry where supported.

---

# 16. Loading and Empty States

## FR-SYS-010 — Loading State

The application shall provide loading states for operations that require asynchronous processing.

Examples:

- Loading dashboard
- Loading tasks
- Loading guests
- Uploading an image
- Processing payment

---

## FR-SYS-011 — Empty State

The application shall provide meaningful empty states.

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
No ideas saved yet.
Save your first inspiration.
```

Empty states should provide an appropriate primary action.

---

# 17. Search, Filtering and Sorting

## FR-SYS-012 — Task Filtering

Users should be able to filter tasks by:

- Status
- Priority
- Category

---

## FR-SYS-013 — Guest Filtering

Users should be able to filter guests by:

- RSVP status
- Group/family

---

## FR-SYS-014 — Vendor Filtering

Users should be able to filter vendors by:

- Category
- Payment status

Advanced search functionality may be introduced after the MVP if required.

---

# 18. Data Integrity Requirements

## FR-SYS-015 — Data Persistence

Successfully created or updated records shall be persisted in PostgreSQL.

---

## FR-SYS-016 — Referential Integrity

The system shall maintain appropriate relationships between:

- Users
- Events
- Tasks
- Vendors
- Guests
- Ideas
- Subscriptions
- Files

---

## FR-SYS-017 — Timestamps

Relevant entities shall maintain creation and update timestamps.

At minimum:

```text
created_at
updated_at
```

---

# 19. Notification Requirements

## FR-NOTIF-001 — Trial Expiry Reminder

The system should notify users when their free trial is approaching expiration.

The exact notification channels will be defined separately.

---

## FR-NOTIF-002 — Task Reminder

The system may notify users about upcoming or overdue tasks.

This functionality may be implemented after the initial MVP if notification infrastructure is not required for launch.

---

# 20. System Behavior Requirements

## FR-SYS-018 — Backend as Source of Truth

The backend shall be the authoritative source for:

- Authentication status
- Authorization
- Event ownership
- Trial status
- Subscription status
- Payment status
- Data persistence

Frontend state shall not be trusted for security-sensitive decisions.

---

## FR-SYS-019 — Time-Based Access

Trial and subscription access shall be determined using server-side timestamps.

The frontend must not independently determine whether a user's trial has expired.

---

## FR-SYS-020 — Configuration

The system shall keep configurable values outside application source code where appropriate.

Examples:

- OTP expiry
- OTP retry limits
- OTP resend cooldown
- Trial duration
- Subscription pricing
- File size limits
- Allowed file types

---

# 21. MVP Functional Scope

The following requirements are considered **mandatory for the initial MVP**:

### Authentication

- FR-AUTH-001
- FR-AUTH-002
- FR-AUTH-003
- FR-AUTH-004
- FR-AUTH-006
- FR-AUTH-007
- FR-AUTH-008

### Onboarding

- FR-ONB-001
- FR-ONB-002
- FR-ONB-003

### Events

- FR-EVENT-001
- FR-EVENT-002
- FR-EVENT-003
- FR-EVENT-005

### Dashboard

- FR-DASH-001
- FR-DASH-002
- FR-DASH-003
- FR-DASH-004
- FR-DASH-005
- FR-DASH-006
- FR-DASH-007

### Tasks

- FR-TASK-001 through FR-TASK-008

### Vendors

- FR-VENDOR-001 through FR-VENDOR-007

### Guests

- FR-GUEST-001 through FR-GUEST-007

### Ideas

- FR-IDEA-001 through FR-IDEA-007

### Subscription

- FR-SUB-001 through FR-SUB-012

### Settings

- FR-SET-001 through FR-SET-005

### System

- FR-SYS-001 through FR-SYS-020

---

# 22. Future Functional Requirements

The following functionality is intentionally outside the initial MVP:

- Multiple event members
- Event collaboration
- Role-based event access
- Budget and expense management
- Calendar integration
- Document management
- Digital invitations
- Guest RSVP portal
- WhatsApp integration
- Email invitations
- Vendor marketplace
- Vendor recommendations
- AI planning assistant
- AI-generated checklists
- Automated event timelines
- Advanced analytics
- Event templates
- Recurring events
- Multi-event dashboards

These features should not be implemented unless they are explicitly moved into the MVP scope.

---

# 23. Requirement Traceability

Each functional requirement should eventually map to the following:

```text
Functional Requirement
        ↓
User Story
        ↓
User Flow
        ↓
UI Screen
        ↓
API Endpoint
        ↓
Database Entity
        ↓
Test Case
```

Example:

```text
FR-VENDOR-001
      ↓
US-VENDOR-001
      ↓
UF-VENDOR-001
      ↓
SCR-007 Vendors
      ↓
POST /api/v1/events/{event_id}/vendors
      ↓
vendors table
      ↓
TC-VENDOR-001
```

This traceability should be maintained throughout development.

---

# 24. Requirement Status

| Module           |      Requirements       |      Status       |
| :--------------- | :---------------------: | :---------------: |
| Authentication   |   AUTH-001 → AUTH-008   |      Defined      |
| Onboarding       |    ONB-001 → ONB-003    |      Defined      |
| Event Management |  EVENT-001 → EVENT-005  |      Defined      |
| Dashboard        |   DASH-001 → DASH-007   |      Defined      |
| Tasks            |   TASK-001 → TASK-008   |      Defined      |
| Vendors          | VENDOR-001 → VENDOR-007 |      Defined      |
| Guests           |  GUEST-001 → GUEST-007  |      Defined      |
| Ideas            |   IDEA-001 → IDEA-007   |      Defined      |
| Subscription     |    SUB-001 → SUB-012    |      Defined      |
| Settings         |    SET-001 → SET-005    |      Defined      |
| Notifications    |  NOTIF-001 → NOTIF-002  | Partially Defined |
| System           |    SYS-001 → SYS-020    |      Defined      |

---

# 25. Related Documentation

This document should be used together with:

- `docs/01-product/product-definition.md`
- `docs/01-product/mvp-scope.md`
- `docs/02-requirements/user-stories.md`
- `docs/03-user-experience/user-flows.md`
- `docs/03-user-experience/screen-inventory.md`
- `docs/04-architecture/architecture.md`
- `docs/05-database/database-design.md`
- `docs/06-api/api-specification.md`
- `docs/07-business/subscription.md`
- `docs/08-security/security.md`
- `docs/09-infrastructure/infrastructure.md`
