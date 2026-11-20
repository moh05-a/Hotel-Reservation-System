# Group 15 — Hotel Reservation System
## Incremental Development Plan

This plan uses the requirement IDs in `docs/requirements.md` and the account-free guest-booking model in `docs/project_description.md`. Each capability has a **design checkpoint** (`-design`) followed by a **tested working checkpoint**. Design/source paths below are planned repository paths and must be updated if file names change.

**Team ownership:** Anas Alseddah — Increments 1–3; Yousef Taha — 4–6; Mohammad Ameerah — 7–8; Zein A. Sunoqrot — 9–10. All participate in final integration.

| Version | Type | Capability / Scope | Deliverable / Evidence | Requirements / Design Links |
|---|---|---|---|---|
| `v0.1.0-design` | Design | Foundation; **staff** authentication and roles (**no guest accounts**) | C4 context/container, architecture and staff-access design | `FR-AU-01–03`, `NFR-03`, `NFR-06`; `docs/architecture.md`, `uml/seq_auth.puml` |
| `v0.1.0` | Working | Runnable web/backend/database; staff login/logout and role checks | Setup instructions, auth interface/API and tests | `FR-AU-01–03`, `NFR-03`, `NFR-06`; auth modules/tests |
| `v0.2.0-design` | Design | Room types, physical rooms, capacity and rates | Class diagram, ERD, validation and room-state rules | `FR-RM-01–05`; `uml/class_domain.puml`, `uml/erd_hotel.drawio` |
| `v0.2.0` | Working | Room and rate management | Manager UI/API and CRUD/validation tests | `FR-RM-01–05`; room modules/tests |
| `v0.3.0-design` | Design | Availability including unexpired Pending holds | Search sequence, date, expiry-aware hold and room rules | `FR-AV-01–03`, `NFR-02`; `uml/seq_availability.puml` |
| `v0.3.0` | Working | Room-type availability accounting for active Pending holds | Search UI/API, availability/expiry tests | `FR-AV-01–03`, `NFR-02`; availability modules/tests |
| `v0.4.0-design` | Design | **Guest records without login accounts**; returning-guest matching and staff history | Guest model, ERD and identity-linking rules | `FR-GS-01–04`; `uml/class_domain.puml`, `uml/erd_hotel.drawio` |
| `v0.4.0` | Working | Guest profiles, authorized search and linked stay history | Guest UI/API, persistence, linking and access tests | `FR-GS-01–04`; guest modules/tests |
| `v0.5.0-design` | Design | Account-free booking with five-minute Pending hold and mandatory simulated payment | Sequence, state, rules and concurrency design | `FR-RS-01–03`, `NFR-01`, `NFR-03`; `docs/booking_rules.md`, `uml/seq_booking.puml`, `uml/state_reservation.puml` |
| `v0.5.0` | Working | Confirm after successful simulated payment; expire holds and issue reference/secret | Booking UI/API, five-minute hold, retry, expiry, code-hash and concurrency tests | `FR-RS-01–03`, `NFR-01`, `NFR-03`; booking modules/tests |
| `v0.6.0-design` | Design | Secure single-booking access, modification and cancellation with 24-hour refund policy | Access, state, date-change and refund rules | `FR-RS-04–06`, `NFR-03`; `docs/booking_rules.md`, `uml/state_reservation.puml` |
| `v0.6.0` | Working | Secure view, capacity-safe modifications, cancellation and refund | Wrong-code, 24-hour cutoff, availability and refund tests | `FR-RS-04–06`, `NFR-03`; booking modules/tests |
| `v0.7.0-design` | Design | **Manual simulated identity check**, room assignment, check-in/out | Front-desk sequence/activity diagrams, verification/readiness rules | `FR-CI-01–04`; `uml/seq_checkin_checkout.puml`, `uml/activity_checkin_checkout.puml` |
| `v0.7.0` | Working | ID-check confirmation, physical-room assignment and check-in/out | Staff UI/API; identity, readiness and stay tests | `FR-CI-01–04`, `NFR-01`; front-desk modules/tests |
| `v0.8.0-design` | Design | Folios, invoices, JOD tax, simulated payments/reversals/refunds | Billing models, payment sequence, currency/rounding and coordination with mandatory booking payment | `FR-BL-01–05`; `uml/seq_billing.puml`, `uml/erd_hotel.drawio` |
| `v0.8.0` | Working | Itemized billing and simulated payment/refund/reversal records | Invoices, arithmetic, reconciliation and refund tests | `FR-BL-01–05`; billing modules/tests |
| `v0.9.0-design` | Design | Housekeeping and physical-room operational statuses | State diagram, change-history, readiness rules | `FR-OP-01–03`; `uml/state_room.puml` |
| `v0.9.0` | Working | Cleaning and Out-of-Order management | Status UI/API, history and assignment-block tests | `FR-OP-01–03`; housekeeping modules/tests |
| `v0.10.0-design` | Design | Manager dashboard metrics and filters | Metric definitions, sources and dashboard sequence | `FR-OP-04`; `uml/seq_dashboard.puml` |
| `v0.10.0` | Working | Occupancy, arrivals/departures, revenue, readiness | Dashboard UI/API and metric tests | `FR-OP-04`; dashboard modules/tests |
| `v1.0.0` | Final | Complete integrated web application | End-to-end tests, final diagrams, AI log, traceability, README and PDF report | **All FRs and NFRs**; `docs/traceability_matrix.md` |

