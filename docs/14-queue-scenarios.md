# Queue Scenarios

## Purpose

This document validates the MVP appointment and queue model against realistic private-clinic situations in India before database or API design begins.

The goal is to discover missing or contradictory business rules early, while keeping the core domain principle intact:

- `Appointment` = planned or scheduled visit
- `Queue Token` = operational position in a clinic session

Appointment lifecycle and queue-token lifecycle are independent. A patient can have an appointment without a token if they have not arrived, and a walk-in can have a token without an appointment.

## Scenario Format

Each scenario uses the same structure:

- Scenario
- Actors
- Initial Conditions
- Patient/Appointment Details
- Actions
- Expected Appointment State
- Expected Queue/Token State
- Expected Patient Experience
- Expected Receptionist Experience
- Expected Doctor Experience
- Business Rules Exercised
- Open Questions

## Scenario 1: Normal Online Appointment Flow

**Scenario:** A patient books a 10:00 AM appointment online and arrives at 9:50 AM.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** Dr. Ravi has an active morning session at Clinic A. The patient has booked online for that session.

**Patient/Appointment Details:** Scheduled appointment at 10:00 AM; patient is not yet in the clinic.

**Actions:** Booking -> arrival/check-in -> token generation -> waiting -> called -> consultation -> completed

**Expected Appointment State:** `BOOKED` or `CONFIRMED`, then `ARRIVED`, then consultation-related progress, then `COMPLETED`

**Expected Queue/Token State:** No token before arrival; token created at check-in; token moves `WAITING` -> `CALLED` -> `IN_CONSULTATION` -> `COMPLETED`

**Expected Patient Experience:** Patient sees the appointment confirmed, then sees queue position after check-in, then sees the consultation complete.

**Expected Receptionist Experience:** Receptionist confirms the booking exists, checks the patient in, and creates the queue token.

**Expected Doctor Experience:** Doctor sees the patient in the active queue only after the check-in/token step.

**Business Rules Exercised:** Appointment is not the same as queue position; token is created on arrival/check-in; queue state is operational.

**Open Questions:** Exact pre-check-in patient visibility remains a future product decision.

## Scenario 2: Phone or Receptionist Booking

**Scenario:** A patient calls the clinic, the receptionist creates an appointment, and the patient arrives later.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** Dr. Ravi has an active clinic session. The patient is known to the clinic or gets registered during the call.

**Patient/Appointment Details:** Appointment is created by the receptionist for a future visit.

**Actions:** Receptionist books appointment -> patient later arrives -> check-in -> token generation -> consultation

**Expected Appointment State:** Appointment exists before arrival and remains separate from the queue token state

**Expected Queue/Token State:** No token at booking time; token created at check-in; token is then served in the active queue

**Expected Patient Experience:** The patient receives appointment confirmation first, then queue information after arrival.

**Expected Receptionist Experience:** The receptionist handles the booking first, then activates the visit operationally on arrival.

**Expected Doctor Experience:** The doctor sees the patient only when the session queue becomes active.

**Business Rules Exercised:** Phone booking creates an appointment, but the token still depends on arrival/check-in.

**Open Questions:** None for the MVP; phone bookings create an appointment first and a token only on arrival/check-in.

## Scenario 3: Walk-In Patient

**Scenario:** A patient has no appointment and arrives at the clinic.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** Dr. Ravi is seeing patients in an active clinic session.

**Patient/Appointment Details:** No appointment exists.

**Actions:** Patient arrives -> receptionist searches/registers the patient -> token generation -> waiting -> called -> consultation

**Expected Appointment State:** No appointment

**Expected Queue/Token State:** Token exists independently of appointment; token enters the queue directly

**Expected Patient Experience:** The patient receives a token and waits in line without needing a booking.

**Expected Receptionist Experience:** The receptionist registers or finds the patient and adds them to the session queue.

**Expected Doctor Experience:** The doctor sees the walk-in as part of the same session queue.

**Business Rules Exercised:** A token can exist without an appointment; walk-ins are supported in MVP.

**Open Questions:** Exact identity-verification workflow remains a future operational detail.

## Scenario 4: Mixed Appointments and Walk-Ins

**Scenario:** Several patients arrive around the same time and the appointment-eligibility + check-in ordering policy is applied.

