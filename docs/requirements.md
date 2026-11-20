# Hotel Reservation System — Requirements Specification

**Group:** 15  
**Course:** CS13477 Software Engineering — Fall 2026  
**Document:** `docs/requirements.md`  
**Status:** Revised draft for team review. Account-free guest access, five-minute booking holds, mandatory simulated payment, and agreed booking policies are documented; remaining integration details are marked **TBD**.

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
- Mandatory simulated payment before booking confirmation; folios, tax, invoices, extra charges, and simulated payment/refund records.
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
5. Confirmed reservations and unexpired **Pending** holds consume room-type inventory per booked night. After the guest proceeds to payment, an atomic capacity check creates one Pending hold with `expires_at = server_time + 5 minutes`. Expired or explicitly cancelled holds release capacity; refreshing or retrying payment must not extend the deadline.
6. Guests do **not** register or log in. Only after successful simulated payment and atomic confirmation of an unexpired hold does the system issue a unique booking reference and independent, unpredictable secret access code. The code is shown once, stored only as a secure hash, and required with the reference for later access to that one booking.
7. **Successful simulated payment is required** to confirm a booking and issue booking credentials. Failed payment attempts can be retried within the original five-minute hold. Payment successes that cannot be committed must be reversed or recorded for reconciliation. No real payments or card data are used; Increment 5 provides basic simulation and Increment 8 expands billing/refunds.
8. Staff authenticate using their accounts and roles. Guest booking credentials cannot be used to access staff functions, guest profiles, or full booking history.
9. A new reservation references a guest record. Receptionists can search for existing guest records and, after checking the identity match, link a returning guest's reservation/stay history. **Do not automatically merge guest records based only on name, email, or phone.**
10. At physical check-in, receptionists check the presented identification against the booked/authorized guest and record a **simulated manual verification** result before completing check-in. No real ID image or governmental service is required.
11. Out-of-Order rooms must not be assigned. How operational blocks reduce future room-type inventory is a design decision to finalize with Increments 3, 7, and 9.
12. Currency is **JOD**, standard check-in is **14:00**, standard check-out is **12:00**, and tax is a configurable percentage (exact demo rate and rounding remain TBD). A Confirmed booking may be cancelled before scheduled check-in: **at least 24 hours** beforehand gives a **100% simulated refund**; less than 24 hours beforehand gives **no refund**. Date modification requires 24 hours' notice, availability rechecks, and simulated settlement of the price difference. At **12:00 noon the following day**, an unchecked-in Confirmed reservation becomes `NoShow` without refund and without a late-arrival exception. Early/late check-out and some pricing details remain TBD.
13. No real SMS/email is delivered. Notification logs, if present, must never expose secret codes.
14. Lost codes are reset only after receptionist verification, invalidating the prior code. Expired holds remain recorded for audit; hold creation, retries and access-code verification have anti-abuse protections.

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
| **FR-AV-03** | The system shall calculate room-type availability using overlapping Confirmed reservations, **unexpired five-minute Pending holds**, and applicable room-unavailability rules, consistently with the check-out-exclusive convention. | Active holds reduce inventory across every requested night; expired/cancelled holds do not, even when background cleanup is delayed. |

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
| **FR-RS-01** | The system shall allow account-free guests and authorized receptionists to enter guest details, dates, room type and guest count; proceeding to payment creates an **atomic five-minute Pending reservation hold** when capacity exists, without assigning a physical room. | Valid details create exactly one Pending hold linked to a guest record, reducing nightly availability; duplicate Next requests do not create extra holds. |
| **FR-RS-02** | The system shall confirm an unexpired Pending reservation **only after successful simulated payment and database commit**, then issue a unique booking reference and separate unpredictable secret access code. | A failed payment permits retry in the original five minutes; successful payment confirms once; the secret is shown once and stored only as a secure hash; no code is issued to an expired or unconfirmed booking. |
| **FR-RS-03** | The system shall protect Pending hold creation and Pending-to-Confirmed conversion with concurrency-safe database operations, counting active holds and Confirmed reservations on every booked night to prevent overbooking. | Two simultaneous guests cannot both hold the final room-type unit; expiry/cancellation release capacity; late payment cannot confirm an expired hold. |

### 5.6 Reservation Viewing, Modification, and Cancellation — v0.6.0 (Yousef Taha)

