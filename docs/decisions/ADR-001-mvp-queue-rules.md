# ADR-001: MVP Queue Rules

Status: Finalized MVP baseline
Date: 2026-09-08

## Context

The platform cannot move into database or API design until the queue behavior is stable enough to be described consistently in product and architecture documents.

The core distinction is:

- `Appointment` = planned or scheduled visit
- `Queue Token` = operational position in a clinic session

These are separate lifecycles. Booking an appointment does not automatically mean the patient is physically present in the clinic, and queue position must reflect real operational flow rather than booking intent alone.

## Decision

### 1. Queue Boundary

**DECIDED**

The MVP queue is scoped to:

- Clinic
- Doctor
- Session

That means a doctor may have multiple sessions in a day and may work at multiple clinics. Each clinic session has its own live queue.

### 2. Appointment and Token Separation

**DECIDED**

- An appointment does not create a queue token by itself
- A queue token is created when the patient arrives or checks in
- A walk-in may receive a queue token without having an appointment
- Appointment cancellation and queue-token removal are separate operational actions

### 3. Token Numbering

**DECIDED**

Token numbering resets for each doctor session.

This keeps the MVP understandable for clinic staff and patients without introducing a global numbering scheme too early.

### 4. Early Arrival

**DECIDED**

A patient who arrives before their appointment time may check in, but early arrival does not automatically give them priority over patients who are already eligible to be served.

Why this is the recommended policy:

- it prevents early check-ins from jumping ahead of patients who are already valid for service
- it keeps appointment timing meaningful without forcing a complex queue algorithm
- it remains understandable to receptionists, doctors, and patients

### 5. Late Arrival

**DECIDED**

Recommended MVP behavior:

- appointments still have scheduled times
- use a 15-minute grace period as the initial MVP default
- if the patient arrives within the grace period, they keep normal appointment eligibility
- after the grace period, receptionists or clinic staff may place the patient behind currently eligible patients or otherwise reclassify the queue position
- the 15-minute value should not become a permanent global rule; clinics should eventually be able to configure it

The exact clinic-specific configuration mechanism remains a future design detail.

### 6. No-Show

**DECIDED**

`NO_SHOW` means the patient failed to arrive or check in within the clinic's policy.

The platform should not implement an overly complex automatic no-show engine in MVP.

The initial no-show threshold should remain a clinic policy/configuration decision rather than hard-coded platform behavior.

### 7. Skipped / Missed Call

**DECIDED**

Recommended MVP behavior:

- patient is called
- patient does not respond
- staff may recall the patient once
- if the patient misses the recall, staff may move them later in the queue or mark the token appropriately
- all such changes should be auditable

### 8. Manual Queue Override

**DECIDED**

Receptionists and doctors should have controlled ability to reorder or override queue position.

Requirements:

- overrides require a reason
- overrides are auditable
- the MVP should avoid a sophisticated priority or rules engine
- emergency or priority patients can use manual override rather than a separate triage system

### 9. Duplicate / Conflicting Appointments

**DECIDED**

The platform should prevent conflicting overlapping appointments for the same patient and doctor where appropriate.

The system should also support a controlled staff workflow for legitimate exceptions.

### 10. Queue Ordering

**DECIDED**

Recommended MVP ordering model:

**Appointment eligibility + arrival/check-in order**

Meaning:

- a patient cannot be served before their appointment eligibility window
- once eligible, queue position is influenced by actual check-in time
- appointment patients do not automatically leap ahead of every walk-in simply because they have an appointment
- later appointment times should not jump ahead of patients already waiting unless clinic policy explicitly allows it

This is the recommended replacement for the earlier "checked-in appointments first, then walk-ins" wording.

### 11. Patient-Visible Queue Information

**PROPOSED/FUTURE**

The patient should eventually be able to see:

- their token number
- the current token being served
- approximate position
- estimated wait
- status
- whether the doctor is delayed

The exact visibility rules are a future product decision so that patient privacy is preserved.

### 12. Auditability

**DECIDED**

The following queue actions should be auditable conceptually:

- token creation
- token cancellation
- token call
- recall
- skip
- hold
- manual reorder
- doctor delay or unavailability
- completion
- no-show
- emergency or priority override

## Alternatives Considered

### A. "Checked-in appointments first, then walk-ins"

This was the earlier policy in `docs/13-queue-policy.md`.

Problems found in the scenario review:

- later appointment patients can jump ahead of earlier waiting walk-ins in ways that feel unintuitive
- early appointment arrival can be overvalued
- the policy is easy to say but not especially fair in mixed real-world clinic flow

### B. Pure First-Come-First-Served

Rejected for MVP because it makes appointments feel too weak and does not preserve the meaning of appointment scheduling.

### C. Complex scoring or triage engine

Rejected for MVP because it would be hard for staff to understand and too expensive to validate at the Srikakulam rollout stage.

## Reason

The recommended policy keeps the product simple enough for private clinics while still respecting the meaning of appointments.

It also fits the observed scenario problems better than the earlier "appointments first" rule:

- booked patients should not be able to jump ahead before they are actually eligible
- walk-ins should remain viable
- the queue should be explainable without a scoring algorithm
- the live queue should reflect actual clinic flow rather than booking metadata alone

## Consequences

- The queue model needs an appointment eligibility concept
- Early arrivals may check in without automatically moving ahead of already eligible patients
- Late arrivals need a clinic-configurable grace period
- Receptionists and doctors need limited manual override tools with auditability
- Queue state must remain a separate operational domain from appointment state

## Open Questions

- Exact per-clinic configuration shape for the late-arrival grace period
- Exact appointment eligibility window and how it is expressed in clinic policy
- Exact future patient-visibility details beyond the core MVP fields

## Relationship to Other Documents

This ADR clarifies and constrains:

- [`docs/13-queue-policy.md`](../13-queue-policy.md)
- [`docs/14-queue-scenarios.md`](../14-queue-scenarios.md)

It is consistent with, and should be read alongside:

- [`README.md`](../../README.md)
- [`docs/02-features.md`](../02-features.md)
- [`docs/04-user-flows.md`](../04-user-flows.md)
- [`docs/05-mvp-scope.md`](../05-mvp-scope.md)
- [`docs/06-architecture.md`](../06-architecture.md)
- [`docs/12-domain-model.md`](../12-domain-model.md)

If any wording in the older docs conflicts with this ADR, this ADR takes precedence and the wording should be updated to avoid contradictory queue rules.

## Decision Summary

| Rule | Status | MVP policy | Notes |
|---|---|---|---|
| Queue boundary | DECIDED | Clinic + Doctor + Session | One queue per doctor session in a clinic |
| Appointment/token separation | DECIDED | Separate lifecycles | Booking does not create a token |
| Token creation | DECIDED | On arrival/check-in | Walk-ins may get tokens without appointments |
| Walk-ins | DECIDED | Supported in MVP | They join a session queue directly |
| Token numbering | DECIDED | Reset per doctor session | Keep numbering simple |
| Early arrival | DECIDED | Check-in allowed, no automatic priority | Must not jump ahead of already eligible patients |
| Late arrival | DECIDED | 15-minute configurable grace period | Initial MVP default, not a permanent global rule |
| No-show | DECIDED | Failure to arrive/check-in | Clinic policy should drive the threshold |
| Skipped token | DECIDED | One recall in basic MVP | Missed recall can move later or be marked appropriately |
| Manual override | DECIDED | Controlled reorder/override with reason | Auditable; avoid complex rules engine |
| Emergency handling | DECIDED | Manual override | No complex triage system in MVP |
| Duplicate appointments | DECIDED | Prevent or surface conflicts | Controlled exception workflow required |
| Queue ordering | DECIDED | Appointment eligibility + check-in order | Fairer than "appointments first" |
| Patient queue visibility | PROPOSED/FUTURE | Token, current served, approximate position, wait, status, delay | Exact visibility remains future scope |
| Auditability | DECIDED | Audit queue actions | Required for trust and traceability |