**Actors:** Patients A, B, C, D; receptionist; doctor

**Initial Conditions:** Clinic A has an active morning session for Dr. Ravi.

**Patient/Appointment Details:**

- A: 9:00 appointment, arrives at 8:55
- B: walk-in, arrives at 9:00
- C: 9:30 appointment, arrives at 9:20
- D: walk-in, arrives at 9:10

**Actions:** Patients arrive and are checked in according to clinic flow.

**Expected Appointment State:** A and C have appointments; B and D do not

**Expected Queue/Token State:** Under the recommended MVP policy, appointment eligibility and check-in time determine order within the active session; later appointments do not automatically jump ahead of patients already waiting

**Expected Queue Result Example:** A, B, D, C if all four are checked in and evaluated by the recommended eligibility + check-in order

**Expected Patient Experience:** Booked patients expect to be ahead of walk-ins once checked in, but the exact ordering remains simple and visible.

**Expected Receptionist Experience:** The receptionist can explain the order without using a scoring algorithm.

**Expected Doctor Experience:** The doctor sees a straightforward ordered queue.

**Business Rules Exercised:** Appointment priority vs walk-in order; clinic-session queue boundaries; check-in time within group.

**Open Questions:** Exact clinic-specific eligibility-window configuration remains a future policy detail.

## Scenario 5: Early Arrival

**Scenario:** A patient has a 10:00 appointment but arrives at 9:15.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** The doctor session is active or about to open.

**Patient/Appointment Details:** Appointment exists; patient arrives well before the slot time.

**Actions:** Patient arrives early -> receptionist decides whether to check in -> token generation policy is applied -> patient waits or is held

**Expected Appointment State:** Appointment remains valid

**Expected Queue/Token State:** The patient may be checked in early, but early arrival does not automatically give them priority over patients already eligible to be served

**Expected Patient Experience:** The patient may see that they are present and may wait, but not necessarily be served before earlier checked-in patients.

**Expected Receptionist Experience:** Receptionist needs a simple rule for early arrivals, but that rule is not finalized yet.

**Expected Doctor Experience:** Doctor should not be forced into a complex priority decision for every early arrival.

**Business Rules Exercised:** Early arrival handling; check-in timing; queue ordering.

**Open Questions:** Exact clinic-specific eligibility window remains a future configuration detail.

## Scenario 6: Late Arrival and No-Show Boundary

**Scenario:** Patients arrive 5 minutes late, 15 minutes late, 30+ minutes late, or after a skipped token.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** The clinic has an active session and an existing appointment schedule.

**Patient/Appointment Details:** A booked patient arrives at different times after the scheduled appointment time.

**Actions:** Patient arrives late -> receptionist decides whether to check in, recall, skip, or mark no-show according to clinic policy

**Expected Appointment State:** The appointment may remain valid, become `SKIPPED`, or become `NO_SHOW` depending on clinic policy

**Expected Queue/Token State:** A token may never exist, may exist and be skipped, and the basic MVP flow allows one recall before reclassification

**Expected Patient Experience:** The patient may keep the appointment, lose priority, or be marked no-show depending on policy

**Expected Receptionist Experience:** The receptionist needs a configurable late-arrival rule, not a universal hard-coded threshold

**Expected Doctor Experience:** The doctor sees the operational consequence without needing a special late-arrival algorithm

**Business Rules Exercised:** Late arrival thresholds; skipped token handling; no-show distinction from cancellation

**Open Questions:** Exact clinic-specific late-arrival configuration remains a future policy detail.

## Scenario 7: Cancellation Before Arrival

**Scenario:** A patient cancels an appointment before coming to the clinic.

**Actors:** Patient, receptionist

**Initial Conditions:** Appointment exists, but the patient has not checked in.

**Patient/Appointment Details:** Future visit is booked, then cancelled.

**Actions:** Appointment is cancelled

**Expected Appointment State:** `CANCELLED`

**Expected Queue/Token State:** No token should exist if the patient never checked in; if a provisional queue record somehow exists, its removal is a separate operational action and remains policy-driven

**Expected Patient Experience:** The patient sees the appointment as cancelled.

**Expected Receptionist Experience:** The receptionist removes the appointment from the schedule.

