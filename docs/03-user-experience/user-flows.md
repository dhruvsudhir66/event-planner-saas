# User Flows

## 1. Purpose

This document defines the primary user journeys within the Event Planner application.

The purpose of these flows is to establish a consistent navigation and interaction model before UI and frontend development begins.

The flows cover:

- User registration and authentication
- Event onboarding
- Dashboard navigation
- Event planning
- Task management
- Vendor management
- Guest management
- Ideas management
- Subscription and trial management
- Account settings

---

# 2. User Journey Overview

The primary user journey is:

```text
Landing
   ↓
Register / Login
   ↓
OTP Verification
   ↓
Create Event
   ↓
Event Dashboard
   ↓
Plan & Organize
   ├── Tasks
   ├── Vendors
   ├── Guests
   └── Ideas
   ↓
Trial Period
   ↓
Subscription
   ↓
Continued Event Management
```

The application should always guide the user toward the next meaningful planning action rather than presenting unnecessary complexity.

---

# 3. Authentication Flows

## UF-AUTH-001 — New User Registration

### Objective

Allow a new user to create an account using their mobile number.

### Flow

```text
Landing
   ↓
Start Free Trial
   ↓
Enter Mobile Number
   ↓
Validate Mobile Number
   ↓
Send OTP
   ↓
OTP Verification
   ↓
OTP Valid?
   ├── No → Show Error
   │          ↓
   │       Retry / Resend OTP
   │
   └── Yes
        ↓
     Create User Account
        ↓
     Start 3-Day Trial
        ↓
     Create First Event
```

### Expected behavior

1. User selects **Start Free Trial**.
2. System displays the mobile number form.
3. User enters their mobile number.
4. System validates the number.
5. System sends an OTP.
6. User enters the OTP.
7. System verifies the OTP.
8. If valid, the user account is created.
9. The 3-day trial begins.
10. User is redirected to event creation.

### Success

User reaches **SCR-004 — Create Event**.

### Failure scenarios

- Invalid mobile number
- OTP expired
- Incorrect OTP
- Maximum OTP attempts exceeded
- SMS delivery failure
- Rate limit exceeded

---

# 4. Existing User Login

## UF-AUTH-002 — User Login

### Objective

Allow an existing user to securely access their account.

### Flow

```text
Login
   ↓
Enter Mobile Number
   ↓
Send OTP
   ↓
Enter OTP
   ↓
OTP Valid?
   ├── No → Error → Retry
   │
   └── Yes
        ↓
     Authenticate User
        ↓
     Subscription Status
        ↓
     Dashboard
```

### Expected behavior

If the user has an active event:

```text
Login
  ↓
Dashboard
```

If the user has no event:

```text
Login
  ↓
Create Event
```

If the trial has expired:

```text
Login
  ↓
Subscription Screen
```

If the subscription is active:

```text
Login
  ↓
Dashboard
```

---

# 5. OTP Verification

## UF-AUTH-003 — OTP Verification

### Flow

```text
Enter Mobile
     ↓
Request OTP
     ↓
OTP Sent
     ↓
Enter OTP
     ↓
Verify
     ↓
 ┌───────────────┐
 │               │
Valid           Invalid
 │               │
 ▼               ▼
Continue       Error
                 │
          ┌──────┴──────┐
          ▼             ▼
        Retry         Resend
```

### Rules

- OTP must expire after a configured period.
- OTP verification attempts must be limited.
- OTP resend must be rate limited.
- The user must be able to change the mobile number.
- The system must not expose sensitive authentication information.

---

# 6. First-Time Onboarding

## UF-ONBOARD-001 — Create First Event

### Objective

Guide a newly registered user from account creation to their first event.

### Flow

```text
Account Created
      ↓
Create Your Event
      ↓
Enter Event Name
      ↓
Select Event Type
      ↓
Select Event Date
      ↓
Enter Location
      ↓
Optional Event Image
      ↓
Create Event
      ↓
Event Created
      ↓
Dashboard
```

### Required information

- Event name
- Event type
- Event date

### Optional information

- Location
- Event image

### Success

The system creates the event and redirects the user to the event dashboard.

---

# 7. Event Creation

## UF-EVENT-001 — Create Event

### Flow

