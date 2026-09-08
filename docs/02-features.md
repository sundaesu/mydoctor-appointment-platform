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

Represents a planned or scheduled visit between a patient, doctor, clinic, and date/time.

### Token

Represents a patient's operational position in a clinic session or queue. A scheduled appointment may produce a token when the patient arrives or checks in, and a walk-in patient may receive a token without a pre-booked appointment.

### Appointment and Token Relationship

- Appointment and queue token are separate concepts.
- Appointment lifecycle and queue-token lifecycle are independent.
- Queue state represents what is happening operationally inside the clinic.

Examples:

- Online appointment -> patient books an appointment -> patient arrives/checks in -> queue token -> consultation
- Phone appointment -> receptionist books an appointment -> patient arrives/checks in -> queue token -> consultation
- Walk-in -> no appointment -> queue token -> consultation
- Cancelled appointment -> appointment ends as cancelled -> any existing queue token is cancelled or removed as a separate operational action
- Patient no-show -> appointment may move to no-show according to clinic policy -> any existing queue token is marked no-show or removed as a separate operational action

## Appointment Lifecycle

The initial lifecycle is proposed and will be finalized during architecture design.

```text
BOOKED
→ CONFIRMED
→ ARRIVED
→ CONSULTING
→ COMPLETED
```

Possible alternate states:

```text
BOOKED → CANCELLED
CONFIRMED → CANCELLED
BOOKED → NO_SHOW
CONFIRMED → NO_SHOW
```

Queue-token lifecycle is separate from appointment lifecycle and is intentionally not defined here.

## Product Principles

- Simplicity
- Real-world clinic compatibility
- Mobile-first patient experience
- Real-time queue updates
- Multi-clinic support
- Multi-tenant support
- Extensible architecture
- Privacy and security