**Expected Doctor Experience:** The doctor should not see the cancelled booking as an active patient.

**Business Rules Exercised:** Cancellation is separate from queue participation.

**Open Questions:** None for the MVP; provisional tokens before arrival are not part of the model.

## Scenario 8: Cancellation After Token Generation

**Scenario:** A patient checks in, receives a token, then decides to leave and cancel.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** The patient already has a checked-in token in an active session queue.

**Patient/Appointment Details:** Appointment exists and the queue token exists.

**Actions:** Patient leaves -> appointment is cancelled or marked according to policy -> token is cancelled or removed as a separate operational action

**Expected Appointment State:** Appointment may move to `CANCELLED`

**Expected Queue/Token State:** Token should not remain active if the patient has left; queue removal is operationally separate from appointment cancellation

**Expected Patient Experience:** The patient is no longer considered active in the clinic flow.

**Expected Receptionist Experience:** The receptionist updates both the booking record and the queue record.

**Expected Doctor Experience:** The doctor sees the patient removed from active circulation.

**Business Rules Exercised:** Appointment and token lifecycles remain independent after check-in.

**Open Questions:** Exact clinic policy for post-check-in departure remains a future operational detail.

## Scenario 9: Skipped Patient and Recall

**Scenario:** Token #12 is called, the patient does not respond, and later returns.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** Token #12 is in an active queue.

**Patient/Appointment Details:** The patient was already checked in and is part of the session queue.

**Actions:** Token is called -> patient does not respond -> token is skipped -> patient returns later -> receptionist or doctor recalls once if the clinic chooses to do so

**Expected Appointment State:** The underlying appointment may still be active or may be completed later if the patient returns and is served

**Expected Queue/Token State:** Token #12 becomes `SKIPPED`; the basic MVP flow allows one recall, after which the token may be reclassified if the patient still does not respond

**Expected Patient Experience:** The patient may be called back or may lose the turn depending on clinic policy

**Expected Receptionist Experience:** The receptionist needs a simple recall control, not a complicated retry system

**Expected Doctor Experience:** The doctor can continue the queue without a special algorithm

**Business Rules Exercised:** Skipped state, recall action, manual queue override, auditability

**Open Questions:** None for the MVP; one recall is the baseline rule.

## Scenario 10: Doctor Delay

**Scenario:** Doctor is scheduled for 9:00 AM but arrives at 9:30 AM.

**Actors:** Doctor, receptionist, patient

**Initial Conditions:** Patients are already checked in or waiting.

**Patient/Appointment Details:** Appointments exist for the session, but the doctor is delayed.

**Actions:** Doctor delay is recorded -> queue timing is adjusted -> patients are notified if supported

**Expected Appointment State:** Appointment time should not be mutated by the delay

**Expected Queue/Token State:** Queue state changes operationally; tokens may remain waiting until the doctor starts

**Expected Patient Experience:** The patient sees delay-related information and an updated wait expectation

**Expected Receptionist Experience:** Receptionist can explain the delay and manage the queue accordingly

**Expected Doctor Experience:** Doctor starts the session late without breaking the queue record

**Business Rules Exercised:** Doctor delay is an operational queue event, not an appointment rewrite.

**Open Questions:** Exact patient-facing delay notification behavior remains a future product decision.

## Scenario 11: Doctor Breaks, Temporary Unavailability, and Session End

**Scenario:** The doctor pauses for 30 minutes, then later becomes temporarily unavailable.

**Actors:** Doctor, receptionist, patient

**Initial Conditions:** A live queue already exists.

**Patient/Appointment Details:** Some patients are waiting; others may be called or in consultation.

**Actions:** Doctor takes a break -> queue is held or paused -> doctor later becomes unavailable -> clinic decides whether to resume, pause longer, or close the session

**Expected Appointment State:** Appointments remain separate from the pause/unavailability signal

**Expected Queue/Token State:** Queue may move to `HOLD` or remain paused; tokens should not be silently lost

**Expected Patient Experience:** Patients see that the doctor has paused or is unavailable and can understand the impact on waiting time

**Expected Receptionist Experience:** Receptionist can pause the active flow and communicate the change

**Expected Doctor Experience:** Doctor can pause and resume the active session without destroying the queue history