```text
Dashboard
    ↓
Create Event
    ↓
Event Form
    ↓
Validate Information
    ↓
Save Event
    ↓
Event Created
    ↓
Dashboard
```

### Validation

The system must validate:

- Event name
- Event type
- Event date

The event date must be valid and must satisfy the application's date rules.

### Failure

If validation fails:

```text
Event Form
    ↓
Validation Error
    ↓
User Corrects Information
    ↓
Submit Again
```

---

# 8. Event Dashboard

## UF-DASH-001 — View Dashboard

### Objective

Provide the user with an immediate overview of their event and the actions requiring attention.

### Flow

```text
Login
  ↓
Subscription Check
  ↓
Event Check
  ↓
Dashboard
```

### Dashboard should display

```text
Event Information
      ↓
Countdown
      ↓
Planning Progress
      ↓
Upcoming Tasks
      ↓
Vendor Summary
      ↓
Guest Summary
      ↓
Budget Summary
      ↓
Quick Actions
```

### Quick actions

The user should be able to quickly:

- Add task
- Add vendor
- Add guest
- Add idea

---

# 9. Task Management

## UF-TASK-001 — Create Task

### Flow

```text
Dashboard
   ↓
Tasks
   ↓
Add Task
   ↓
Enter Task Details
   ↓
Save
   ↓
Task Created
   ↓
Tasks List
```

### Task information

- Task title
- Description
- Due date
- Priority
- Category

### Success

The new task appears in the task list and contributes to the event's planning progress.

---

## UF-TASK-002 — Complete Task

### Flow

```text
Tasks
  ↓
Select Task
  ↓
Mark Complete
  ↓
Task Status = Completed
  ↓
Update Planning Progress
```

The completed task should remain accessible through completed-task filtering/history.

---

## UF-TASK-003 — Edit or Delete Task

### Edit

```text
Tasks
  ↓
Select Task
  ↓
Edit
  ↓
Modify Details
  ↓
Save
  ↓
Updated Task
```

### Delete

```text
Tasks
  ↓
Select Task
  ↓
Delete
  ↓
Confirmation
  ↓
Confirm
  ↓
Task Removed
```

Destructive actions should require confirmation.

---

# 10. Vendor Management

## UF-VENDOR-001 — Add Vendor

### Flow

```text
Dashboard
   ↓
Vendors
   ↓
Add Vendor
   ↓
Enter Vendor Details
   ↓
Save
   ↓
Vendor Created
   ↓
Vendor List
```

### Vendor information

- Vendor name
- Category
- Phone number
- Email
- Estimated cost
- Actual cost
- Amount paid
- Payment status
- Notes

---

## UF-VENDOR-002 — View Vendor

```text
Vendors
   ↓
Select Vendor
   ↓
Vendor Details
```

The vendor detail view should provide access to:

- Contact information
- Cost information
- Payment status
- Notes
- Edit
- Delete

---

## UF-VENDOR-003 — Update Vendor Payment

```text
Vendor Details
      ↓
Payment Information
      ↓
Enter Amount Paid
      ↓
Save
      ↓
Update Payment Status
      ↓
Update Event Financial Summary
```

Payment status may be:

```text
Pending
Partially Paid
Paid
```

---

# 11. Guest Management

## UF-GUEST-001 — Add Guest

### Flow

```text
Dashboard
   ↓
Guests
   ↓
Add Guest
   ↓
Enter Guest Information
   ↓
Save
   ↓
Guest Added
   ↓
Guest List
```

### Guest information

- Name
- Contact number
- Group/family
- RSVP status
- Number of attendees
- Meal preference
- Notes

---

## UF-GUEST-002 — Update RSVP

```text
Guest List
    ↓
Select Guest
    ↓
Update RSVP
    ↓
Select Status
    ↓
Save
    ↓
Guest List Updated
```

### RSVP statuses

```text
Pending
Attending
Not Attending
Maybe
```

The dashboard guest summary should update automatically.

---

# 12. Ideas Management

## UF-IDEA-001 — Add Idea

### Objective

Allow users to quickly capture inspiration without requiring extensive data entry.

### Flow

```text
Dashboard
   ↓
Ideas
   ↓
Add Idea
   ↓
Choose Content Type
   │
   ├── Note
   ├── Image
   └── Link
   ↓
Enter Content
   ↓
Optional Category
   ↓
Save
   ↓
Idea Added
```

