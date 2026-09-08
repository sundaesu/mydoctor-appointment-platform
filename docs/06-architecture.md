# Architecture

## Current Direction

Start with a modular monolith. Do not begin with many microservices.

## Initial Backend Modules

- Authentication
- Patient
- Doctor
- Clinic
- Appointment
- Queue
- Notification
- Payment (future)
- Review (future)

## Queue Engine

The queue and token engine is a critical component of the platform.

The backend should treat queue state as an operational domain rather than deriving the entire queue from appointment status.

Conceptually:

```text
Appointment domain
    |
    | patient scheduled visit
    v

Clinic Session / Queue domain
    |
    | operational visit flow
    v

Queue Token
```

The clinic session may be doctor-specific or clinic-session-specific depending on the clinic operating model, but the important point is that queue state is the live clinic flow and not just a mirror of appointment lifecycle.

The backend should eventually handle:

- Token generation
- Queue ordering
- Current token
- Waiting patients
- Calling next patient
- Skip
- Hold
- Recall
- Completion
- Estimated waiting time
- Doctor delay
- Real-time queue updates

## Data and Real-Time Notes

- PostgreSQL should remain the source of truth
- Redis, WebSockets, Kafka, or similar technologies remain proposed directions unless later documented as final decisions
- Redis may be used for high-frequency queue state, cache, or coordination
- Real-time communication may use WebSockets or another suitable mechanism

## Multi-Clinic and Future Scale

The backend should maintain clear module boundaries so individual modules can potentially become services later if scale requires it.

## Architecture Status

The state machine and other important implementation details are proposed until architecture design is completed.