**Business Rules Exercised:** Break handling, temporary unavailability, session boundaries, estimated wait impact

**Open Questions:** Exact session-resume and session-close policy remains a future operational decision.

## Scenario 12: Multiple Doctors at One Clinic

**Scenario:** Clinic A has Dr. Ravi and Dr. Priya seeing patients at the same time.

**Actors:** Two doctors, receptionist, multiple patients

**Initial Conditions:** Both doctors have active sessions at the same clinic.

**Patient/Appointment Details:** Some patients are booked with Dr. Ravi; others are booked with Dr. Priya.

**Actions:** Each doctor runs their own queue.

**Expected Appointment State:** Appointments remain doctor-specific and clinic-specific

**Expected Queue/Token State:** Dr. Ravi's tokens never affect Dr. Priya's queue, and vice versa

**Expected Patient Experience:** The patient sees the correct doctor-specific queue

**Expected Receptionist Experience:** Receptionist manages multiple queues independently

**Expected Doctor Experience:** Each doctor sees only their own session queue

**Business Rules Exercised:** Independent queues per doctor session within the same clinic.

**Open Questions:** Exact clinic-level dashboard presentation remains a future UI decision.

## Scenario 13: Doctor at Multiple Clinics and Morning/Evening Sessions

**Scenario:** Dr. Ravi works at Clinic A in the morning and Clinic B in the afternoon, then returns for an evening session at Clinic A.

**Actors:** Doctor, receptionist, patients at two clinics

**Initial Conditions:** Separate clinic sessions exist for each practice period.

**Patient/Appointment Details:** Bookings are made for the correct clinic and session.

**Actions:** Patients are checked in at each clinic session; tokens are generated within the correct session.

**Expected Appointment State:** Appointments remain clinic-specific

**Expected Queue/Token State:** Tokens never cross clinic boundaries; morning and evening sessions keep separate queues and token numbering

**Expected Patient Experience:** The patient sees the correct clinic and session context

**Expected Receptionist Experience:** Receptionist at each clinic works only within that clinic's queue

**Expected Doctor Experience:** Doctor moves between clinic sessions without mixing records

**Business Rules Exercised:** Multi-clinic doctor support, session boundaries, token numbering per session.

**Open Questions:** Exact human-readable prefix or display format remains a future presentation detail.

## Scenario 14: Patient Queue Visibility

**Scenario:** A patient with token #27 wants to know their queue status.

**Actors:** Patient, clinic system

**Initial Conditions:** The patient is checked in and waiting in an active session.

**Patient/Appointment Details:** Token #27 belongs to a specific doctor session.

**Actions:** Patient views queue status.

**Expected Appointment State:** Appointment remains separate from live queue state

**Expected Queue/Token State:** The patient can see their own token, current serving token, number ahead, and estimated wait

**Expected Patient Experience:** Example display:

- Your token: #27
- Currently serving: #21
- Patients ahead: 5
- Estimated wait: 35-45 min
- Doctor status: Consulting

**Expected Receptionist Experience:** Receptionist can answer queue questions without revealing unrelated private details

**Expected Doctor Experience:** Doctor sees the operational queue state needed for calling the next patient

**Business Rules Exercised:** Patient visibility, limited exposure of other patients' information, estimated waiting information.

**Open Questions:** Exact additional patient-visible fields remain a future product decision.

## Scenario 15: Receptionist Manual Override

**Scenario:** The receptionist needs to move, skip, hold, or recall a patient.

**Actors:** Receptionist, doctor, patient

**Initial Conditions:** A live queue exists with several waiting patients.

**Patient/Appointment Details:** One or more patients have active tokens.

**Actions:** Receptionist moves a patient forward, backward, skips, holds, or recalls them

**Expected Appointment State:** Appointment records do not change just because queue order changed

**Expected Queue/Token State:** Token order changes operationally; the action should be auditable

**Expected Patient Experience:** The patient may be notified depending on the action

**Expected Receptionist Experience:** Receptionist can manage the clinic flow with a limited override control

**Expected Doctor Experience:** Doctor sees the queue order change in a controlled way

**Business Rules Exercised:** Manual override, audit trail, role-based queue control.

**Open Questions:** None for the MVP; doctors and receptionists can both perform controlled manual overrides.

