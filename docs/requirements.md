# Hotel Reservation System — Requirements Specification

**Group:** 15  
**Course:** CS13477 Software Engineering — Fall 2026  
**Document:** `docs/requirements.md`  
**Status:** Revised draft for team review. Agreed guest access model included; remaining policy decisions are marked **TBD**.

## 1. Purpose

This document states the functional and non-functional requirements for the Hotel Reservation System. Requirement IDs are stable across the project description, incremental development plan, UML models, tests, and traceability matrix. The planned system is a web application for one hotel.

## 2. System Overview and Scope

### 2.1 In scope

- Staff login, logout, and role-based authorization.
- Room types, physical rooms, nightly rates, and guest capacities.
- Availability search, room-type reservations, and prevention of overbooking.
- **Account-free guest booking**, with a booking reference and separate secret access code for access to one booking.
- Persistent guest records, authorized staff search, and linked reservation/stay history.
- Physical-room assignment and **simulated manual identity verification** at check-in; check-out.
- Folios, tax, invoices, extra charges, and simulated payment/refund records.
- Housekeeping/operational room status and management dashboard.

### 2.2 Out of scope

- Guest online accounts or online guest login; public access to a guest's full history.
- Multiple hotels, live booking platforms, or real card processing/card storage.
- Real SMS/email delivery and automated external identity-document verification.

## 3. Actors and Access Responsibilities

| Actor | Responsibilities and access |
|---|---|
| **Guest** | Search and create bookings without an account; use the booking reference **and** secret code to view, modify, or cancel **that booking only**. |
| **Receptionist** | After staff login, manage guest records/history and reservations; verify presented ID manually at check-in; assign rooms and handle check-in/out and billing. |
| **Hotel Manager** | After staff login, manage rooms/types/rates and view operational dashboard data. |
| **Housekeeping Staff** | After staff login, view and update permitted room readiness/cleaning status. |
| **Simulated Payment Gateway** | Supplies simulated payment/refund outcomes inside the project; no external bank interaction. |

**Important distinction:** A **Guest** is a persistent hotel data record, not necessarily a login account. A guest may have several reservations under one internal `guest_id`, but a booking reference + code grants online access to **one reservation**, not all stays linked to that guest. Authorized staff can see guest history.

## 4. Agreed Rules and Assumptions

1. The application manages **one hotel** using demo records.
2. A guest books a **room type**, not a particular room number. A physical room is assigned only at check-in.
3. Each room type has a maximum guest count and a nightly rate. Availability is based on room-type inventory across **every night** of the stay.
4. The check-in date is inclusive and check-out date exclusive. For example, November 20–23 and November 23–25 do not overlap.
5. Confirmed reservations use room-type inventory. The booking operation must recheck capacity atomically so concurrent requests cannot overbook. The treatment of **Pending** reservations is **TBD**.
6. Guests do **not** register or log in. Successful creation/confirmation issues a unique booking reference and an independently generated, unpredictable access code. The access code is shown once, stored as a secure hash, and required with the reference for later guest booking access.
7. Booking access credentials are **not** dependent on simulated payment. Real payments and real card data are not used; payment simulation is part of billing (Increment 8).
8. Staff authenticate using their accounts and roles. Guest booking credentials cannot be used to access staff functions, guest profiles, or full booking history.
9. A new reservation references a guest record. Receptionists can search for existing guest records and, after checking the identity match, link a returning guest's reservation/stay history. **Do not automatically merge guest records based only on name, email, or phone.**
10. At physical check-in, receptionists check the presented identification against the booked/authorized guest and record a **simulated manual verification** result before completing check-in. No real ID image or governmental service is required.
11. Out-of-Order rooms must not be assigned. How operational blocks reduce future room-type inventory is a design decision to finalize with Increments 3, 7, and 9.
12. Tax is a configurable fixed percentage, nightly rates are per room type, and charges/refunds are simulated. Cancellation, no-show, detailed pricing, early/late check-out, and access-code recovery policies are **TBD**.
13. No real SMS/email is delivered. If notification events are represented, they are logged locally and must not expose secret codes.

