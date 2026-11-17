# Hotel Reservation System — Requirements Specification

**Group:** 15  
**Course:** CS13477 Software Engineering — Fall 2026  
**Document:** `docs/requirements.md`  
**Status:** Complete first draft for team review (business policies marked **TBD** are not yet approved)

## 1. Purpose

This document defines the functional and non-functional requirements for Group 15's Hotel Reservation System. It is based on the updated four-member project plan and the previously agreed project description. Requirement IDs follow the ranges in the updated plan so they can be linked to development increments, UML diagrams, implementation modules, and tests.

## 2. System Overview and Scope

The system is a web application for **one hotel**. Guests can find suitable room types and create or manage their reservations. Hotel staff can manage room inventory, guest records, check-in/check-out, billing, housekeeping, and operational information.

### 2.1 In scope

- Staff authentication and role-based access control.
- Room types, physical rooms, guest capacity, and nightly rates.
- Room availability searches for a selected stay and guest count.
- Guest records and reservation history.
- Reservation creation, confirmation, search, modification, and cancellation.
- Physical-room assignment during check-in and check-out processing.
- Guest folios, additional charges, tax, invoices, simulated payments, and simulated refunds.
- Cleaning and operational room statuses, plus a basic management dashboard.

### 2.2 Out of scope

- Multiple hotels or a multi-property booking platform.
- Real card processing or storage of real payment-card details.
- Live integrations with external booking services.
- Real SMS or email delivery; any notification events are recorded locally if implemented.

## 3. Actors and Access Responsibilities

| Actor | Main responsibilities |
|---|---|
| **Guest** | Search room types; create, view, modify, and cancel **their own** reservations. |
| **Receptionist** | Manage guest records and reservations; assign rooms; check guests in/out; handle billing. |
| **Hotel Manager** | Manage room types, physical rooms, rates, and view the operations dashboard. |
| **Housekeeping Staff** | View and update permitted cleaning/operational room statuses. |
| **Simulated Payment Gateway** | Represents the external payment service; simulated results are handled within the application. |

Staff users must authenticate and receive permissions based on their assigned role. **Guest identity verification is not yet specified**; the final mechanism must prevent a guest from accessing another guest's reservations.

## 4. Confirmed Assumptions and Key Rules