### Principle

The Ideas section should prioritize **speed and flexibility**.

The user should be able to capture an idea within a few seconds.

---

## UF-IDEA-002 — Organize Idea

```text
Ideas
  ↓
Select Idea
  ↓
Edit
  ↓
Add / Change Category
  ↓
Save
```

Potential categories:

- Decor
- Outfit
- Photography
- Invitation
- Venue
- Food
- Makeup
- Other

---

# 13. Subscription & Trial Flows

## UF-SUB-001 — Start Trial

The trial begins when the user's account is successfully created.

```text
Account Created
      ↓
Trial Started
      ↓
trial_started_at
      ↓
trial_ends_at
      ↓
Full Application Access
```

The trial duration is **3 days**.

---

## UF-SUB-002 — Active Trial

```text
User Logs In
     ↓
Check Trial
     ↓
Trial Active?
     │
     ├── Yes
     │    ↓
     │  Dashboard
     │
     └── No
          ↓
       Check Subscription
```

The application should display the remaining trial period in a non-intrusive manner.

---

## UF-SUB-003 — Trial Expiring

The application should inform the user that their trial is approaching expiry.

```text
Trial Active
     ↓
Approaching Expiry
     ↓
Display Trial Reminder
     ↓
User Can Continue Using App
     ↓
Subscription Available
```

The reminder should not prevent normal application usage while the trial is still active.

---

## UF-SUB-004 — Trial Expired

```text
Trial Expires
      ↓
User Opens Application
      ↓
Subscription Check
      ↓
Trial Expired?
      ↓
Subscription Active?
      │
      ├── Yes → Dashboard
      │
      └── No
           ↓
      Subscription Screen
           ↓
      Select Plan
           ↓
      Payment
           ↓
      Payment Successful?
           │
           ├── Yes → Activate Subscription → Dashboard
           │
           └── No → Payment Failed → Retry
```

The server must determine whether the user has access. Trial expiration must not rely solely on frontend logic.

---

# 14. Subscription Cancellation

## UF-SUB-005 — Cancel Subscription

```text
Settings
   ↓
Subscription
   ↓
Cancel Subscription
   ↓
Cancellation Confirmation
   ↓
Confirm
   ↓
Subscription Cancelled
   ↓
Access Until End of Billing Period
   ↓
Subscription Expires
```

The exact behavior may depend on the selected payment provider and subscription model.

---

# 15. Settings Flow

## UF-SET-001 — Account Settings

```text
Dashboard
   ↓
Settings
   ↓
Select Section
   │
   ├── Profile
   ├── Notifications
   ├── Subscription
   ├── Privacy
   └── Account
```

---

## UF-SET-002 — Logout

```text
Settings
   ↓
Logout
   ↓
Clear Session
   ↓
Landing / Login
```

The user's authentication session must be invalidated appropriately.

---

## UF-SET-003 — Delete Account

```text
Settings
   ↓
Delete Account
   ↓
Warning
   ↓
Confirmation
   ↓
Delete Account
   ↓
Data Deletion / Retention Process
   ↓
Logout
   ↓
Landing
```

Account deletion behavior must comply with the application's data-retention and privacy policy.

---

# 16. Global Navigation Flow

Once authenticated, users should be able to navigate between the primary sections without returning to the dashboard.

```text
                    Dashboard
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
      Tasks          Vendors         Guests
        │              │              │
        └──────────────┼──────────────┘
                       │
                       ▼
                     Ideas
                       │
                       ▼
                    Settings
                       │
                       ▼
                 Subscription
```

The primary navigation should remain consistent throughout the authenticated application.

---

# 17. Unauthorized Access Flow

## UF-AUTHZ-001 — Unauthenticated User

If an unauthenticated user attempts to access a protected screen:

```text
Protected Route
      ↓
Authentication Check
      ↓
Authenticated?
      │
      ├── Yes → Continue
      │
      └── No
           ↓
         Login
           ↓
      OTP Verification
           ↓
        Original Route
```

Where possible, the application should return the user to the page they originally attempted to access after successful authentication.

---

# 18. Session Expiration Flow

## UF-AUTH-004 — Expired Session