## 5. Functional Requirements

Each requirement has a stable ID, a testable statement, and an example acceptance check. The requirement numbering preserves the original ranges: **34 FRs** in total.

### 5.1 Authentication and Roles — v0.1.0 (Anas Alseddah)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-AU-01** | The system shall allow authorized hotel staff to sign in with valid staff credentials; guests shall not need a login account to make a reservation. | Staff login succeeds with correct credentials and fails with incorrect ones; guest booking is accessible without staff login. |
| **FR-AU-02** | The system shall enforce role-based access for Receptionist, Hotel Manager, and Housekeeping Staff in the UI and backend/API. | Authorized staff can perform permitted operations; direct unauthorized requests are rejected. |
| **FR-AU-03** | The system shall allow an authenticated staff member to sign out and invalidate their access to protected operations. | After logout, a protected request requires login again. |

### 5.2 Room and Rate Management — v0.2.0 (Anas Alseddah)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RM-01** | The system shall allow the Hotel Manager to create, view, and update room types with name, description, and maximum guest capacity. | Updates persist; zero/invalid capacities are rejected. |
| **FR-RM-02** | The system shall allow the Hotel Manager to add, view, update, and safely deactivate physical rooms linked to room types. | Room records persist; inactive rooms are not eligible for assignment. |
| **FR-RM-03** | The system shall allow the Hotel Manager to set and update a nonnegative nightly rate for each room type. | Saved rates appear correctly; negative rates are rejected. |
| **FR-RM-04** | The system shall display room types, physical rooms, rates, and relevant room details to authorized staff. | Staff lists display the correct room identifiers, types, and rates. |
| **FR-RM-05** | The system shall reject duplicate physical room numbers and changes that would leave existing reservations/stays with invalid room references. | Duplicate identifiers and inconsistent updates are rejected. |

### 5.3 Availability Search — v0.3.0 (Anas Alseddah)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-AV-01** | The system shall let guests search room types using check-in/check-out dates, guest count, and an optional room-type filter. | Valid searches return results; invalid dates and nonpositive guest counts are rejected. |
| **FR-AV-02** | The system shall display only room types whose guest capacity and remaining inventory cover the **entire** requested stay. | Sold-out or undersized types are excluded, including a type unavailable on just one requested night. |
| **FR-AV-03** | The system shall calculate room-type availability using overlapping inventory-consuming reservations and applicable room-unavailability rules, consistent with the check-out-exclusive convention. | Adjacent non-overlapping stays are permitted; relevant overlapping bookings reduce inventory; Out-of-Order treatment follows the agreed rule. |

### 5.4 Guest Records and History — v0.4.0 (Yousef Taha)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-GS-01** | The system shall create or link a persistent guest record for each reservation **without requiring a guest website account**. | A new booking references a valid internal guest ID; no guest password/login is required. |
| **FR-GS-02** | The system shall allow authorized staff to view and update permitted guest profile/contact information; guests using booking credentials shall not gain direct access to full guest records. | Staff edits persist; unauthorized users cannot view/edit private profiles. |
| **FR-GS-03** | The system shall allow receptionists to search guest records by supported identifiers (such as name, phone, email, or reservation reference) and link a returning guest only after checking the match. | Suggested matches can be reviewed; same-name records are not automatically merged or linked without verification. |
| **FR-GS-04** | The system shall allow authorized receptionists to view reservations and completed stays linked to a guest's internal ID. | Multiple verified bookings appear in the correct guest's history; a booking access code does not expose that history. |

