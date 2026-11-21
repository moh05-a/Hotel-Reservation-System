# Increment 3 — Availability Search Design

## 1. Overview

This document describes how the Hotel Reservation System searches for available room types.

Guests can search using check-in and check-out dates, the number of guests, and an optional room-type filter. The system must return only room types that can accommodate the guests and have enough inventory throughout the requested stay.

This design also accounts for Confirmed reservations and unexpired five-minute Pending holds.

**Requirements:** FR-AV-01, FR-AV-02, FR-AV-03, NFR-02.

## 2. Search Inputs

| Input | Description |
|---|---|
| check_in | Requested check-in date |
| check_out | Requested check-out date |
| guest_count | Number of guests |
| room_type_id | Optional filter for a specific room type |

### Validation Rules

- Check-in and check-out must be valid dates.
- Check-out must be later than check-in.
- Guest count must be a positive integer.
- A supplied room-type filter must refer to an existing room type.
- Invalid inputs must be rejected with an appropriate error.

Any additional restrictions on past dates or maximum stay length will be agreed during implementation.

## 3. Room Type and Capacity

Guests reserve a Room Type rather than a specific physical room.

Each RoomType has a maximum guest capacity and a collection of physical rooms.

For example:

| Room Type | Maximum Guests | Physical Rooms |
|---|---:|---:|
| Standard | 2 | 5 |
| Deluxe | 3 | 3 |
| Suite | 5 | 2 |

A room type is excluded when its maximum guest capacity is less than the requested guest count.

Physical rooms are assigned later during check-in, not during the availability search.

## 4. Availability Calculation

Availability is calculated for every night between the requested check-in and check-out dates.

The check-in date is inclusive, while the check-out date is exclusive.

For each room type and night, the conceptual calculation is:

**Available Rooms = Eligible Room Inventory − Occupied Reservation Inventory − Active Pending Holds**

Here, occupied reservation inventory means nights already committed to inventory-consuming reservations. Confirmed reservations must be counted, and checked-in stays must remain accounted for where applicable.

An active Pending hold consumes one room-type unit when its expiration time is still in the future.

The final number of rooms available for the whole stay is the lowest nightly availability across the requested date range.

A room type is returned only when that number is at least one and its guest capacity is sufficient.

The final treatment of Out-of-Order rooms and future operational blocks requires team agreement.

## 5. Reservation Date Overlap

Two reservation date ranges overlap when:

**Existing Check-In < Requested Check-Out**

AND

**Existing Check-Out > Requested Check-In**

This follows the check-out-exclusive convention.

For example:

- Existing stay: November 20–23
- Requested stay: November 23–25

These stays do not overlap because the first guest checks out on the same date the second guest checks in.

If the requested stay were November 22–25, the stays would overlap.

## 6. Five-Minute Pending Holds

When a guest proceeds to simulated payment, Increment 5 creates a Pending reservation hold lasting five minutes.

During that period, the hold reduces room-type availability for every night of the requested stay.

The following rules apply:

1. Only unexpired Pending holds consume inventory.
2. A failed simulated payment does not extend the original expiration deadline.
3. A cancelled or expired Pending hold no longer reduces availability.
4. Availability must compare `expires_at` with the current server time.
5. An expired hold must stop affecting availability even if a background cleanup task has not updated its database status.

Increment 3 designs the search and availability calculation. The actual creation, expiration, and concurrency-safe confirmation of holds belong to Increment 5.

## 7. Example Availability Calculation

Assume the Deluxe Room Type contains five eligible physical rooms.

For a particular night:

| Inventory Component | Number |
|---|---:|
| Eligible physical rooms | 5 |
| Existing Confirmed reservations | 3 |
| Active Pending holds | 1 |
| Remaining available rooms | 1 |

The system should return Deluxe as available if its capacity also meets the guest count and at least one unit remains available on every requested night.

If another night in the requested range has zero remaining inventory, Deluxe must be excluded from the search results.

## 8. Search Results

A successful search returns the available room types and relevant information.

Suggested result fields include:

| Field | Description |
|---|---|
| room_type_id | Room type identifier |
| name | Room type name |
| description | Room type description |
| max_guests | Maximum guest capacity |
| nightly_rate | Current nightly rate in JOD |
| available_units | Minimum available inventory across the requested nights |

The guest may use the results to choose a room type and proceed toward booking.

Search results are informational. They do not reserve inventory. Increment 5 must recheck capacity atomically when creating a Pending hold.

## 9. Role-Based Access

Availability search is accessible to guests without registration or staff login.

The backend validates search inputs and calculates inventory. The frontend must not be trusted to determine final availability.

Staff may also use availability search when assisting guests.

## 10. Dependencies on Other Increments

**Increment 2 — Room Management:** Provides RoomType, Room, guest capacity, nightly rate, and physical-room inventory.

**Increment 5 — Reservation Creation:** Provides Pending holds, Confirmed reservations, expiration deadlines, and concurrency-safe booking operations.

**Increment 6 — Booking Management:** Reservation modifications and cancellations change inventory consumption.

**Increment 7 — Check-in/Check-out:** Check-in changes reservation/stay states but must not incorrectly release inventory for occupied nights.

**Increment 9 — Housekeeping:** Provides operational room statuses and rules for unavailable rooms.

## 11. Open Design Decisions

The following topics require coordination with the team:

- How future Out-of-Order periods affect room-type inventory.
- How planned room deactivation affects existing reservations.
- Exact treatment of inventory after NoShow and early check-out.
- Reservation status handling after check-in.
- Performance-testing workload for the two-second response target.

These details must be resolved before implementing the affected functionality.

## 12. Acceptance Criteria

The availability design is ready for review when:

1. Search inputs and validation rules are documented.
2. Room-type guest capacity is checked.
3. Inventory is calculated across every requested night.
4. Confirmed reservations reduce availability.
5. Unexpired Pending holds reduce availability.
6. Expired and cancelled holds do not reduce availability.
7. Adjacent non-overlapping stays are supported.
8. Invalid dates and nonpositive guest counts are rejected.
9. The Sequence Diagram and scenario tests reflect the same rules.
10. Out-of-Order inventory assumptions are identified for team review.

The working search API, user interface, and automated tests will be implemented in `v0.3.0`.