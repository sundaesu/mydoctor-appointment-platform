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
→ Receive token where applicable
→ Track queue
→ Visit clinic
```

## Workflow 2 - Patient Books Through Receptionist

```text
Patient calls clinic
→ Receptionist searches/registers patient
→ Receptionist selects doctor
→ Receptionist books appointment
→ System creates appointment
→ Patient receives confirmation
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
→ Appointment/token completed
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

## Flow Notes

These flows describe the initial operational model and may be refined after validation in the Srikakulam pilot.
