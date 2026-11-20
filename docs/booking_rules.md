# Increment 5 — Reservation Creation: Booking and Security Rules

**Status:** Approved by the Increment 5 owner in discussion as working design decisions; pending teammate review and GitHub requirements/project-description updates.  
**Scope:** `v0.5.0-design` with linked policies for Increments 6–8.  
**Related requirements:** `FR-RS-01–03`, `FR-GS-01–03`, `FR-AV-01–03`, `NFR-01`, `NFR-03`, `NFR-04`.  
**Design sources:** `uml/seq_booking.puml` and `uml/state_reservation.puml`.

## 1. Purpose and boundaries

- A guest can reserve a **room type** without registering or signing in.
- A specific physical room number is assigned later at check-in (Increment 7).
- A **successful simulated payment is mandatory** to confirm a reservation. No real card payment, banking integration, or real card-data storage is involved.
- The guest receives a unique booking reference and a separate unpredictable secret access code after successful confirmation.
- A booking reference plus the matching secret code grants access to **that reservation only** (Increment 6), never to the guest's full history or staff features.

## 2. Input validation

Required inputs: room type, check-in date, check-out date, positive guest count, and required guest contact information.

- Check-out must be after check-in; check-in is inclusive and check-out is exclusive.
- Guest count cannot exceed the selected room type's maximum capacity.
- The selected room type must exist and be valid for the requested dates.
- Apply validation on the **backend**, even when the frontend validates the same fields.
- The booking workflow must not automatically merge/link an existing Guest record based only on a matching name, phone, or email. Returning guest identity linking requires receptionist verification (Increment 4).

## 3. Five-minute temporary hold (Pending)

1. The guest enters details and chooses **Next / Proceed to Payment**.
2. The backend validates the request and performs a **concurrency-safe inventory check** for every requested night.
3. If capacity exists, a `Pending` reservation/hold is persisted atomically, with `created_at` and `expires_at = created_at + 5 minutes` (server time).
4. A Pending hold consumes **one room-type inventory unit per booked night**. It does not reserve a specific room number.
5. Other availability searches count **Confirmed reservations and unexpired Pending holds**; holds that reached `expires_at` are treated as unavailable to their holder and do not count against capacity, even if cleanup has not run.
6. The frontend shows the remaining five-minute countdown, but only the **server** decides whether a hold is active.
7. Refreshing the page and retrying a payment **must not restart or extend** the deadline.
8. Repeated Next submissions for the same attempt should reuse the same hold using an idempotency/request identifier rather than making duplicates.
9. If the guest explicitly cancels during payment, mark the hold `Cancelled` and release inventory immediately. Closing the browser without an explicit cancellation leaves the hold until expiry.
10. If no successful confirmation is completed by the deadline, mark the hold `Expired` and release its inventory. Keep the expired record for audit/history, rather than deleting it.

## 4. Simulated payment attempts and booking confirmation

- During an active Pending hold, the guest may attempt payment **multiple times within the original five minutes**.
- Failed attempts are recorded, but the reservation remains `Pending`; the timer continues from its original deadline.
- At successful simulated payment, the backend **revalidates status and expiry under transactional protection** before changing the hold to `Confirmed`.
- Confirm only when payment succeeded, the hold is still valid, and the database transaction committing the confirmed reservation succeeds.
- If payment succeeds after the hold expires (or expires before confirmation), do **not** confirm the reservation. Record a **simulated reversal** or reconciliation case.
- If the confirmation database commit fails after a successful payment, do not report success; reverse the simulated payment or mark it for explicit reconciliation.
- Payment attempts should use idempotency identifiers so retried requests do not create duplicate successful charges or confirmations.
- Payment records must distinguish outcomes such as `Failed`, `Successful`, `Reversed`, and `Refunded`. Reversal failure or an unknown result must be recorded for reconciliation, not silently treated as success.
- The interface displays a booking reference and secret code **only after the confirmed reservation commits**.

## 5. Preventing overbooking

