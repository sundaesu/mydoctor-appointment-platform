# User Flows

## Workflow 1 - Patient Books Appointment

```text
Patient
→ Search doctor
→ Select doctor
→ Select clinic
→ Select date/time
→ Confirm appointment
→ Receive confirmation
→ Visit clinic
→ Arrival / check-in
→ Receive queue token where applicable
→ Track queue
→ Consultation
```

## Workflow 2 - Patient Books Through Receptionist

```text
Patient calls clinic
→ Receptionist searches/registers patient
→ Receptionist selects doctor
→ Receptionist books appointment
→ System creates appointment
→ Patient receives confirmation
→ Patient arrives / checks in
→ Receptionist generates queue token where applicable
→ Consultation
```

## Workflow 3 - Walk-In Patient

```text
Patient arrives at clinic
→ Receptionist searches/registers patient
→ Selects doctor
→ Generates token
→ Patient waits
→ Doctor calls next
→ Consultation
```

## Workflow 4 - Doctor Manages Queue

```text
Doctor starts clinic
→ Views today's queue
→ Calls next patient
→ Consultation
→ Completes patient
→ Calls next patient
```

## Workflow 5 - Doctor Delay

```text
Doctor becomes delayed
→ Doctor/receptionist updates availability/status
→ Queue timing is recalculated
→ Patients are notified
```

## Workflow 6 - Appointment Cancellation

```text
Appointment exists
→ Appointment is cancelled
→ If a queue token exists, token cancellation or queue removal is handled as a separate operational action
```

## Workflow 7 - Patient No-Show

```text
Patient does not arrive
→ Appointment may be marked no-show according to clinic policy
→ If a queue token exists, it may also be marked no-show or removed as a separate operational action
```

## Flow Notes

These flows describe the initial operational model and may be refined after validation in the Srikakulam pilot.