## Scenario 16: Emergency or Priority Case

**Scenario:** An urgent patient arrives and needs to be seen sooner than the normal order.

**Actors:** Receptionist, doctor, patient

**Initial Conditions:** A normal queue is already active.

**Patient/Appointment Details:** The urgent case is exceptional and not part of a general triage system.

**Actions:** Clinic staff place the patient ahead manually or mark them as an exception

**Expected Appointment State:** The appointment record, if any, remains separate from the exception handling

**Expected Queue/Token State:** The queue may be manually adjusted, but advanced triage rules are not defined in MVP

**Expected Patient Experience:** The urgent patient is handled operationally without a complex algorithm

**Expected Receptionist Experience:** Receptionist needs a simple manual override path

**Expected Doctor Experience:** Doctor can see the patient sooner without the system pretending to solve triage

**Business Rules Exercised:** Exceptional handling without advanced prioritization.

**Open Questions:** None for the MVP; emergency handling is covered by manual override.

## Scenario 17: Patient Leaves Clinic

**Scenario:** A patient waits for some time and leaves without informing the clinic.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** The patient has an active token and is waiting.

**Patient/Appointment Details:** Appointment or token exists in the active session.

**Actions:** Patient leaves -> receptionist notices later or is informed -> token is handled according to policy

**Expected Appointment State:** Appointment may remain booked, become no-show, or be cancelled according to clinic policy

**Expected Queue/Token State:** The token may become skipped, no-show, or cancelled as an operational action

**Expected Patient Experience:** The patient is no longer considered active in the queue

**Expected Receptionist Experience:** The receptionist updates the queue record

**Expected Doctor Experience:** The doctor stops expecting the patient in the active order

**Business Rules Exercised:** Patient abandonment, queue removal, no-show distinction.

**Open Questions:** Exact timeout rule for leaving the clinic remains a future clinic-policy configuration detail.

## Scenario 18: Duplicate or Conflicting Bookings

**Scenario:** A patient books two appointments with the same doctor session, or the receptionist accidentally creates a duplicate booking.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** The clinic session already has at least one booking for the patient.

**Patient/Appointment Details:** Duplicate or overlapping bookings exist conceptually.

**Actions:** A second appointment is created or detected

**Expected Appointment State:** The system should surface the conflict as a business rule problem rather than silently allowing confusion

**Expected Queue/Token State:** Queue should not automatically duplicate the same patient into the active line without a clear policy

**Expected Patient Experience:** Patient should not be expected to attend two overlapping visits at the same time

**Expected Receptionist Experience:** Receptionist should be warned about duplicates

**Expected Doctor Experience:** Doctor should not see conflicting duplicate visits as separate normal patients

**Business Rules Exercised:** Duplicate booking prevention, conflict visibility, operational sanity.

**Open Questions:** Exact duplicate-booking prevention mechanism and staff exception workflow remain future design details.

## Scenario 19: Queue Failure and Operational Edge Cases

**Scenario:** The clinic experiences operational mistakes or concurrency problems.

**Actors:** Receptionist, doctor, patient

**Initial Conditions:** A live queue is active.

**Patient/Appointment Details:** Existing appointments and tokens are present.

**Actions:** Possible edge cases include:

- receptionist generates a token twice
- patient checks in twice
- appointment is cancelled after check-in
- token is accidentally skipped
- two receptionists act at the same time
- doctor closes the session while patients remain

**Expected Appointment State:** Appointment records remain distinct from operational queue mistakes

**Expected Queue/Token State:** The queue should preserve a coherent operational history even when human mistakes happen

**Expected Patient Experience:** Patients should not be silently lost or duplicated

**Expected Receptionist Experience:** Receptionist needs clear rules for repairing queue mistakes

**Expected Doctor Experience:** Doctor should not be forced to reason through contradictory queue state

**Business Rules Exercised:** Operational error handling, auditability, session closing rules, duplicate token protection.

**Open Questions:** Technical concurrency resolution is out of scope; business invariants are captured by the current queue policy.

## Scenario 20: Wait-Time Estimation Examples

**Scenario:** The clinic wants to estimate waiting time for patients in the active queue.

**Actors:** Patient, receptionist, doctor

**Initial Conditions:** Several patients are waiting; the doctor has an observed consultation pattern.