- Capacity is determined for **every night** of the stay and is not based on selecting a physical room number.
- Availability is reduced by active confirmed reservations and **unexpired Pending holds** for overlapping nights.
- The availability check and Pending hold creation must use a **database concurrency-control strategy** (e.g., appropriate locks, serializable transaction with retries, or equivalent per-room-type/night inventory protection).
- A plain read followed by an unrelated insert is **not** sufficient. Two concurrent requests for the last room must not both create holds.
- Confirmation atomically converts a valid Pending hold into Confirmed; it must not consume an additional room-type inventory unit.
- Expiration, cancellation, and confirmed booking changes must release inventory consistently.
- Exact locking implementation depends on the chosen database and must be tested under simultaneous requests.

## 6. Booking credentials and access control

- Generate a unique booking reference and a separate, cryptographically unpredictable secret access code upon confirmed booking.
- Show the secret code **once**; store only a secure one-way **hash** in the database, never its plaintext value.
- Do not include the code in ordinary logs, reports, notification logs, or subsequent API responses.
- Viewing/modifying/cancelling an existing booking requires the **matching reference and secret code** or authorized receptionist permissions; this is enforced on the backend.
- Apply rate limiting and anti-guessing controls to booking-code verification.
- Lost codes are reissued/reset only through **receptionist-verified identity**, with the old code invalidated. Detailed identity-check steps are coordinated with the team; no real SMS/email is assumed.

## 7. Pricing and timestamps

- Currency: **JOD** for the course demonstration.
- Standard hotel check-in time: **14:00 (2:00 PM)**, hotel local time.
- Standard check-out time: **12:00 noon**, hotel local time.
- Booking dates remain **check-in inclusive, check-out exclusive**.
- Save a **price snapshot** at confirmation, so later changes to room-type rates do not silently alter existing bookings.
- Tax is a **configurable rate**, using a team-approved demonstration value; exact tax/rounding and additional charge rules will be coordinated with Increment 8.
- For the 24-hour cancellation/modification rule, measure the cutoff against **14:00 on the scheduled check-in date** in hotel local time.

## 8. Cancellation policy (Increment 6 with Increment 8)

- A Confirmed reservation may be cancelled **before its scheduled check-in time (14:00)** and before check-in is completed.
- Cancellation at least **24 hours before scheduled check-in**: **100% simulated refund** of the eligible paid booking amount.
- Cancellation less than **24 hours before scheduled check-in**: **no refund**.
- After check-in, ordinary cancellation is not allowed; early check-out is a separate policy.
- On cancellation, change status to `Cancelled`, free remaining booked inventory and record the applicable simulated refund (if any).
- Cancellation of an unpaid `Pending` hold simply releases capacity immediately and does not generate a refund.
- Refund processing is simulated; even a zero-refund cancellation must release future inventory consistently.

## 9. No-show policy (Increments 6–7)

- Standard arrival/check-in starts at **14:00** on the scheduled check-in day.
- If the guest has **not checked in by 12:00 noon on the following day**, an eligible Confirmed booking becomes `NoShow` (subject to a staff-recorded exception made before the deadline).
- `NoShow` receives **no simulated refund** under the initial policy.
- Release any remaining future room-type inventory. Record the state change for audit purposes.
- The system may perform the deadline transition automatically, while allowing authorized receptionist handling of documented exceptions.
- The exact user interface and staff procedure for late-arrival exceptions are implementation details to agree before Increment 7.

## 10. Modification policy (Increment 6)

- A verified guest (booking reference + code) or authorized receptionist may modify an **eligible Confirmed** booking.
- Date changes are allowed **at least 24 hours before scheduled check-in**, provided the requested dates have enough available inventory.
- Contact details may be updated with the appropriate verification and permissions.
- Guest count can change only within the selected room type's capacity.
- Guest-initiated room-type changes are deferred from the initial version to reduce complexity.
- Guests cannot use the booking page to modify `CheckedIn`, `CheckedOut`, `Cancelled`, `Expired`, or `NoShow` bookings.
- Date changes must atomically update inventory without losing the original booking if the new dates are unavailable.
- If the revised booking costs more, require a successful **simulated additional payment** of the difference before confirming the change.
- If it costs less, record a **simulated refund** of the difference after the eligible change is committed.
- If the modification fails, preserve the original reservation; reconcile/reverse any simulated additional payment as needed.

