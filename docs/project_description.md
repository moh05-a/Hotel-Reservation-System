# Hotel Reservation System — Project Description

## 1. Project Overview

The Hotel Reservation System is a web application for a single hotel. It helps the hotel manage room availability, reservations, guest records, check-in and check-out, billing, housekeeping, and basic operations.

Guests can search for and reserve a **room type** online without creating an account. Proceeding to payment creates a five-minute temporary hold; successful simulated payment is required to confirm the booking. Hotel staff use role-based accounts to manage the hotel's daily work.

## 2. Main Users

- **Guest:** Searches room types, places a five-minute Pending hold, completes simulated payment without an account, and uses booking reference + secret code to view, modify or cancel **one confirmed booking**.
- **Receptionist:** Searches and maintains guest records, manages reservations, checks a guest's identification at arrival, assigns a physical room, completes check-in/check-out, and handles billing.
- **Hotel Manager:** Manages room types, physical rooms, nightly rates, and views operational reports.
- **Housekeeping Staff:** Views and updates authorized room cleaning/operational statuses.
- **Simulated Payment Gateway:** Represents payment and refund outcomes within the application; it is not connected to real financial services.

## 3. Main Functionalities

- Manage room types, physical rooms, guest capacities, and nightly rates.
- Search available room types using stay dates, number of guests, and an optional room-type filter.
- Prevent overbooking by atomically checking inventory across overlapping Confirmed bookings and unexpired Pending holds.
- Create five-minute **Pending** room-type holds when guests proceed to payment, and confirm them only after successful simulated payment and a protected database commit; physical room assignment occurs at check-in.
- Issue a unique booking reference and separate unpredictable secret access code **only after successful simulated payment and confirmed booking**. Show the secret once; it grants access only to that booking.
- Allow guests to view, modify, or cancel their own bookings after providing both booking credentials; allow authorized receptionists to manage bookings through their staff accounts.
- Store guest profiles and link their reservations and stays to an internal guest ID, **even when the guest has no website account**.
- Allow receptionists to search for returning guests and view their past reservations/stays. Possible duplicate records are linked only after the receptionist checks they belong to the same person.
- At arrival, have the receptionist **manually verify presented identification** against the reservation before recording the identity check and completing check-in. Use a simulated check for the course demonstration; do not capture or upload real ID documents.
- Assign a suitable, ready physical room during check-in and manage check-out.
- Require a successful simulated booking payment and support folios/invoices, configurable tax, additional simulated charges, reversals and refunds, with no actual bank/card transactions.
- Track room cleaning and Out-of-Order status, and display hotel operating information in a dashboard.

## 4. Scope and Assumptions

- The system serves **one hotel** and uses demonstration guest/stay information.
- **No guest registration or login is required.** Creating a booking creates or links a guest record, not an online user account.
- Booking credentials are issued **only after successful simulated payment and committed confirmation**. Failed payment may be retried during the same five-minute hold, without resetting the deadline; expiry frees inventory.
- Guest booking credentials grant access only to their specific reservation. Hotel staff use authenticated accounts with assigned permissions.
- The secret access code is displayed to the guest at creation, stored as a secure hash, and never displayed later in plaintext. Guessing attempts are restricted.
- The hotel retains guest records separately from the guest's online access. Returning guests can have multiple linked reservations and stays.
- Receptionists carry out the physical identity check at check-in. The application records confirmation of that check, not real ID scans or third-party identity verification.
- Guests book a **room type**; the receptionist assigns a **physical room** at check-in.
- Nightly prices are per room type and snapshotted at confirmation; currency is **JOD**; tax is a configurable percentage. Standard hotel check-in is **14:00** and check-out is **12:00** local time.
- A Confirmed booking may be cancelled before scheduled check-in: **at least 24 hours before = 100% simulated refund**; less than 24 hours before = no refund. Eligible date changes need at least 24 hours, sufficient inventory and payment/refund of price differences.
- A Confirmed booking becomes `NoShow` if check-in has not occurred by **12:00 noon on the next day**, with no refund and no late-arrival exception. An unpaid Pending hold becomes `Expired` after its five-minute deadline.
- A **Pending hold** consumes room-type inventory for each night for **five minutes**. Creating the hold and confirming it are concurrency-safe. Success requires simulated payment; failure/expiry releases the hold.
- The intended date convention is **check-in inclusive, check-out exclusive**; the same rule is used by availability and booking logic.
- No real email/SMS notifications, bank/card transactions, or live hotel-platform integrations are required.

## 5. Out of Scope

- Guest accounts, guest-password login, and a guest-facing page listing their entire stay history.
- Real payments or storage of card details.
- Real SMS/email delivery or automatic secret-code recovery through those services.
- External hotel-booking platforms, multiple hotels, and real government-ID verification services.

## 6. Details to Define During Design

- Exact receptionist identity-verification steps for secure forgotten-code reissue (old code invalidated; no real SMS/email).
- Detailed implementation of the agreed 24-hour cancellation/modification rules and next-day-noon NoShow policy, including audit and reversal recovery.
- Exact demonstration tax rate, rounding, extra charges, and payment reconciliation with Increment 8; JOD and mandatory booking payment are agreed.
- Room-readiness and Out-of-Order rules, including what happens when a room becomes unavailable after reservations are confirmed.
- Procedures for resolving duplicate guest records and recognizing returning guests without merging people based only on matching names/contact details.
- Early/late check-out and payment-related fees, dashboard metric definitions, and performance-test conditions.

These details will be agreed on in the relevant design increments and reflected consistently in the requirements, diagrams, and tests.