# Hotel Reservation System — Project Description

## 1. Project Overview

The Hotel Reservation System is a web application for a single hotel. It helps the hotel manage room availability, reservations, guest records, check-in and check-out, billing, housekeeping, and basic operations.

Guests can search for and reserve a **room type** online without creating an account. Hotel staff use role-based accounts to manage the hotel's daily work.

## 2. Main Users

- **Guest:** Searches available room types, creates a reservation without an account, and views, modifies, or cancels **one of their own bookings** using its booking reference and separate secret access code.
- **Receptionist:** Searches and maintains guest records, manages reservations, checks a guest's identification at arrival, assigns a physical room, completes check-in/check-out, and handles billing.
- **Hotel Manager:** Manages room types, physical rooms, nightly rates, and views operational reports.
- **Housekeeping Staff:** Views and updates authorized room cleaning/operational statuses.
- **Simulated Payment Gateway:** Represents payment and refund outcomes within the application; it is not connected to real financial services.

## 3. Main Functionalities

- Manage room types, physical rooms, guest capacities, and nightly rates.
- Search available room types using stay dates, number of guests, and an optional room-type filter.
- Prevent overbooking by checking room-type inventory across overlapping reservation dates.
- Create and confirm reservations without assigning physical room numbers at booking time.
- Issue a unique **booking reference** and a separate, unpredictable **secret access code** when a reservation is created. These credentials permit access to that booking only, not to the guest's complete history.
- Allow guests to view, modify, or cancel their own bookings after providing both booking credentials; allow authorized receptionists to manage bookings through their staff accounts.
- Store guest profiles and link their reservations and stays to an internal guest ID, **even when the guest has no website account**.
- Allow receptionists to search for returning guests and view their past reservations/stays. Possible duplicate records are linked only after the receptionist checks they belong to the same person.
- At arrival, have the receptionist **manually verify presented identification** against the reservation before recording the identity check and completing check-in. Use a simulated check for the course demonstration; do not capture or upload real ID documents.
- Assign a suitable, ready physical room during check-in and manage check-out.
- Generate folios/invoices, calculate tax, and record simulated payments and refunds.
- Track room cleaning and Out-of-Order status, and display hotel operating information in a dashboard.

## 4. Scope and Assumptions

- The system serves **one hotel** and uses demonstration guest/stay information.
- **No guest registration or login is required.** Creating a booking creates or links a guest record, not an online user account.
- A guest receives the booking reference and secret access code upon successful booking creation/confirmation. **Payment is not required before issuing the code**; simulated payment is handled in the billing workflow.
- Guest booking credentials grant access only to their specific reservation. Hotel staff use authenticated accounts with assigned permissions.
- The secret access code is displayed to the guest at creation, stored as a secure hash, and never displayed later in plaintext. Guessing attempts are restricted.
- The hotel retains guest records separately from the guest's online access. Returning guests can have multiple linked reservations and stays.
- Receptionists carry out the physical identity check at check-in. The application records confirmation of that check, not real ID scans or third-party identity verification.
- Guests book a **room type**; the receptionist assigns a **physical room** at check-in.
- Nightly prices are defined per room type, and tax is a fixed configurable percentage.
- Reservations and payments are separate: reservations may be confirmed before a simulated payment is recorded. Cancellation/refund rules will be kept simple.
- The intended date convention is **check-in inclusive, check-out exclusive**; the same rule is used by availability and booking logic.
- No real email/SMS notifications, bank/card transactions, or live hotel-platform integrations are required.

## 5. Out of Scope

- Guest accounts, guest-password login, and a guest-facing page listing their entire stay history.
- Real payments or storage of card details.
- Real SMS/email delivery or automatic secret-code recovery through those services.
- External hotel-booking platforms, multiple hotels, and real government-ID verification services.

## 6. Details to Define During Design

- How a forgotten guest access code can be **securely reissued** after staff verification, without email/SMS delivery.
- Final cancellation, modification, no-show, and refund policies.
- Final pricing details, currency, tax rounding, and timing of optional simulated payments.
- Room-readiness and Out-of-Order rules, including what happens when a room becomes unavailable after reservations are confirmed.
- Procedures for resolving duplicate guest records and recognizing returning guests without merging people based only on matching names/contact details.
- Early/late check-out and payment-related fees, dashboard metric definitions, and performance-test conditions.

These details will be agreed on in the relevant design increments and reflected consistently in the requirements, diagrams, and tests.