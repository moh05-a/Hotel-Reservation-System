
# Group 15 — Hotel Reservation System
## Incremental Development Plan

| Version | Type | Capability / Scope | Deliverable / Evidence | Requirements / Design Links |
|---|---|---|---|---|
| `v0.1.0-design` | Design | Foundation; guest/staff authentication and roles | Architecture, C4 context/container, use cases, login flow | `FR-AU-01–03`; `docs/architecture.md`, `uml/seq_auth.puml` |
| `v0.1.0` | Working | Runnable application, database and role-based login | Working login; access tests | `FR-AU-01–03`; auth modules/tests |
| `v0.2.0-design` | Design | Room types, physical rooms, rates and capacity | Class diagram, ERD, room rules | `FR-RM-01–05`; `uml/class_domain.puml`, `uml/erd_hotel.drawio` |
| `v0.2.0` | Working | Room and rate management | Staff interface/API; validation tests | `FR-RM-01–05`; room modules/tests |
| `v0.3.0-design` | Design | Availability by dates and guest count | Search sequence; inventory rules | `FR-AV-01–03`; `uml/seq_availability.puml` |
| `v0.3.0` | Working | Search available room types | Search interface/API; availability tests | `FR-AV-01–03`; availability modules/tests |
| `v0.4.0-design` | Design | Guest records and access | Guest data model and rules | `FR-GS-01–04`; class diagram, ERD |
| `v0.4.0` | Working | Guest record management | Guest interface/API; persistence tests | `FR-GS-01–04`; guest modules/tests |
| `v0.5.0-design` | Design | Create room-type reservations | Booking sequence; reservation states | `FR-RS-01/03/04`; `uml/seq_booking.puml`, `uml/state_reservation.puml` |
| `v0.5.0` | Working | Create and confirm reservations | Booking interface/API; overbooking tests | `FR-RS-01/03/04`; reservation modules/tests |
| `v0.6.0-design` | Design | View, modify and cancel reservations | Ownership and cancellation rules; state updates | `FR-RS-02/03`; reservation state diagram |
| `v0.6.0` | Working | Guest/staff booking management | Change/cancel interface/API; authorization tests | `FR-RS-02/03`; booking modules/tests |
| `v0.7.0-design` | Design | Room assignment, check-in/out | Front-desk sequence/activity diagrams | `FR-CI-01–04`; `uml/seq_checkin_checkout.puml` |
| `v0.7.0` | Working | Assign physical room at check-in; check out | Staff interface/API; assignment tests | `FR-CI-01–04`; front-desk modules/tests |
| `v0.8.0-design` | Design | Billing, tax, invoices and simulated payments | Billing model and payment sequence | `FR-BL-01–05`; `uml/seq_billing.puml`, ERD |
| `v0.8.0` | Working | Generate bills and record simulated payments | Invoice/payment interface; calculation tests | `FR-BL-01–05`; billing modules/tests |
| `v0.9.0-design` | Design | Housekeeping and room operational status | State diagram; status rules and change history | `FR-OP-01–03`; `uml/state_room.puml` |
| `v0.9.0` | Working | Manage cleaning and out-of-order status | Status UI/history; assignment tests | `FR-OP-01–03`; housekeeping modules/tests |
| `v0.10.0-design` | Design | Operations dashboard | Metrics, filters and dashboard interaction design | `FR-OP-04`; `uml/seq_dashboard.puml` |
| `v0.10.0` | Working | Show occupancy, arrivals/departures, revenue and room readiness | Filtered dashboard; metric tests | `FR-OP-04`; dashboard modules/tests |
| `v1.0.0` | Final | Complete integrated system | End-to-end tests, demo, final diagrams and report | All requirements; `docs/traceability_matrix.md` |

## Questions, Tools and Acceptance Criteria

### Increment 1: Foundation and Access — Anas Alseddah

- **v0.1.0-design:** Question — Who can access each feature? **Tools —** Markdown, C4/UML. **Accept —** Architecture and role rules reviewed.
- **v0.1.0:** Question — Can users log in and run the system? **Tools —** Agentic AI, database, tests. **Accept —** Reproducible startup; permitted/denied access verified.

