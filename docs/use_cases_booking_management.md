# Increment 6 — Booking Viewing, Modification and Cancellation (`v0.6.0-design`)

**Owner:** Yousef Taha  
**Requirements:** `FR-RS-04–06`, `NFR-01`, `NFR-03`, `NFR-04`  
**Diagrams:** `uml/use_case_booking_management.puml`, `uml/seq_view_booking.puml`, `uml/seq_modify_booking.puml`, `uml/seq_cancel_booking.puml`  
**Existing references:** `docs/booking_rules.md`, `uml/state_reservation.puml`

## 1. Scope and access rules

- Guests do **not** create website accounts. They access **one booking** using its unique booking reference **and** matching secret code.
- Backend verification uses the stored code hash and rate-limits invalid attempts. A reference by itself grants no access; neither credential grants access to another booking or a guest's complete history.
- Authorized receptionists use their authenticated staff permissions instead of a guest's secret code.
- Guests may **view** a previously confirmed booking, including its later status, but cannot edit an ineligible booking.
- Booking credentials and sensitive guest information must not appear in logs or ordinary error responses. Record meaningful changes in the audit log.
- Booking modification does **not** create a new reservation or a second access code. Successful changes leave the reservation in `Confirmed` status.

## 2. UC-RS-04 — View a booking

| Field | Description |
|---|---|
| **Actor** | Guest; authorized Receptionist |
| **Precondition** | Booking exists. Guest has its reference and secret code; or Receptionist is signed in and authorized. |
| **Main flow** | 1. Guest enters reference and code. 2. Backend validates both and limits guesses. 3. System loads only the matching reservation. 4. It shows permitted booking details and current status. |
| **Alternatives** | Missing/wrong credentials, unknown reference or rate limit: deny access without exposing booking details. Receptionist access instead requires a verified staff role. |
| **Postcondition** | Booking unchanged. No access to other reservations or guest history. |

## 3. UC-RS-05 — Modify a booking

| Field | Description |
|---|---|
| **Actor** | Guest with matching reference/code; authorized Receptionist |
| **Precondition** | Booking is `Confirmed`; authorization and applicable change policy pass. |
| **Main flow** | 1. Actor requests changes. 2. Backend rechecks authorization, status and timing. 3. It validates contact information, guest count and/or new dates. 4. For a date change, it checks room-type inventory and calculates the new price. 5. If price increases, a successful simulated payment of the difference is required. 6. Backend locks and rechecks the reservation and relevant nightly inventory, then commits the change atomically. 7. If price decreases, it records/processes a simulated refund after the change commits. 8. Actor sees the updated booking. |
| **Alternatives** | Invalid request, wrong code, insufficient inventory, late date change, payment failure or database failure: reject and **keep the original booking**. Reverse/reconcile any successful additional payment that cannot be applied. |
| **Postcondition** | On success, original reservation identity and access code remain; new details, inventory, price snapshot and audit record are consistent. |

**Modification rules:**

- Date changes require **at least 24 hours before the original scheduled check-in** (14:00, hotel-local time); selected dates must also be valid and available.
- Check-in is inclusive, check-out exclusive. Inventory must be checked for **every requested night**, counting `Confirmed` bookings and unexpired `Pending` holds while excluding this reservation's own existing inventory from its availability comparison.
- Guest count must stay within the chosen room type's capacity. Updating permitted booking contact details does **not** grant permission to edit the guest's entire hotel profile.
- Guest-initiated room-type changes are **out of scope** for the first version.
- `CheckedIn`, `CheckedOut`, `Cancelled`, `Expired` and `NoShow` reservations cannot be modified through the guest booking page.
- A successful simulated extra payment is needed **before** committing a higher-priced date change; a lower price leads to a simulated refund after successful commit. Refund/reversal details are coordinated with Increment 8.
- A concurrent cancellation/modification must not silently overwrite another update. Recheck status and dates in the protected commit and reject stale requests.

## 4. UC-RS-06 — Cancel a booking

| Field | Description |
|---|---|
| **Actor** | Guest with matching reference/code; authorized Receptionist |
| **Precondition** | Booking is `Confirmed` and scheduled check-in has **not** started; guest is not checked in. |
| **Main flow** | 1. Actor requests cancellation. 2. Backend verifies access, status and cutoff using server time. 3. Backend atomically sets `Cancelled`, releases the reserved inventory and audits the change. 4. Apply the simulated refund policy and show the outcome. |
| **Alternatives** | Invalid credentials, ineligible status, after check-in time or duplicate cancellation: reject or return an idempotent prior result, without refunding twice. If simulated refund fails, keep cancellation and record a pending reconciliation. |
| **Postcondition** | Booking is `Cancelled`; released capacity is available, and refund outcome is recorded. |

**Cancellation/refund rules:**

- **At least 24 hours before** 14:00 on scheduled check-in date: **100% simulated refund** of eligible booking payments.
- **Less than 24 hours before**, but still **before 14:00 on check-in date**: cancellation allowed with **no refund**.
- **At or after 14:00** on check-in date: ordinary cancellation is not allowed.
- `NoShow` occurs at **12:00 noon the next day** if there was no check-in; there is **no refund** and no late-arrival exception.
- An unpaid `Pending` hold can be explicitly cancelled in the Increment 5 payment flow; it releases its inventory with no refund.

## 5. Integration and acceptance checks

1. A correct reference **and** code displays one booking only. Wrong/missing credentials and repeated guesses are rejected or throttled.
2. An authorized receptionist can find/manage bookings without using guest booking codes; unauthorized staff cannot.
3. View-only access works for a `Cancelled` booking but modification/cancellation does not.
4. A date change exactly **24 hours** before scheduled check-in is eligible; one minute later is not.
5. Changing to sold-out or overlapping dates fails without losing the original booking or its inventory.
6. Two concurrent updates cannot both silently succeed against stale reservation data.
7. A higher-price change without successful simulated difference payment fails; if payment succeeded but the change fails, reversal/reconciliation is recorded.
8. A lower-price change issues an appropriate simulated difference refund **after** successful commit.
9. Cancellation at least 24 hours before check-in gives full eligible refund; cancellation less than 24 hours before check-in but before 14:00 gives none.
10. Check-in, NoShow and past-check-in cancellation attempts are rejected; repeated cancellation never refunds twice.

**Implementation coordination:** Increment 8 owns detailed refund/reversal recording and rounding. Exact tax configuration and payment reconciliation steps remain shared design details. No new guest accounts, physical-room assignment or real payments are introduced in Increment 6.
