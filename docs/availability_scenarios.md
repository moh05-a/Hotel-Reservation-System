# Increment 3 — Availability Search Scenarios

## 1. Overview

This document defines test scenarios for the Availability Search feature of the Hotel Reservation System.

The scenarios cover room capacity, overlapping reservations, five-minute Pending holds, expired holds, date validation, and availability across the entire stay.

**Requirements:** FR-AV-01, FR-AV-02, FR-AV-03, NFR-02.

These are design-stage scenarios. Automated testing will be implemented in the working version of Increment 3.

## 2. Sample Hotel Inventory

The following inventory is used for the examples.

| Room Type | Maximum Guests | Physical Rooms |
|---|---:|---:|
| Standard | 2 | 3 |
| Deluxe | 4 | 2 |
| Suite | 5 | 1 |

Unless stated otherwise, all physical rooms are active and eligible for inventory calculations, and the scenarios are independent of one another.

The final handling of Out-of-Order rooms is still subject to team agreement.

## 3. Availability Test Scenarios

| ID | Scenario | Expected Result |
|---|---|---|
| AV-01 | Search with valid dates and sufficient inventory | Return available room types |
| AV-02 | Guest count exceeds room-type capacity | Exclude room types with insufficient capacity |
| AV-03 | All rooms of a type are already Confirmed for an overlapping night | Exclude that room type |
| AV-04 | Existing booking ends on the requested check-in date | Allow availability because dates do not overlap |
| AV-05 | Existing booking partially overlaps requested dates | Reduce inventory on overlapping nights |
| AV-06 | An unexpired Pending hold overlaps requested dates | Count the hold as occupied inventory |
| AV-07 | A Pending hold has reached its five-minute expiration time | Do not count the expired hold |
| AV-08 | A Pending hold was explicitly cancelled | Do not count the cancelled hold |
| AV-09 | A room type is sold out on only one night of the requested stay | Exclude it from the results |
| AV-10 | Check-out date is equal to or earlier than check-in | Reject the request |
| AV-11 | Guest count is zero or negative | Reject the request |
| AV-12 | Optional room-type filter is provided | Return only matching room types when available |
| AV-13 | All room types are unavailable | Return an empty availability result |
| AV-14 | A search is repeated without proceeding to payment | Do not create a Pending hold |

## 4. Detailed Examples

### Scenario A — Confirmed Reservations

Assume Standard has three physical rooms.

A guest searches for December 10–12, 2026.

Two Standard rooms are already reserved for both nights.

**Expected result:** Standard remains available with one room-type unit.

### Scenario B — Adjacent Reservations

An existing Standard reservation covers December 10–12, 2026.

Another guest searches for December 12–14, 2026.

Because check-out is exclusive, these reservations do not overlap.

**Expected result:** The existing reservation does not reduce availability for December 12–14.

### Scenario C — Active Pending Hold

Standard has three physical rooms.

For the requested nights, two rooms have Confirmed reservations and one room has an unexpired Pending hold.

**Expected result:** Standard is unavailable because all three units are consumed.

### Scenario D — Expired Pending Hold

Use the same inventory as Scenario C, but assume the Pending hold has expired.

**Expected result:** The expired hold no longer consumes inventory, so one Standard unit becomes available.

This must work even if the background cleanup process has not yet changed the hold's stored status to Expired.

A Pending hold is active only when its expiration time is strictly later than the current server time.

### Scenario E — Availability Across Multiple Nights

Standard has three rooms.

A guest searches for December 10–13, 2026.

| Night | Available Standard Rooms |
|---|---:|
| December 10 | 2 |
| December 11 | 0 |
| December 12 | 1 |

**Expected result:** Standard is excluded because no rooms are available on December 11.

Availability for the entire stay is based on the lowest nightly availability, not the average.

### Scenario F — Room Capacity

A guest searches for five people.

- Standard supports two guests.
- Deluxe supports four guests.
- Suite supports five guests.

**Expected result:** Only Suite qualifies by capacity, provided it has inventory for the entire stay.

## 5. Date Overlap and Expiration Rules

Two booking intervals overlap when:

`existing_check_in < requested_check_out`

AND

`existing_check_out > requested_check_in`

Check-in is inclusive and check-out is exclusive.

For Pending holds, the availability calculation uses the server time:

- `Pending` and `expires_at > server_time`: consumes inventory.
- `Pending` and `expires_at <= server_time`: does not consume inventory.
- `Expired` or `Cancelled`: does not consume inventory.
- `Confirmed`: consumes inventory for its applicable reserved nights.

A reservation must not be counted twice when it changes from Confirmed to CheckedIn. Applicable occupied nights must remain accounted for.

## 6. Performance Requirement

**NFR-02** targets a response time below two seconds under an agreed demonstration workload.

The test dataset size, concurrency level, and measurement conditions will be defined before working-version testing.

## 7. Dependencies and Open Decisions

These scenarios depend on the RoomType and Room models from Increment 2.

Increment 5 will handle concurrency-safe Pending hold creation and booking confirmation. Availability search does not create or extend holds.

The following topics still require team coordination:

- Future Out-of-Order room blocks and their inventory effects.
- Inventory handling following NoShow or early check-out.
- Final definition of the performance demonstration workload.

## 8. Acceptance Criteria

The scenarios cover the required availability behaviors:

- Guest capacity filtering.
- Confirmed reservation overlap.
- Adjacent and partially overlapping reservations.
- Active, expired, and cancelled Pending holds.
- Availability across every night.
- Invalid dates and guest counts.
- Optional room-type filtering.
- Search without creating a reservation hold.

The corresponding automated tests will be implemented in `v0.3.0`.