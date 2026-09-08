# Features

## Primary Experiences

### PatientView

Patient-facing application for finding doctors, booking appointments, getting tokens, and tracking queue progress.

### DoctorView

Doctor-facing application for today's schedule, queue handling, availability, and consultation progression.

### ReceptionistView

Clinic operational interface for registration, booking, queue management, and daily clinic operations.

## Core Concepts

### Appointment

Represents a scheduled relationship between a patient, doctor, clinic, and date/time.

### Token

Represents a patient's position in a clinic's queue. A scheduled appointment may produce a token, and a walk-in patient may receive a token without a pre-booked appointment.

### Appointment and Token Relationship

- Online appointment -> appointment -> queue token
- Phone appointment -> appointment -> queue token
- Walk-in -> queue token

## Appointment Lifecycle

The initial lifecycle is proposed and will be finalized during architecture design.

```text
BOOKED
→ CONFIRMED
→ ARRIVED
→ WAITING
→ CONSULTING
→ COMPLETED
```

Possible alternate states:

```text
BOOKED → CANCELLED
CONFIRMED → CANCELLED
ARRIVED → NO_SHOW
WAITING → SKIPPED
```

## Product Principles

- Simplicity
- Real-world clinic compatibility
- Mobile-first patient experience
- Real-time queue updates
- Multi-clinic support
- Multi-tenant support
- Extensible architecture
- Privacy and security