## Questions, Tools and Acceptance Criteria

### Increment 1: Foundation and Staff Access — Anas Alseddah

- **v0.1.0-design:** **Question:** What are the system's main runtime parts and staff permissions? **Tools:** Markdown, C4, UML. **Accept:** Architecture, RBAC matrix, and staff-login flow are reviewed; guest accounts are explicitly out of scope.
- **v0.1.0:** **Question:** Can the application run and enforce staff roles? **Tools:** Agentic AI, selected frontend/backend/database, pytest. **Accept:** Fresh-clone startup succeeds, staff login/logout works, and direct unauthorized calls are rejected.

### Increment 2: Room Management — Anas Alseddah

- **v0.2.0-design:** **Question:** How are room types, physical rooms, rates, and capacity modeled? **Tools:** Class Diagram, ERD. **Accept:** Schema, uniqueness, permission and rate-validation rules reviewed with guest/housekeeping owners.
- **v0.2.0:** **Question:** Can authorized hotel managers manage inventory and rates? **Tools:** Agentic AI, API/UI tests. **Accept:** Room CRUD works; invalid capacities, negative rates, duplicate room numbers and unsafe deactivation fail.

### Increment 3: Availability Search — Anas Alseddah

- **v0.3.0-design:** **Question:** How many rooms are available per night, including active Pending holds? **Tools:** Sequence Diagram, scenarios. **Accept:** Date overlaps, expiry, checkout-exclusive and Out-of-Order assumptions reviewed.
- **v0.3.0:** **Question:** Do searches return truly available room types? **Tools:** Agentic AI, integration/performance tests. **Accept:** Sold-out, partial-overlap, adjacent-date, active/expired holds and invalid-input cases pass.

### Increment 4: Guest Records and History — Yousef Taha

- **v0.4.0-design:** **Question:** How do we identify returning guests without guest accounts? **Tools:** ERD, Class Diagram, guest-record rules. **Accept:** Persistent guest ID, authorized history view, suggested-match and manual link verification agreed.
- **v0.4.0:** **Question:** Can authorized staff manage and search guest records safely? **Tools:** Agentic AI, database/access tests. **Accept:** Guest profile, search and linked multi-stay history work; contacts/names alone never automatically merge people.

### Increment 5: Reservation Creation — Yousef Taha

- **v0.5.0-design:** **Question:** How do five-minute holds, mandatory simulated payment and access credentials work without guest registration? **Tools:** Sequence/State Diagrams, security and inventory rules. **Accept:** Pending, expiry/retries, confirmation, hashing and reversal paths reviewed.
- **v0.5.0:** **Question:** Can payment confirm exactly one booking without overbooking? **Tools:** Agentic AI, concurrency/integration/security tests. **Accept:** Five-minute holds, original deadline on retries, one-time reference/code, successful payments and simultaneous-last-room cases pass.

### Increment 6: Booking Viewing and Changes — Yousef Taha