| ID | Requirement | Verification / acceptance check |
|---|---|---|
| **FR-RS-04** | The system shall allow guests to view **only the reservation identified by both its booking reference and matching secret access code**; authorized receptionists may find bookings through their staff permissions. | Reference alone, wrong code, or another booking's code does not reveal reservation information; receptionist lookup is authorized. |
| **FR-RS-05** | The system shall let an authorized guest (matching reference and secret code) or receptionist modify eligible Confirmed bookings; date changes require at least **24 hours before scheduled check-in**, a protected availability recheck, and simulated additional payment/refund of any price difference. | Eligible changes settle the difference without losing original inventory; unauthorized, too-late, unavailable or invalid-state changes fail safely. |
| **FR-RS-06** | The system shall allow an authorized guest/receptionist to cancel a Confirmed booking **before scheduled check-in at 14:00** and release inventory; cancellation **at least 24 hours before** gives a **100% simulated refund**, otherwise **no refund**. An unpaid Pending hold may also be explicitly cancelled. | Correct credentials and eligible state permit cancellation; refund cutoff is relative to 14:00 hotel-local check-in; invalid/late requests fail; a cancelled Pending hold releases inventory without refund. |

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
| **FR-BL-05** | The system shall record simulated booking payments, additional payments, reversals and refunds with statuses, without real bank connections or card storage. Successful simulated booking payment is **required before booking confirmation and credential issuance**; Increment 8 extends billing and transaction records. | Failed payment cannot confirm; success and valid hold may confirm; failed/expired confirmations record reversal or reconciliation, and refunds match policy; no real money moves. |

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
| **NFR-01 — Integrity** | The system shall prevent overbooking and conflicting physical-room assignment, including simultaneous Pending hold creation and confirmation, while keeping guest, hold, reservation and payment records consistent. | Concurrent holds for the last room yield at most one valid hold; expired holds release capacity; failed commits cannot report confirmed bookings. |
| **NFR-02 — Performance** | Core interactive requests, particularly room-type search and booking lookup, shall complete in **under 2 seconds** under the agreed demonstration workload. | Repeatable timed tests; exact data volume, concurrency, and measurement method are **TBD**. |
| **NFR-03 — Security** | Staff passwords and booking codes use strong one-way hashing. Staff and per-booking guest permissions are enforced server-side; hold creation, payment attempts and access-code guesses are rate-limited. Codes cannot appear in ordinary logs or later API responses. | Plaintext secret inspection finds none; cross-booking access and repeated guesses fail or are throttled; duplicate hold/payment submissions cannot bypass controls. |
| **NFR-04 — Auditability** | The system shall audit important staff actions and booking/status changes with action, affected record, time, and responsible user or actor where possible; secret codes and sensitive ID material shall not be logged. | Example modifications, check-ins, payments, and status updates produce audit records without secrets. |
| **NFR-05 — Usability** | Guests shall search rooms, hold inventory, complete required simulated payment, and manage one booking through the web UI without registration; staff use appropriate web UI. | Demo shows five-minute countdown, retries with unchanged deadline, confirmation credentials only after success and no required guest login or terminal commands. |
| **NFR-06 — Reproducibility** | The repo shall document dependency installation, environment configuration, database initialization, demo data, running the application, and automated tests. | Another team member can run the system and tests from a clean clone following instructions. |

## 7. Data and Lifecycle Concepts

The Class Diagram and ERD will specify the details of these entities:

- **RoomType:** capacity, nightly rate, and physical-room inventory.
- **Room:** unique room number, type, operational/housekeeping condition, and eligibility for assignment.
- **Guest:** internal `guest_id`, personal/contact details needed for reservations; **not** a login account.
- **Reservation:** guest reference, room type, dates, guest count, JOD price snapshot, booking reference, hashed access code, status, Pending `created_at`/`expires_at`, and simulated payment linkage.
- **Stay/RoomAssignment:** confirmed identity-check indicator, physical room, arrival/departure, and occupancy details.
- **Folio/Charge/Invoice/Payment/Refund:** itemized financial details and simulated transaction outcomes.
- **StaffUser/Role/AuditEvent:** staff login identity, permissions, and reviewable audit history.

A single Guest may have zero or many Reservations, and each Reservation belongs to one Guest. One verified guest's reservations/stays can be shown in staff history. Never treat knowing one reservation's secret code as proof of authority to view other records linked to the same guest.

The reservation state model uses **Pending**, **Confirmed**, **Expired**, **CheckedIn**, **CheckedOut**, **Cancelled**, and **NoShow**. Pending holds inventory for five minutes; failed payment leaves it Pending without extending expiry. Confirmed requires successful simulated payment, an active hold and committed reservation. NoShow follows the next-day-noon deadline.

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

The **account-free model and booking policies** are working design decisions approved by the Increment 5 owner, pending team review: five-minute inventory holds, mandatory simulated payment, same-window retries, 24-hour cancellation/modification rules, and next-day-noon NoShow without a late-arrival exception. The following implementation details remain open:

| Topic | Decision needed | Relevant increments |
|---|---|---|
| **Lost booking code** | Exact receptionist verification and secure reissue process (old code invalidated; no real SMS/email). | v0.5.0–v0.6.0 |
| **Returning guest matching** | Exact fields checked and how duplicate records are handled without automatic contact-only merging. | v0.4.0–v0.5.0 |
| **Unavailable room blocks** | Impact of past/future Out-of-Order periods on inventory and existing reservations. | v0.3.0, v0.7.0, v0.9.0 |
| **Payment implementation** | Idempotency, reversal/reconciliation failure handling and ownership across Increments 5 and 8. | v0.5.0, v0.8.0 |
| **Pricing and tax** | Demo tax rate, rounding, extras and invoice details; currency JOD and rate snapshots are agreed. | v0.2.0, v0.5.0, v0.8.0 |
| **Anti-abuse** | Configurable request limits across sessions; one active Pending hold per session is the initial rule. | v0.5.0 |
| **Room readiness** | Clean versus Inspected assignment requirements and authorized exceptions. | v0.7.0–v0.9.0 |
| **Early/late checkout** | Allowed timing and fee calculation. | v0.7.0–v0.8.0 |
| **Dashboard metrics** | Occupancy, recognized revenue and date filters. | v0.10.0 |
| **Performance evaluation** | Dataset, concurrency and two-second-target measurement process. | NFR-02 |

## 10. Review and Change Control

- **Anas Alseddah** owns Increments 1–3 and coordinates changes to foundation/room/availability rules in this document.
- **Yousef Taha** owns guest records and reservations (Increments 4–6), including access-code issuance and per-booking verification.
- **Mohammad Ameerah** owns physical check-in/out verification and billing (Increments 7–8).
- **Zein A. Sunoqrot** owns housekeeping and dashboard requirements (Increments 9–10).
- The requirements file is maintained collaboratively with one agreed version; change its IDs only when absolutely necessary and update the incremental plan, diagrams, implementation tests, and traceability matrix together.
- A requirement is ready to implement once its business rules, acceptance checks, and relevant diagrams are reviewed. No completed work or tests are claimed by this draft alone.