### 5.5 Reservation Creation and Confirmation — v0.5.0 (Yousef Taha)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RS-01** | The system shall allow a guest without an account, or an authorized receptionist, to initiate a room-**type** reservation with guest details, dates, and guest count; no physical room is assigned at this stage. | Booking inputs are validated and linked to a guest record without a room number. |
| **FR-RS-02** | The system shall store a valid reservation with a unique booking reference, status (Pending/Confirmed where applicable), and a separately generated secret access code made available to the guest upon successful booking creation/confirmation. | Booking reference is unique; code is shown to the booking guest once, stored only as a hash, and never re-displayed from the database; payment is not required to issue it. |
| **FR-RS-03** | Before confirmation, the system shall recheck room-type capacity for every reserved night and prevent overbooking, including concurrent conflicting requests. | A booking with available capacity succeeds; two simultaneous requests cannot both consume the final available room. |

### 5.6 Reservation Viewing, Modification, and Cancellation — v0.6.0 (Yousef Taha)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RS-04** | The system shall allow guests to view **only the reservation identified by both its booking reference and matching secret access code**; authorized receptionists may find bookings through their staff permissions. | Reference alone, wrong code, or another booking's code does not reveal reservation information; receptionist lookup is authorized. |
| **FR-RS-05** | The system shall allow a guest who supplies the matching reference and code, or an authorized receptionist, to modify an eligible reservation, following the agreed policy and rechecking availability. | Authorized changes persist; incorrect codes, disallowed changes, and over-capacity dates are rejected. |
| **FR-RS-06** | The system shall allow a verified guest or authorized receptionist to cancel an eligible reservation according to the cancellation policy and release its inventory. | Correct credentials plus eligible status permit cancellation; canceled capacity is returned; wrong credentials or disallowed requests fail. |

### 5.7 Check-in, Room Assignment, and Check-out — v0.7.0 (Mohammad Ameerah)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-CI-01** | The system shall allow a receptionist to assign an available, ready physical room of the reserved type at check-in; overlapping active stays and Out-of-Order rooms cannot receive conflicting assignments. | Only suitable rooms can be assigned; room number is recorded at check-in rather than reservation creation. |
| **FR-CI-02** | Before completing check-in, the system shall require a receptionist to record that they **manually compared presented identification** with the booked/authorized guest information (simulated for demonstration), then record the arrival and update stay/reservation status. | Check-in is blocked without the verification confirmation; wrong guest or already-checked-in reservation is rejected; no ID images are stored. |
| **FR-CI-03** | The system shall allow a receptionist to check out an active stay, record departure, and update relevant reservation, billing, and room-readiness information. | A valid checkout changes the stay to CheckedOut and triggers the appropriate housekeeping status. |
| **FR-CI-04** | The system shall apply the agreed early/late checkout policy, including any permitted changes or charges. | Allowed cases succeed and disallowed cases fail; exact timing/fee policy remains **TBD**. |

### 5.8 Billing, Invoices, Simulated Payments, and Refunds — v0.8.0 (Mohammad Ameerah)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-BL-01** | The system shall create a folio for a reservation/stay and calculate accommodation charges from the applicable nightly rate and stay length. | A known stay yields the correct nightly subtotal. |
| **FR-BL-02** | The system shall allow authorized staff to add and view itemized extra charges on a folio. | Each extra charge appears correctly and is not duplicated. |
| **FR-BL-03** | The system shall apply the configurable tax percentage and compute subtotal, tax, and total accurately. | A known test invoice matches independently calculated totals. |
| **FR-BL-04** | The system shall allow authorized staff to generate and view an itemized bill/invoice using recorded stay and folio details. | Itemization, totals, and related reservation are correct. |
| **FR-BL-05** | The system shall record simulated payment/refund transactions, their amounts and statuses, without connecting to a real gateway or collecting card details; a simulated payment is **not a prerequisite** to generating booking access credentials. | Successful/failed simulated payments and permitted refunds are recorded; reservation creation still works without payment. |

