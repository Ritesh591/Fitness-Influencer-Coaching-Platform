# FitCoach Platform — ER Diagram

---

## What This Repository Contains

| File              | Description                                                                              |
| ----------------- | ---------------------------------------------------------------------------------------- |
| `index.html`      | Interactive HTML page with the ER diagram, legend, entity walkthrough, and business Q&A |
| `erd_diagram.png` | Exported image of the ER diagram                                                         |
| `README.md`       | This file                                                                                |

---

## The Business

A fitness influencer has built an online coaching platform where one or more trainers manage multiple clients through structured digital coaching. The platform needs to support:

- Trainer and client onboarding with separate profile data
- Coaching plans/programs that clients can subscribe to
- Scheduled live sessions and consultations
- Weekly client check-ins and asynchronous trainer feedback
- Objective progress tracking (weight, body measurements) over time
- Custom diet plans and workout routines per subscription
- Payment and billing records per subscription

---

## ER Diagram Overview

The diagram models **12 entities**:

| Entity             | Type        | Purpose                                                                      |
| ------------------ | ----------- | ---------------------------------------------------------------------------- |
| `USER`             | Core        | Shared identity for all platform users — trainers and clients                |
| `TRAINER_PROFILE`  | Core        | Trainer-specific details: specialization, bio, experience                    |
| `CLIENT_PROFILE`   | Core        | Client health baseline: age, height, initial weight, fitness goal            |
| `COACHING_PLAN`    | Product     | Program template created by a trainer; can have many subscribers             |
| `SUBSCRIPTION`     | Junction    | Client enrollment in a plan — anchor for all coaching activity               |
| `SESSION`          | Transaction | Scheduled live consultation or video call between trainer and client         |
| `CHECKIN`          | Transaction | Weekly self-report submitted by a client (subjective data)                   |
| `TRAINER_FEEDBACK` | Operational | Trainer's async written response to a client check-in                        |
| `PROGRESS_ENTRY`   | Analytics   | Objective measurements (weight, body fat %, measurements) logged over time   |
| `PAYMENT`          | Financial   | Payment transaction record per subscription billing cycle                    |
| `DIET_PLAN`        | Content     | Custom nutrition plan created by trainer for a specific client subscription  |
| `WORKOUT_PLAN`     | Content     | Custom training routine created by trainer for a specific client subscription|

---

## Key Design Decisions

### 1. Single User table with separate profiles

Trainers and clients share one `USER` table for identity (name, email, auth). Trainer-specific and client-specific data live in `TRAINER_PROFILE` and `CLIENT_PROFILE` respectively, each linked 1-to-1 with USER. This avoids duplicating contact fields across two tables while keeping role-specific data clean.

### 2. Subscription is the central anchor

`SUBSCRIPTION` is the most important operational entity. It links a client to a plan and carries the enrollment dates and status. Sessions, check-ins, progress entries, payments, diet plans, and workout plans all hang off the subscription — not off the client or plan directly. This means all coaching activity is scoped to a specific enrollment period.

### 3. Session and Check-in are separate

A `SESSION` is a scheduled live interaction with a time slot, medium, and status. A `CHECKIN` is a weekly asynchronous self-report the client fills in. These are fundamentally different operations and must not be merged — you need to independently count sessions attended vs. check-ins submitted.

### 4. Trainer feedback is separate from check-ins

`TRAINER_FEEDBACK` is its own table linked to a `checkin_id`. This means the trainer reads the check-in and then creates a feedback record asynchronously. Feedback doesn't modify the check-in, and the check-in exists whether or not the trainer has responded yet.

### 5. Progress tracking is separate from check-ins and client profile

`PROGRESS_ENTRY` stores objective measurements (weight, body fat %, body measurements) as time-series rows. It is separate from `CLIENT_PROFILE` (which holds only the initial onboarding baseline) and from `CHECKIN` (which holds subjective weekly feelings). This separation lets you chart a weight-loss graph from progress entries without mixing in subjective data.

### 6. Diet and workout plans are separate content tables

`DIET_PLAN` and `WORKOUT_PLAN` are linked to a `subscription_id` — not to the coaching plan template. This means the trainer can update a client's nutrition or training content mid-subscription without changing the subscription or plan record. A client on a combined-type plan has both; a client on a workout-only plan has only a workout plan row.

---

## Relationships and Cardinality

| Relationship                          | Notation    | Meaning                                                  |
| ------------------------------------- | ----------- | -------------------------------------------------------- |
| User → Trainer_Profile                | `\|\|--o\|` | One user, zero or one trainer profile                    |
| User → Client_Profile                 | `\|\|--o\|` | One user, zero or one client profile                     |
| Trainer_Profile → Coaching_Plan       | `\|\|--o{`  | One trainer, zero or many plans created                  |
| Coaching_Plan → Subscription          | `\|\|--o{`  | One plan, zero or many client subscriptions              |
| Client_Profile → Subscription         | `\|\|--o{`  | One client, zero or many subscriptions over time         |
| Subscription → Session                | `\|\|--o{`  | One subscription, zero or many sessions                  |
| Trainer_Profile → Session             | `\|\|--o{`  | One trainer, zero or many sessions conducted             |
| Client_Profile → Session              | `\|\|--o{`  | One client, zero or many sessions attended               |
| Subscription → CheckIn                | `\|\|--o{`  | One subscription, zero or many weekly check-ins          |
| Client_Profile → CheckIn              | `\|\|--o{`  | One client, zero or many check-ins submitted             |
| CheckIn → Trainer_Feedback            | `\|\|--o\|` | One check-in, zero or one trainer feedback               |
| Trainer_Profile → Trainer_Feedback    | `\|\|--o{`  | One trainer, zero or many feedback entries written       |
| Client_Profile → Progress_Entry       | `\|\|--o{`  | One client, zero or many progress measurements           |
| Subscription → Progress_Entry         | `\|\|--o{`  | One subscription, zero or many progress entries          |
| Subscription → Payment                | `\|\|--o{`  | One subscription, zero or many payment records           |
| Client_Profile → Payment              | `\|\|--o{`  | One client, zero or many payments made                   |
| Subscription → Diet_Plan              | `\|\|--o{`  | One subscription, zero or many diet plan versions        |
| Trainer_Profile → Diet_Plan           | `\|\|--o{`  | One trainer, zero or many diet plans created             |
| Subscription → Workout_Plan           | `\|\|--o{`  | One subscription, zero or many workout plan versions     |
| Trainer_Profile → Workout_Plan        | `\|\|--o{`  | One trainer, zero or many workout plans created          |

---

## Notation Reference

| Symbol   | Meaning       |
| -------- | ------------- |
| `\|\|`   | Exactly one   |
| `o\|`    | Zero or one   |
| `o{`     | Zero or many  |
| `\|{`    | One or many   |

**PK** = Primary Key · **FK** = Foreign Key