## 11. Anti-abuse protection and auditing

- Apply **rate limiting** on creating temporary holds and attempting payments or secret-code verification.
- Limit the number of **active Pending holds per session**; the first implementation should allow **one active hold per session** and reuse it for duplicate Next requests. Also enforce configurable server-side limits to reduce repeated holds from different sessions.
- Use request/attempt identifiers to prevent accidental duplicates from refreshes, retries and double-clicks.
- Audit meaningful reservation, hold, verification, payment and staff status changes, without logging secret codes or sensitive identity documents.
- Limits must not rely only on the browser: they are checked server-side.

## 12. State lifecycle summary

`Pending` (5-minute hold) → `Confirmed` (successful payment + valid hold + DB commit) → `CheckedIn` → `CheckedOut`.

Alternative transitions:

- `Pending` → `Expired`: five-minute deadline reached, inventory released.
- `Pending` → `Cancelled`: guest explicitly cancels payment, inventory released.
- `Pending` → `Pending`: simulated payment fails; a retry is permitted within the original deadline.
- `Confirmed` → `Cancelled`: policy-eligible cancellation, refund policy applied, inventory released.
- `Confirmed` → `NoShow`: next-day noon deadline reached without check-in, no refund, future inventory released.

`PaymentFailed` is not a reservation state in this model; it is an **outcome of an attempt**, while the reservation stays Pending.

## 13. Coordination and implementation checks

The business-policy choices above are accepted by the Increment 5 owner as working design decisions, pending team review. The following integration details should be checked during implementation and teammate review rather than silently assumed:

- **Documentation synchronization:** existing `docs/requirements.md`, `docs/project_description.md` and `docs/incremental_development_plan.md` on `main` currently describe optional payment; revise through a separate reviewed PR as agreed.
- **Ownership:** Increment 5 needs the basic simulated payment outcome; coordinate with Mohammad's Increment 8 for payment records, reversals, invoices and refunds.
- **Inventory:** coordinate the availability and Out-of-Order room policies with Anas's Increment 3 and Zein's room-status work.
- **Guest identity:** coordinate guest linking/reissue identity checks with Increment 4 and receptionist permissions.
- **Background expiry:** an expiry worker can mark expired holds, but availability must ignore them immediately based on server time even if the worker is delayed.
- **Concurrency:** choose and test an appropriate strategy for the project's actual database and transactions.
- **Operational edge cases:** consider last-second payment, duplicate attempts, lost browser session, server failure, partial simulated reversals, one-night reservations reaching the NoShow cutoff, and reported time-zone consistency.
- **Pricing:** agree the demonstration tax value, exact currency rounding and invoice/refund records before payment/billing implementation.

## 14. Suggested design acceptance checks

1. Valid request creates exactly one Pending hold and reduces availability for its nights for five minutes.
2. Another simultaneous request cannot take the last available inventory while it is held.
3. Failed simulated payment permits retry without resetting the deadline.
4. Successful payment before expiry confirms exactly one reservation and issues a reference + one-time code.
5. Expiry releases capacity and prevents a late payment result from confirming the expired hold.
6. Failed database commit cannot produce a false confirmation; any successful simulated payment is reversed/reconciled.
7. A canceled Pending hold releases inventory immediately.
8. Cancelling a Confirmed booking at the 24-hour boundary applies the agreed refund policy.
9. No-show at the next-day-noon deadline records NoShow and releases remaining future inventory.
10. Unauthorized viewing, repeated code guesses and duplicate booking attempts are rejected or throttled.