### 5.9 Housekeeping and Operations — v0.9.0 / v0.10.0 (Zein A. Sunoqrot)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-OP-01** | The system shall track and display each physical room's cleaning/operational condition, including Clean, Dirty, Inspected, and Out-of-Order as applicable. | Authorized staff see the correct state for each physical room. |
| **FR-OP-02** | The system shall allow authorized housekeeping staff to update room status using approved state transitions, recording the change. | Valid transitions succeed; prohibited transitions fail; status updates are recorded. |
| **FR-OP-03** | The system shall block assigning Out-of-Order rooms, or rooms failing the agreed readiness rules, at check-in. | Invalid room assignments are rejected. |
| **FR-OP-04** | The system shall show the Hotel Manager occupancy, expected arrivals/departures, revenue, and relevant room-readiness information based on recorded data. | Dashboard counts and totals match the database for the same chosen period. |

**Modeling note:** A reservation's state, a physical room's occupied/available condition, and its housekeeping/operational status are separate concepts. Room readiness does not itself indicate whether a room is booked for future dates.

## 6. Non-Functional Requirements

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **NFR-01 — Integrity** | The system shall prevent inventory overbooking and conflicting physical-room assignments, including simultaneous reservation attempts; important data relationships shall remain consistent. | Concurrent reservation and assignment tests preserve available inventory and database relationships. |
| **NFR-02 — Performance** | Core interactive requests, particularly room-type search and booking lookup, shall complete in **under 2 seconds** under the agreed demonstration workload. | Repeatable timed tests; exact data volume, concurrency, and measurement method are **TBD**. |
| **NFR-03 — Security** | Staff passwords and guest booking-access codes shall be stored as strong hashes; staff permissions and per-booking guest access shall be enforced server-side, with limits on repeated code guesses. Access codes shall not appear in ordinary logs or later API responses. | Plaintext secret inspection finds none; wrong codes, attempts to enumerate references, and unauthorized actions are rejected or throttled. |
| **NFR-04 — Auditability** | The system shall audit important staff actions and booking/status changes with action, affected record, time, and responsible user or actor where possible; secret codes and sensitive ID material shall not be logged. | Example modifications, check-ins, payments, and status updates produce audit records without secrets. |
| **NFR-05 — Usability** | Guests shall be able to search/book and manage one booking through the web UI without registering; staff shall be able to complete role-appropriate tasks via web UI without terminal commands. | Demo guest books without an account and accesses a booking using both credentials; staff workflows function in the browser. |
| **NFR-06 — Reproducibility** | The repo shall document dependency installation, environment configuration, database initialization, demo data, running the application, and automated tests. | Another team member can run the system and tests from a clean clone following instructions. |

## 7. Data and Lifecycle Concepts

The Class Diagram and ERD will specify the details of these entities:

- **RoomType:** capacity, nightly rate, and physical-room inventory.
- **Room:** unique room number, type, operational/housekeeping condition, and eligibility for assignment.
- **Guest:** internal `guest_id`, personal/contact details needed for reservations; **not** a login account.
- **Reservation:** guest reference, room type, dates, guest count, price snapshot where applicable, booking reference, **hashed** booking-access code, and status.
- **Stay/RoomAssignment:** confirmed identity-check indicator, physical room, arrival/departure, and occupancy details.
- **Folio/Charge/Invoice/Payment/Refund:** itemized financial details and simulated transaction outcomes.
- **StaffUser/Role/AuditEvent:** staff login identity, permissions, and reviewable audit history.

A single Guest may have zero or many Reservations, and each Reservation belongs to one Guest. One verified guest's reservations/stays can be shown in staff history. Never treat knowing one reservation's secret code as proof of authority to view other records linked to the same guest.

The reservation state model will use **Pending**, **Confirmed**, **CheckedIn**, **CheckedOut**, **Cancelled**, and **NoShow** as applicable. The rules for Pending and NoShow are **TBD**.

## 8. Requirements-to-Increment Mapping

