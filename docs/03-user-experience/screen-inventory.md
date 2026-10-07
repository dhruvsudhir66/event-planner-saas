# Screen Inventory

## Initial Product Screens

This document defines the screens planned for the initial version of the Event Planner application. It serves as the reference for UX design, frontend development, navigation, and acceptance testing.

### Scope

The initial release focuses on four core areas:

- **Authentication** — account creation and secure login
- **Event Management** — creating and managing an event
- **Event Planning** — tasks, vendors, guests, and ideas
- **Account & Billing** — subscription and account management

---

## Screen Overview

| ID          | Screen           | Purpose                                                                          | Authentication | Priority |
| :---------- | :--------------- | :------------------------------------------------------------------------------- | :------------: | :------: |
| **SCR-001** | Landing          | Introduce the product, communicate its value, and drive users to start a trial   |       No       |    P0    |
| **SCR-002** | Login            | Allow existing users to access their account using their mobile number           |       No       |    P0    |
| **SCR-003** | OTP Verification | Verify the user's mobile number and complete authentication                      |       No       |    P0    |
| **SCR-004** | Create Event     | Collect the essential information required to create a new event                 |      Yes       |    P0    |
| **SCR-005** | Dashboard        | Provide a central overview of the event, planning progress, and upcoming actions |      Yes       |    P0    |
| **SCR-006** | Tasks            | Create, organize, track, and complete event-related tasks                        |      Yes       |    P0    |
| **SCR-007** | Vendors          | Store and manage vendor information, costs, payments, and notes                  |      Yes       |    P0    |
| **SCR-008** | Guests           | Manage the guest list, groups, RSVP status, and attendance information           |      Yes       |    P0    |
| **SCR-009** | Ideas            | Capture and organize event inspiration, notes, images, and links                 |      Yes       |    P0    |
| **SCR-010** | Subscription     | View trial status, subscription plan, billing status, and payment options        |      Yes       |    P0    |
| **SCR-011** | Settings         | Manage profile, account preferences, notifications, and account-related settings |      Yes       |    P1    |

---

## Authentication Flow

```text
SCR-001
Landing
   │
   ├── Start Free Trial
   │
   ▼
SCR-002
Login
   │
   ▼
SCR-003
OTP Verification
   │
   ├── New User ──────► SCR-004 Create Event
   │
   └── Existing User ─► SCR-005 Dashboard
```

---

## Core Application Flow

```text
                    ┌─────────────────┐
                    │    Dashboard    │
                    │    SCR-005      │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
     ┌──────────┐       ┌──────────┐      ┌──────────┐
     │  Tasks   │       │ Vendors  │      │  Guests  │
     │ SCR-006  │       │ SCR-007  │      │ SCR-008  │
     └──────────┘       └──────────┘      └──────────┘
                             │
                             ▼
                       ┌──────────┐
                       │  Ideas   │
                       │ SCR-009  │
                       └──────────┘

             ┌────────────────────────────┐
             │ Subscription / Settings    │
             │ SCR-010 / SCR-011         │
             └────────────────────────────┘
```

---

## Screen Details

### SCR-001 — Landing

**Purpose:**  
Introduce the Event Planner application and clearly communicate its value proposition.

**Primary actions:**

- Start free trial
- Login
- View product benefits
- View pricing

**Authentication:** Not required

---

### SCR-002 — Login

**Purpose:**  
Allow existing users to initiate authentication.

**Primary actions:**

- Enter mobile number
- Request OTP
- Continue to verification

**Authentication:** Not required

---

### SCR-003 — OTP Verification

**Purpose:**  
Verify the user's mobile number.

**Primary actions:**

- Enter OTP
- Verify OTP
- Resend OTP
- Change mobile number

**Authentication:** Not required

---

### SCR-004 — Create Event

**Purpose:**  
Collect the minimum information required to create the user's first event.

**Initial information:**

- Event name
- Event type
- Event date
- Location
- Optional event image

**Authentication:** Required

---

### SCR-005 — Dashboard

**Purpose:**  
Provide the user with a clear overview of their event and identify what requires attention.

**Key information:**

- Event name
- Event date
- Countdown
- Planning progress
- Pending tasks
- Upcoming tasks
- Vendor summary
- Guest summary
- Budget summary
- Subscription/trial status

**Primary actions:**

- Add task
- Add vendor
- Add guest
- Add idea

**Authentication:** Required

---

### SCR-006 — Tasks

**Purpose:**  
Help users organize and track everything that needs to be completed before the event.

**Core capabilities:**

- Create task
- Edit task
- Complete task
- Delete task
- Set due date
- Set priority
- Assign category
- Filter and sort tasks

**Authentication:** Required

---

### SCR-007 — Vendors

**Purpose:**  
Maintain a centralized record of all vendors involved in the event.

**Core information:**

- Vendor name
- Category
- Contact details
- Estimated cost
- Actual cost
- Amount paid
- Payment status
- Notes

**Authentication:** Required

---

### SCR-008 — Guests

**Purpose:**  
Maintain and manage the event guest list.

**Core information:**

- Guest name
- Contact information
- Group/family
- RSVP status
- Number of attendees
- Meal preference
- Notes

**Authentication:** Required

---

### SCR-009 — Ideas

**Purpose:**  
Provide a quick, flexible space for capturing event inspiration without requiring users to organize everything immediately.

**Supported content:**

- Text notes
- Images
- Links
- Screenshots
- References/inspiration

**Potential categories:**

- Decor
- Outfit
- Photography
- Invitation
- Venue
- Food
- Makeup
- Other

**Authentication:** Required

---

### SCR-010 — Subscription

**Purpose:**  
Allow users to understand and manage their subscription after the free trial.

**Key information:**

- Trial status
- Trial expiry date
- Current plan
- Subscription status
- Next billing date
- Payment status

**Primary actions:**

- Subscribe
- Upgrade
- Cancel subscription
- Manage billing

**Authentication:** Required

---

### SCR-011 — Settings

**Purpose:**  
Allow users to manage their account and application preferences.

**Initial sections:**

- Profile
- Mobile number
- Notifications
- Subscription
- Account preferences
- Privacy
- Logout
- Delete account

**Authentication:** Required

---

## Priority Definitions

| Priority | Meaning                                           |
| :------: | :------------------------------------------------ |
|  **P0**  | Required for the initial MVP                      |
|  **P1**  | Important but can be introduced shortly after MVP |
|  **P2**  | Future enhancement                                |

---

## Initial Navigation

Once authenticated, the primary application navigation should provide access to:

```text
Dashboard
Tasks
Vendors
Guests
Ideas
Settings
```

Subscription information should be accessible from Settings and surfaced contextually when the user's trial is approaching expiry or has expired.

---

## Future Screens

The following screens are intentionally excluded from the initial release and may be introduced in later versions:

- Budget & Expenses
- Calendar
- Documents
- Notifications Center
- Event Members / Collaboration
- Invitations
- Guest RSVP Portal
- Vendor Marketplace
- AI Planning Assistant
- Reports & Analytics
- Event Timeline
- Event Checklist Templates