1. The application serves one hotel only.
2. Guests reserve a **room type**, not an individual physical room. A suitable physical room is assigned at check-in (from the team's earlier project description).
3. A room type has a defined guest capacity and a nightly rate; a hotel may have multiple physical rooms of that type.
4. The system shall not confirm reservations that exceed available room-type inventory over overlapping dates.
5. Only a permitted, assignable physical room may be assigned at check-in. A room marked **Out-of-Order** cannot be assigned.
6. Payments and refunds are simulated; no real card data is processed or stored.
7. Tax is a fixed, configurable percentage. Exact treatment of charges and rounding is **TBD**.
8. Cancellation, refund, early/late check-out, no-show, and guest-verification policies are **TBD** and must be agreed before the affected features are implemented.
9. No real SMS or email is sent. Any notification events the team chooses to demonstrate are logged.

**Proposed date convention (requires confirmation):** A stay occupies the nights from check-in (inclusive) to check-out (exclusive). Under this convention, a booking ending on November 23 does not conflict with another booking starting on November 23. The availability and reservation components must use the same convention.

## 5. Functional Requirements

Each requirement has a stable ID. The **Verification / acceptance check** column illustrates how the team can demonstrate that the requirement is met; it is not an implementation prescription.

### 5.1 Authentication and Roles — v0.1.0 (Member D; foundation shared)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-AU-01** | The system shall allow authorized hotel staff to sign in using valid staff credentials. | A valid staff account signs in; incorrect credentials are rejected. |
| **FR-AU-02** | The system shall restrict staff operations according to the user's assigned role (Receptionist, Hotel Manager, or Housekeeping Staff). | A permitted role can access its operations; an unauthorized role cannot perform restricted actions, including by direct API request. |
| **FR-AU-03** | The system shall allow an authenticated staff member to sign out and prevent further use of that signed-out session. | After sign-out, protected operations require authentication again. |

### 5.2 Room and Rate Management — v0.2.0 (Member A)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RM-01** | The system shall allow the Hotel Manager to create, view, and update room types, including their names, descriptions, and maximum guest capacities. | A newly created or modified room type appears with the correct information; invalid capacity values are rejected. |
| **FR-RM-02** | The system shall allow the Hotel Manager to add, view, update, and deactivate or remove physical-room records, each associated with a room type. | Room changes are saved; an inactive room is not offered for assignment. |
| **FR-RM-03** | The system shall allow the Hotel Manager to set and update a nightly rate for each room type. | The displayed rate matches the saved value; invalid negative rates are rejected. |
| **FR-RM-04** | The system shall display room types and physical rooms with their associated rates and relevant details to authorized staff. | The room listing shows the correct type, room identifier, and rate information. |
| **FR-RM-05** | The system shall validate room and room-type changes to prevent duplicate physical-room identifiers and inconsistent references to existing reservations or stays. | Duplicate room identifiers are rejected; changes do not leave reservations or stays pointing to invalid records. |

### 5.3 Availability Search — v0.3.0 (Member B)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-AV-01** | The system shall allow guests to search for room types using check-in date, check-out date, number of guests, and an optional room-type filter. | Valid criteria return a search result; invalid dates or guest counts are rejected. |
| **FR-AV-02** | The system shall display only room types with sufficient guest capacity and available inventory throughout the requested stay. | Fully booked or undersized room types are not presented as available for the requested stay. |
| **FR-AV-03** | The system shall calculate remaining inventory for each room type by considering overlapping reservations and rooms that are unavailable for assignment under the agreed operational rules. | Overlapping bookings reduce availability correctly; non-overlapping dates and available inventory produce correct results. |

### 5.4 Guest Records — v0.4.0 (Member C)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-GS-01** | The system shall allow an authorized workflow or staff member to create a guest record containing the information needed for a reservation. | A guest record can be created and later retrieved. |
| **FR-GS-02** | The system shall allow authorized users to view and update permitted guest-profile information. | Valid edits are saved; unauthorized access or invalid updates are rejected. |
| **FR-GS-03** | The system shall allow receptionists to search for guest records using supported identifiers such as name, phone, or reservation reference. | A search returns matching guest records without exposing records to unauthorized roles. |
| **FR-GS-04** | The system shall show an authorized user the guest's relevant reservation and stay history. | Past and current reservations linked to a guest can be retrieved accurately. |

### 5.5 Reservation Creation and Confirmation — v0.5.0 (Member B)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RS-01** | The system shall allow a guest or authorized receptionist to initiate a reservation for a **room type** by providing guest details, dates, and guest count, without selecting a physical-room number. | A valid reservation request records the selected room type and stay details; no physical room is assigned at this stage. |
| **FR-RS-02** | The system shall support reservation creation and confirmation with a recorded reservation reference and an appropriate reservation status (including Pending and Confirmed where applicable). | A valid booking is stored with a unique reference and the correct status, and its confirmation can be retrieved. |
| **FR-RS-03** | Before confirming a reservation, the system shall recheck room-type capacity for every requested night and reject requests that would cause overbooking, including concurrent conflicting requests. | A booking succeeds when inventory is available; conflicting or simultaneous over-capacity requests cannot both succeed. |

### 5.6 Reservation Search, Modification, and Cancellation — v0.6.0 (Member B)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RS-04** | The system shall let guests view their own reservations and let authorized receptionists search and view reservations by supported booking or guest details. | Guests cannot view another guest's booking; receptionists can find an existing booking. |
| **FR-RS-05** | The system shall allow a guest to modify their own eligible reservation and a receptionist to modify an authorized reservation, subject to the agreed modification rules and a new availability check. | A valid change is saved; changes that break policy or exceed inventory are rejected. |
| **FR-RS-06** | The system shall allow a guest to cancel their own eligible reservation and a receptionist to cancel an authorized reservation according to the agreed cancellation policy; cancelled bookings shall release the inventory they held. | A permitted cancellation changes the status to Cancelled and updates availability; disallowed cancellations are rejected. |

### 5.7 Check-in, Room Assignment, and Check-out — v0.7.0 (Member C)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-CI-01** | The system shall allow a receptionist to assign an available, suitable physical room of the reserved room type to a confirmed reservation at check-in. | The assigned room matches the room type and is not assigned to another overlapping active stay. |
| **FR-CI-02** | The system shall allow a receptionist to check in an eligible reservation, record the arrival, and update the reservation/stay status. | A valid check-in is recorded as CheckedIn; an invalid or already checked-in booking cannot be checked in again. |
| **FR-CI-03** | The system shall allow a receptionist to check out an active stay, record departure, and update the room and reservation/stay records for billing and housekeeping. | A valid check-out is recorded as CheckedOut; the room becomes available for the appropriate housekeeping workflow. |
| **FR-CI-04** | The system shall validate early and late check-out requests according to the hotel's agreed policy, including any permitted adjustments. | The system applies the approved policy consistently and rejects disallowed actions. **Specific time and fee rules: TBD.** |

### 5.8 Billing, Invoices, Simulated Payments, and Refunds — v0.8.0 (Member D)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-BL-01** | The system shall create and maintain a folio for a reservation or stay, including accommodation charges based on the applicable nightly rate and length of stay. | The folio displays the correct number of nights and accommodation amount. |
| **FR-BL-02** | The system shall allow authorized staff to add and view itemized extra charges on a guest folio. | A saved extra charge appears once with the correct description and amount. |
| **FR-BL-03** | The system shall calculate the folio subtotal, apply the hotel's configured fixed tax percentage, and calculate the total amount due. | Totals match independently calculated expected values for a known test example. |
| **FR-BL-04** | The system shall allow authorized staff to generate and view an itemized invoice or bill from the folio. | An invoice shows reservation/stay details, charges, tax, and total correctly. |
| **FR-BL-05** | The system shall record simulated payment and refund transactions, including amount, outcome/status, and related reservation or folio, without processing real payment information. | Successful and unsuccessful simulated payments and permitted refunds are recorded correctly; no real card data is required. |

### 5.9 Housekeeping and Operations Dashboard — v0.9.0 and v0.10.0 (Member A)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-OP-01** | The system shall maintain and display each physical room's cleaning/operational status, supporting **Clean**, **Dirty**, **Inspected**, and **Out-of-Order** as applicable. | Staff can view the recorded status of each room. |
| **FR-OP-02** | The system shall allow authorized housekeeping staff to update room status using the approved valid status transitions. | A valid cleaning-status transition succeeds; a prohibited transition is rejected. |
| **FR-OP-03** | The system shall prevent a physical room that is Out-of-Order, or otherwise ineligible under the agreed readiness rules, from being assigned at check-in. | An attempt to assign an Out-of-Order room is rejected. |
| **FR-OP-04** | The system shall provide the Hotel Manager with a dashboard showing occupancy, expected arrivals/departures, and revenue figures based on recorded hotel data. | Dashboard values match counts and totals computed from the underlying records for the same period. |

**Room-model note:** Reservation status, physical-room occupancy, and housekeeping/operational status are related but not identical. The room state diagram and the implementation must represent these consistently; the exact state/transition rules will be finalized with Member A.

## 6. Non-Functional Requirements

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **NFR-01 — Data integrity** | The system shall prevent double booking or assignment conflicts, including concurrent reservation attempts, and shall preserve valid links among rooms, guests, reservations, stays, and financial records. | Automated concurrent-booking and data-integrity tests find no over-capacity confirmed bookings or conflicting room assignments. |
| **NFR-02 — Performance** | Core interactive operations (especially room searches and reservation lookups) shall return a result in **under 2 seconds** under the agreed demonstration workload. | Measure response times in a repeatable test. **Dataset size, request concurrency, environment, and measurement threshold: TBD.** |
| **NFR-03 — Authentication and security** | Staff passwords shall be stored as secure hashes rather than plaintext; protected functions shall enforce the user's permissions, and guest booking information shall be accessible only through an authorized ownership-verification flow. | Database inspection finds no plaintext staff passwords; role/ownership tests reject unauthorized access. |
| **NFR-04 — Auditability** | The system shall keep an audit log of important staff and reservation operations, including the event/action, affected record, time, and responsible user or system actor where available. | A sample booking modification, check-in, billing action, or room-status update produces a corresponding audit entry. |
| **NFR-05 — Usability** | The web interface shall let guests and hotel staff complete their main role-appropriate workflows without using command-line tools. | Demo users can perform search/reservation and staff check-in/out or management workflows through the interface. |
| **NFR-06 — Reproducibility and maintainability** | The repository shall provide documented steps for installing dependencies, configuring/initializing the database, loading demo data, starting the application, and running tests. | A teammate can follow the documented instructions from a fresh clone and run the application and applicable tests. |

## 7. Primary Data and Lifecycle Concepts

The requirements imply at least the following concepts; the final ERD and Class Diagram define their exact fields and relationships:

- **RoomType:** name, guest capacity, nightly rate, and related physical rooms.
- **Room:** unique room identifier/number, room type, physical/housekeeping status.
- **Guest:** identity and contact information necessary for reservations.
- **Reservation:** guest, reserved room type, date range, guest count, reference, status, and applicable price information.
- **Stay / RoomAssignment:** actual physical room assigned and check-in/check-out details.
- **Folio / Charge / Invoice:** accommodation charges, extras, tax, totals, and bill details.
- **Payment / Refund:** simulated transaction amounts, results, and references.
- **StaffUser / Role / AuditEvent:** login identity, permissions, and logged actions.

The planned reservation state diagram includes **Pending**, **Confirmed**, **CheckedIn**, **CheckedOut**, **Cancelled**, and **NoShow**. The exact events that create/expire Pending reservations or mark NoShow, and their effects on inventory and billing, remain **TBD**.

## 8. Requirements-to-Increment Mapping

This mapping follows the updated four-member project plan. The team's separate `incremental_development_plan.md` and `traceability_matrix.md` should use the same IDs.

| Version | Owner | Requirement IDs | Main capability |
|---|---|---|---|
| **v0.1.0** | D + shared foundation | FR-AU-01–03; NFR-03 (foundation) | Staff login, roles, database/repository setup |
| **v0.2.0** | A | FR-RM-01–05 | Rooms, room types, and rates |
| **v0.3.0** | B | FR-AV-01–03 | Availability search |
| **v0.4.0** | C | FR-GS-01–04 | Guest records and profiles |
| **v0.5.0** | B | FR-RS-01–03; NFR-01 (reservation integrity) | Reservation creation and confirmation |
| **v0.6.0** | B | FR-RS-04–06 | Booking search, modification, and cancellation |
| **v0.7.0** | C | FR-CI-01–04 | Room assignment, check-in and check-out |
| **v0.8.0** | D | FR-BL-01–05 | Folio, billing, invoice, simulated transactions |
| **v0.9.0** | A | FR-OP-01–03 | Room status and housekeeping |
| **v0.10.0** | A | FR-OP-04 | Operations dashboard |
| **v1.0.0** | All | All FRs and NFRs | Full integration, testing, traceability and release |

**Cross-cutting NFRs:** NFR-01, NFR-02, NFR-04, NFR-05, and NFR-06 must also be considered and tested in the increments where relevant, not only at final release.

## 9. Open Decisions Requiring Team Agreement

The following are **not missing features**; they are decisions that need agreement before the relevant increments are fully specified. Changes must be reflected in this file, the UML/C4 diagrams, incremental plan, and traceability matrix.

| Topic | Decision needed | Relevant increments |
|---|---|---|
| **Guest booking access** | Guest accounts/login, booking reference plus verification, or another secure ownership mechanism? | v0.5.0–v0.6.0 |
| **Pending reservations** | Does Pending hold inventory? If so, for how long, and how does it expire? | v0.5.0 |
| **Date-overlap convention** | Confirm check-out date is exclusive, and specify how reservations spanning several nights use type-level inventory. | v0.3.0–v0.5.0 |
| **Unavailable rooms** | How do Out-of-Order blocks affect future inventory? What happens if a room becomes unavailable after reservations are confirmed? | v0.3.0, v0.7.0, v0.9.0 |
| **Booking changes and cancellations** | Cutoff times, eligible statuses, fees, and effects on tax/payment/refunds? | v0.6.0–v0.8.0 |
| **No-show and early/late check-out policies** | When is NoShow recorded? When are early/late check-outs allowed, and are extra charges applied? | v0.7.0–v0.8.0 |
| **Pricing and currency** | Reservation price snapshot vs current rate, currency, rounding, tax on extras, and applicable rates if a stay changes? | v0.2.0, v0.5.0, v0.8.0 |
| **Room readiness transitions** | What states are valid, who can change them, and is Clean or Inspected required for assignment? | v0.7.0–v0.9.0 |
| **Dashboard definitions** | What counts as occupancy and revenue, and which period/filter is used? | v0.10.0 |
| **Performance test profile** | Database size, concurrent users, test environment, and how the 2-second requirement is measured? | NFR-02 / testing |

## 10. Review and Change Control

- **Member B** maintains this requirements document, with **Member D** reviewing authentication, billing, and testability details and other owners reviewing their areas.
- Before a requirement is implemented, its wording, business rules, and acceptance check should be consistent with the relevant diagram and increment plan.
- If requirements change, keep the existing IDs where their meaning stays the same; update affected design, tests, implementation, and traceability references.
- This draft becomes a team-approved specification after review of the **TBD** items relevant to current increments. Future decisions may be recorded as separate updates and commits.