| Version | Owner | Requirement IDs | Main capability |
|---|---|---|---|
| **v0.1.0** | **Anas Alseddah** | FR-AU-01–03; NFR-03 (staff security); NFR-06 | Staff login, roles, runnable backend/database/frontend foundation |
| **v0.2.0** | **Anas Alseddah** | FR-RM-01–05 | Rooms, room types, and rates |
| **v0.3.0** | **Anas Alseddah** | FR-AV-01–03; NFR-02 | Room-type availability search |
| **v0.4.0** | **Yousef Taha** | FR-GS-01–04 | Guest records, verification of returning guest matches, staff-visible history |
| **v0.5.0** | **Yousef Taha** | FR-RS-01–03; NFR-01, NFR-03 (guest code) | Create booking, issue reference/code, prevent overbooking |
| **v0.6.0** | **Yousef Taha** | FR-RS-04–06; NFR-03 (booking access) | Per-booking viewing, modification and cancellation |
| **v0.7.0** | **Mohammad Ameerah** | FR-CI-01–04; NFR-01 | Simulated ID check, physical-room assignment, check-in/out |
| **v0.8.0** | **Mohammad Ameerah** | FR-BL-01–05 | Billing, invoice, simulated payments and refunds |
| **v0.9.0** | **Zein A. Sunoqrot** | FR-OP-01–03 | Housekeeping and room status |
| **v0.10.0** | **Zein A. Sunoqrot** | FR-OP-04 | Operations dashboard |
| **v1.0.0** | **All four** | All functional and non-functional requirements | Full integration, tests, documentation, traceability, final release |

**Cross-cutting requirements:** NFR-01–06 apply throughout wherever relevant. Owners coordinate changes to common models, tests, and APIs.

## 9. Open Decisions Requiring Team Agreement

The **guest account decision is closed**: guests book without logging in and access only a specific reservation using a reference plus a secret code. The following details still require agreement before the affected increments are fully implemented.

| Topic | Decision needed | Relevant increments |
|---|---|---|
| **Lost booking code** | Staff-verified secure reissue/reset process without real email/SMS; rules for invalidating old codes. | v0.5.0–v0.6.0 |
| **Returning guest matching** | Which fields the receptionist checks before linking records; handling duplicate profiles; no automatic matching based on contact info alone. | v0.4.0–v0.5.0 |
| **Pending/NoShow** | Whether Pending holds inventory, its expiry, and when a confirmed booking becomes NoShow. | v0.5.0–v0.7.0 |
| **Unavailable room blocks** | How past/current/future Out-of-Order periods affect type-level availability and existing reservations. | v0.3.0, v0.7.0, v0.9.0 |
| **Cancellation and modification** | Eligible booking statuses, deadlines, penalties and effects on financial records. | v0.6.0–v0.8.0 |
| **Pricing and payments** | Currency, rate snapshots, tax rounding, tax on extras, when simulated payment is optionally collected, and refund triggers. | v0.2.0, v0.5.0, v0.8.0 |
| **Room readiness** | Whether Clean or Inspected is needed for assignment, who may update states, and exceptions. | v0.7.0–v0.9.0 |
| **Early/late checkout** | When checkout is allowed and how extra fees are calculated. | v0.7.0–v0.8.0 |
| **Dashboard metrics** | Definitions of occupancy, recognized revenue, and date filters. | v0.10.0 |
| **Performance evaluation** | Dataset size, concurrent requests, environment, and how to measure the 2-second target. | NFR-02 |

## 10. Review and Change Control

- **Anas Alseddah** owns Increments 1–3 and coordinates changes to foundation/room/availability rules in this document.
- **Yousef Taha** owns guest records and reservations (Increments 4–6), including access-code issuance and per-booking verification.
- **Mohammad Ameerah** owns physical check-in/out verification and billing (Increments 7–8).
- **Zein A. Sunoqrot** owns housekeeping and dashboard requirements (Increments 9–10).
- The requirements file is maintained collaboratively with one agreed version; change its IDs only when absolutely necessary and update the incremental plan, diagrams, implementation tests, and traceability matrix together.
- A requirement is ready to implement once its business rules, acceptance checks, and relevant diagrams are reviewed. No completed work or tests are claimed by this draft alone.