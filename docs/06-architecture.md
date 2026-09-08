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
- Redis may be used for high-frequency queue state, cache, or coordination
- Real-time communication may use WebSockets or another suitable mechanism

## Multi-Clinic and Future Scale

The backend should maintain clear module boundaries so individual modules can potentially become services later if scale requires it.

## Architecture Status

The state machine and other important implementation details are proposed until architecture design is completed.