- **v0.6.0-design:** **Question:** What booking access and cancellation/modification/refund rules apply? **Tools:** Requirements, state and access rules. **Accept:** Per-booking scope, 24-hour cutoff, NoShow, payment differences and refunds documented.
- **v0.6.0:** **Question:** Can verified guests manage one eligible booking safely? **Tools:** Agentic AI, authorization/integration tests. **Accept:** Wrong codes fail; date changes preserve inventory, and cutoff/refund rules pass.

### Increment 7: Check-in / Check-out — Mohammad Ameerah

- **v0.7.0-design:** **Question:** How does a receptionist verify the arriving person and safely assign a room? **Tools:** Sequence/Activity Diagrams, room-readiness and identity-check rules. **Accept:** Simulated manual verification, room assignment and stay-state workflow documented without ID-scanning integration.
- **v0.7.0:** **Question:** Can verified arrivals be checked in and active stays checked out? **Tools:** Agentic AI, integration tests. **Accept:** Check-in blocked without staff-recorded verification or a suitable room; check-out records stay/room updates.

### Increment 8: Billing and Payments — Mohammad Ameerah

- **v0.8.0-design:** **Question:** How are taxes, extras, mandatory booking payments, reversals and refunds recorded? **Tools:** Data/Sequence Diagrams. **Accept:** JOD, configurable tax, pricing and simulated transaction rules reviewed without real cards.
- **v0.8.0:** **Question:** Are folios and simulated transactions consistent? **Tools:** Agentic AI, arithmetic/API tests. **Accept:** Correct invoices, payments, reversals and refunds; mandatory booking payment is supported.

### Increment 9: Housekeeping — Zein A. Sunoqrot

- **v0.9.0-design:** **Question:** Which room conditions can staff set and when is assignment permitted? **Tools:** Room State Diagram, data model. **Accept:** Status transitions, change-history and readiness rules reviewed with Anas and Mohammad.
- **v0.9.0:** **Question:** Are housekeeping updates and assignment restrictions enforced? **Tools:** Agentic AI, integration tests. **Accept:** Status changes are logged and invalid transitions/Out-of-Order assignments rejected.

### Increment 10: Operations Dashboard — Zein A. Sunoqrot

- **v0.10.0-design:** **Question:** How are occupancy, expected arrivals/departures, revenue and readiness calculated? **Tools:** Architecture/Sequence Diagram, metric definitions. **Accept:** Metrics, time period and source tables agreed.
- **v0.10.0:** **Question:** Does the dashboard accurately summarize hotel data? **Tools:** Agentic AI, UI/database tests. **Accept:** Filtered figures match stored reservations, stays, payments and rooms.

### Final Release: v1.0.0 — All four members

**Question:** Does the full Hotel Reservation System meet its requirements? **Tools:** Agentic AI, end-to-end tests, GitHub and UML tools. **Accept:** All major flows work through the web UI; docs, tests, AI-development log, traceability and final report are complete; `v1.0.0` tags the reviewed/merged release.

## Dependencies and Parallel Work

- **All members may begin design immediately.** Increment 7 can be designed and tested against mock confirmed reservations; Increment 9 can be designed against a proposed Room model.
- **Foundation first for integration:** Increment 2 requires the database and staff roles from 1; Increment 4 needs the shared guest schema; Increment 3 uses room types from 2.
- **Booking workflow:** Increment 5 integrates guest records (4), availability (3) and a basic mandatory simulated payment. Increment 6 builds on confirmed bookings, reference/code access and 24-hour cancellation/modification policies.
- **Front desk:** Increment 7 depends on staff authorization, room records, guest information and confirmed reservations; it can be developed on its own branch before those integrations are ready.
- **Billing/housekeeping/dashboard:** Increment 8 expands the Increment 5 payment simulation with invoices, refunds, reversals and reconciliation. Increment 9 relies on physical rooms; Increment 10 can begin with sample metrics.
- **Version tags:** Each `-design` and working tag identifies a **reviewed and merged** repository checkpoint. Maintain pull requests, branches, tests, `/prompts/` records, `docs/ai_development_log.md`, and `docs/traceability_matrix.md`; never delete merged branches.

*Requirement IDs and diagram paths are planned references. Update the paths as editable diagrams and implementation modules are added to the repository.*