**Patient/Appointment Details:** Some patients are completed, some are waiting, and the doctor may be delayed or on break.

**Actions:** The system uses queue position, current serving patient, delay, and average consultation duration to estimate waiting time

**Expected Appointment State:** Appointment lifecycle is not enough by itself to calculate wait time

**Expected Queue/Token State:** Wait time depends on queue state, doctor delay, break status, and completed consultations

**Expected Patient Experience:** Example outputs could be:

- 3 patients ahead, average consultation 12 minutes, doctor currently consulting, estimated wait 25-35 minutes
- 5 patients ahead, doctor delayed 20 minutes, estimated wait increases accordingly
- doctor on break, estimate should reflect pause rather than progress

**Expected Receptionist Experience:** Receptionist can explain the estimate in simple terms

**Expected Doctor Experience:** Doctor can see how delays affect queue pressure

**Business Rules Exercised:** Wait-time estimation inputs, operational queue state, delay effect.

**Open Questions:** Exact wait-time algorithm remains intentionally undefined for MVP implementation.

## Business Rule Conflicts

The scenarios expose these remaining future-design areas:

- Exact appointment eligibility window
- Exact clinic-specific configuration mechanism for late arrivals
- Exact patient-visibility fields beyond the core MVP set
- Exact clinic-policy timeout for leaving the clinic
- Future duplicate-booking resolution workflow details
- Future presentation details for session-based numbering

## Recommended MVP Policy

### DECIDED

- Appointment and queue token are separate concepts
- Queue token is operational, not booking-only
- A walk-in may have a token without an appointment
- A patient may have an appointment without a token if they have not arrived
- Queue must remain session-specific
- Doctor-session-based queue with clinic context is the MVP model
- Token is created at check-in or arrival
- Appointment eligibility plus actual check-in time determine order
- Early arrival does not automatically create priority
- Token numbering should reset per session
- Late arrival uses a 15-minute initial grace period
- `RECALLED` should remain an action, not a lifecycle state
- One recall is allowed in the basic MVP flow
- Manual queue overrides require auditability
- Receptionists and doctors can perform controlled manual overrides
- Manual overrides require a reason
- Emergency and priority handling use manual override rather than a separate triage system
- Appointment type does not add queue priority
- Conflicting appointments should be prevented or surfaced

### FUTURE

- Exact appointment eligibility window
- Exact clinic-specific configuration mechanism for the late-arrival grace period
- Exact patient-visibility fields beyond the core MVP set
- Exact clinic-policy timeout for leaving the clinic
- Future duplicate-booking resolution workflow details
- Future presentation details for session-based numbering

### OPEN

- None in the core MVP queue rules

## Requirements Derived from Scenarios

The future database and API design must support:

- appointment without token
- token without appointment
- doctor-session-specific queue
- multiple clinics per doctor
- multiple doctors per clinic
- queue token lifecycle
- appointment lifecycle
- manual queue actions
- auditability
- doctor delay
- breaks and temporary unavailability
- patient queue visibility
- no-show handling
- cancellation handling
- duplicate booking detection or surfacing
- wait-time inputs and outputs
- session start, pause, resume, and end

## Documentation Impact

Recommended documentation follow-up after this scenario review:

1. Keep `docs/13-queue-policy.md` aligned with `docs/decisions/ADR-001-mvp-queue-rules.md`
2. Use this scenario set as the validation reference before database design
3. Expand `docs/04-user-flows.md` only if the team later decides to show more operational edge cases in the main flows
4. Update `docs/05-mvp-scope.md` only if wait-time visibility or manual override scope changes

## Summary

1. File created: `docs/14-queue-scenarios.md`
2. Number of scenarios: 20
3. Most important business-rule findings: queue policy must stay session-based, check-in drives token creation, and appointment state cannot stand in for queue state
4. Weaknesses in the current queue policy: the remaining gray areas are mostly configuration and presentation details, not core queue behavior
5. Decisions needed before database design: exact eligibility-window handling, exact late-arrival configuration, exact patient visibility, and duplicate-booking workflow detail
6. Remaining OPEN questions: none in the core MVP queue rules
7. Recommended next documentation task: validate the exact clinic-specific configuration shape before database design