### Increment 2: Room Management — Anas Alseddah

- **v0.2.0-design:** Question — How are rooms, types, rates and capacity represented? **Tools —** Class diagram, ERD. **Accept —** Data model and validation rules reviewed.
- **v0.2.0:** Question — Can staff manage rooms and rates? **Tools —** Agentic AI, API/UI tests. **Accept —** Valid changes succeed; invalid changes fail.

### Increment 3: Availability Search — Anas Alseddah

- **v0.3.0-design:** Question — How is room-type availability calculated? **Tools —** Sequence diagram, scenarios. **Accept —** Date overlap, guest capacity and inventory rules defined.
- **v0.3.0:** Question — Does search show genuinely available types? **Tools —** Agentic AI, integration tests. **Accept —** Full inventory and invalid-date cases handled.

### Increment 4: Guest Records — Yousef Taha

- **v0.4.0-design:** Question — What guest data and access rules are needed? **Tools —** Class diagram, ERD. **Accept —** Data fields and permissions reviewed.
- **v0.4.0:** Question — Can records be stored and retrieved correctly? **Tools —** Agentic AI, database tests. **Accept —** Persistence and access checks pass.

### Increment 5: Reservation Creation — Yousef Taha

- **v0.5.0-design:** Question — How are room types reserved without assigning room numbers? **Tools —** Sequence/state diagrams. **Accept —** Booking rules and states reviewed.
- **v0.5.0:** Question — Can reservations be created without overbooking? **Tools —** Agentic AI, integration tests. **Accept —** Bookings persist; sold-out and concurrent requests handled.

### Increment 6: Booking Changes — Yousef Taha

- **v0.6.0-design:** Question — Who can change bookings, and under what rules? **Tools —** Requirements, state diagram. **Accept —** Ownership, cancellation and availability recheck defined.
- **v0.6.0:** Question — Do changes preserve correct availability? **Tools —** Agentic AI, tests. **Accept —** Authorized changes pass; cancellations release inventory.

### Increment 7: Check-in / Check-out — Mohammad Ameerah

- **v0.7.0-design:** Question — How are rooms assigned and stays completed? **Tools —** Sequence/activity diagrams. **Accept —** Assignment, readiness and stay-state rules reviewed.
- **v0.7.0:** Question — Can staff check guests in and out? **Tools —** Agentic AI, integration tests. **Accept —** Room assigned only at check-in; conflicts rejected.

### Increment 8: Billing and Payments — Mohammad Ameerah

- **v0.8.0-design:** Question — How are charges and payments calculated? **Tools —** Data/sequence diagrams. **Accept —** Tax, invoice and simple refund rules reviewed.
- **v0.8.0:** Question — Are bills and simulated payments correct? **Tools —** Agentic AI, tests. **Accept —** Totals, payments and applicable refunds verified.

### Increment 9: Housekeeping — Zein A. Sunoqrot

- **v0.9.0-design:** Question — How do room states affect operations? **Tools —** State diagram, data model. **Accept —** Allowed transitions, history fields and assignment restrictions defined.
- **v0.9.0:** Question — Are room updates tracked and unsafe assignments blocked? **Tools —** Agentic AI, integration tests. **Accept —** Status history saved; invalid transitions and out-of-order assignments rejected.

### Increment 10: Operations Dashboard — Zein A. Sunoqrot

- **v0.10.0-design:** Question — How are hotel metrics calculated? **Tools —** Architecture/sequence diagram. **Accept —** Date filters, metrics and data sources defined.
- **v0.10.0:** Question — Can managers monitor stays, revenue and room readiness? **Tools —** Agentic AI, UI/database tests. **Accept —** Filtered figures match stored records.

### Final Release: v1.0.0 — All four members

**Question —** Does the integrated hotel system meet its requirements? **Tools —** Agentic AI, end-to-end tests, GitHub. **Accept —** Main flows pass; documentation and traceability are complete; final release is tagged.

*Requirement IDs and diagram paths are planned references; they will be finalized and linked as the related artifacts are created.*