```text
User Activity
      ↓
API Request
      ↓
Session Valid?
      │
      ├── Yes → Continue
      │
      └── No
           ↓
      Attempt Session Refresh
           │
           ├── Success → Continue
           │
           └── Failure
                ↓
              Login
```

The application should avoid unexpectedly losing unsaved user input.

---

# 19. Error Handling Principles

All major user flows should account for failure scenarios.

### Common errors

```text
Network Error
    ↓
Display User-Friendly Error
    ↓
Retry
```

```text
Validation Error
    ↓
Highlight Invalid Field
    ↓
User Corrects Input
```

```text
Server Error
    ↓
Display Generic Error
    ↓
Allow Retry
```

```text
Payment Failure
    ↓
Explain Failure
    ↓
Retry Payment
```

The application should never expose raw backend errors, stack traces, database errors, or sensitive technical information to the user.

---

# 20. Loading & Empty States

Every major screen must define its loading and empty states.

### Tasks

```text
No Tasks
   ↓
Show Empty State
   ↓
"Add your first task"
```

### Vendors

```text
No Vendors
   ↓
Show Empty State
   ↓
"Add your first vendor"
```

### Guests

```text
No Guests
   ↓
Show Empty State
   ↓
"Add your first guest"
```

### Ideas

```text
No Ideas
   ↓
Show Empty State
   ↓
"Save your first idea"
```

Loading states should be designed as part of the UX rather than added after development.

---

# 21. Core User Journey

The complete initial user journey should be:

```text
                         LANDING
                            │
                  ┌─────────┴─────────┐
                  │                   │
             New User            Existing User
                  │                   │
                  ▼                   ▼
              Register              Login
                  │                   │
                  └─────────┬─────────┘
                            ▼
                       OTP Verify
                            │
                            ▼
                    Authentication
                            │
                            ▼
                    ┌───────────────┐
                    │ Event Exists? │
                    └───────┬───────┘
                       No   │   Yes
                       │    │
                       ▼    └─────────────┐
                  Create Event             │
                       │                   │
                       └─────────┬─────────┘
                                 ▼
                             DASHBOARD
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
            Tasks             Vendors             Guests
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
                                 ▼
                               Ideas
                                 │
                                 ▼
                            Subscription
                                 │
                                 ▼
                              Settings
```

---

# 22. Flow Design Principles

The following principles should guide all UX decisions:

### 1. Minimize complexity

Users should not need to understand the application's internal structure.

### 2. Minimize required input

Only request information that is necessary at each step.

### 3. Prefer progressive disclosure

Advanced options should appear only when they are relevant.

### 4. Always provide a clear next action

Every major screen should have an obvious primary action.

### 5. Make capturing information fast

Especially for tasks and ideas, users should be able to add information with minimal friction.

### 6. Preserve user data

Unexpected navigation, session expiration, network failures, or errors should not unnecessarily discard user input.

### 7. Design for mobile first

The majority of event-planning activity is expected to happen on mobile devices. All flows should therefore work comfortably on small screens before optimizing for desktop.

### 8. Keep the dashboard action-oriented

The dashboard should answer:

> **"What should I do next?"**

rather than simply displaying statistics.

---

# 23. Flow Status

| Flow Area          | Status  | Next Step        |
| :----------------- | :-----: | :--------------- |
| Authentication     | Defined | UX Design        |
| Onboarding         | Defined | UX Design        |
| Event Management   | Defined | UX Design        |
| Dashboard          | Defined | UX Design        |
| Tasks              | Defined | UX Design        |
| Vendors            | Defined | UX Design        |
| Guests             | Defined | UX Design        |
| Ideas              | Defined | UX Design        |
| Subscription       | Defined | Business Rules   |
| Settings           | Defined | UX Design        |
| Error Handling     | Defined | Technical Design |
| Session Management | Defined | Technical Design |

---

## 24. Related Documentation

This document should be used together with:

- `docs/01-product/product-definition.md`
- `docs/01-product/mvp-scope.md`
- `docs/02-requirements/functional-requirements.md`
- `docs/02-requirements/user-stories.md`
- `docs/03-user-experience/screen-inventory.md`

The user flows should be considered the bridge between **product requirements** and **UI/UX design**.
