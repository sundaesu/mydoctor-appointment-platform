Status: Product Definition / Pre-MVP
Version: 0.1

# Doctor Appointment Platform

Doctor Appointment Platform is a healthcare appointment and clinic queue platform for India, starting in Srikakulam, Andhra Pradesh.

## Problem

Patients and clinics still rely on phone calls, WhatsApp, walk-ins, paper tokens, and manual appointment books. That makes availability, queue position, and waiting time hard to track.

## Vision

Build a scalable healthcare appointment and clinic queue platform for India, beginning with independent doctors and clinics in smaller cities and expanding city-by-city.

## Current Phase

Phase 0: Problem validation and product definition.

## Product Views

- PatientView
- DoctorView
- ReceptionistView

## MVP

- Doctor discovery and search
- Appointment booking
- Token management
- Real-time queue management
- Clinic workflow support for receptionists and doctors

## Architecture Direction

- Start with a modular monolith
- Keep backend modules clearly bounded
- Treat PostgreSQL as the source of truth
- Use Redis and real-time communication only as proposed directions until implementation design is finalized

## Repository Structure

- `docs/` product and architecture source of truth
- `backend/` future backend implementation
- `patient-app/` future patient-facing app
- `doctor-app/` future doctor-facing app
- `clinic-web/` future receptionist and clinic web app

## Documentation

- [Product vision](docs/01-product-vision.md)
- [Features](docs/02-features.md)
- [User roles](docs/03-user-roles.md)
- [User flows](docs/04-user-flows.md)
- [MVP scope](docs/05-mvp-scope.md)
- [Architecture](docs/06-architecture.md)
- [Database design placeholder](docs/07-database-design.md)
- [API design placeholder](docs/08-api-design.md)
- [Security](docs/09-security.md)
- [Notifications](docs/10-notifications.md)
- [Roadmap](docs/11-roadmap.md)
- [Decision log](docs/decisions/README.md)

## Development Roadmap

1. Phase 0 - Problem validation
2. Phase 1 - Srikakulam MVP
3. Phase 2 - Product-market validation in Srikakulam
4. Phase 3 - Srikakulam district expansion
5. Phase 4 - Expansion to Tier-2/Tier-3 Andhra Pradesh cities
6. Phase 5 - Andhra Pradesh network
7. Phase 6 - Patient healthcare platform
8. Phase 7 - Clinic management SaaS
9. Phase 8 - Multi-state expansion
10. Phase 9 - Broader healthcare ecosystem

## Current Status

This repository is intentionally documentation-first. Product, architecture, and implementation decisions should be captured in `docs/` before code is introduced.
