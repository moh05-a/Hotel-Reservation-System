# Increment 2 — Room Management Design

## 1. Overview

This document describes the room management design for the Hotel Reservation System.

The system allows the Hotel Manager to manage room types, physical rooms, guest capacities, and nightly rates. This increment provides the room information required for availability searches, reservations, check-in, and housekeeping.

**Requirements:** FR-RM-01, FR-RM-02, FR-RM-03, FR-RM-04, FR-RM-05.

## 2. Room Types

A Room Type represents a category of rooms that share the same guest capacity and nightly rate.

For example, a hotel may offer Standard, Deluxe, and Suite room types.

Each room type contains:

| Attribute | Description |
|---|---|
| room_type_id | Unique identifier |
| name | Room type name |
| description | Description of the room type |
| max_guests | Maximum number of guests allowed |
| nightly_rate | Price per night in JOD |

The Hotel Manager can create, view, and update room types.

Guest capacity must be a positive integer. The nightly rate must be nonnegative.

## 3. Physical Rooms

A Physical Room represents an actual hotel room with its own room number.

Each physical room belongs to one Room Type, while a Room Type can have multiple physical rooms.

Each physical room contains:

| Attribute | Description |
|---|---|
| room_id | Unique identifier |
| room_number | Unique hotel room number |
| room_type_id | Reference to the associated Room Type |
| is_active | Indicates whether the room is active |
| housekeeping_status | Current cleaning or operational condition |

The Hotel Manager can add, view, update, and safely deactivate rooms.

Physical rooms are not assigned when a guest creates a reservation. A suitable room is assigned later during check-in.

## 4. Room Management Rules

The system follows these rules:

1. Only the Hotel Manager can create or update room types, physical rooms, and nightly rates.
2. Authorized hotel staff can view room information required for their responsibilities.
3. Each room must have a unique room number.
4. Every physical room must reference a valid room type.
5. Guest capacity must be greater than zero.
6. Nightly rates cannot be negative.
7. Room deactivation must not invalidate existing reservations or active stays.
8. Inactive rooms cannot be assigned during check-in.
9. Room type changes must preserve existing data relationships.
10. Reservations are associated with room types rather than physical rooms until check-in.

## 5. Nightly Rates

Each Room Type has a nightly rate expressed in Jordanian Dinars (JOD).

The Hotel Manager can update this rate.

When a reservation is confirmed, the applicable price is saved with the reservation so that later changes to the Room Type rate do not unexpectedly change an existing confirmed booking.

The detailed tax calculation and payment process belong to later increments.

## 6. Role-Based Permissions

| Function | Hotel Manager | Receptionist | Housekeeping |
|---|---|---|---|
| View room types | Yes | Yes | Yes |
| View physical rooms | Yes | Yes | Yes |
| Create or update room types | Yes | No | No |
| Create or update physical rooms | Yes | No | No |
| Update nightly rates | Yes | No | No |
| Deactivate physical rooms | Yes | No | No |
| Update housekeeping status | No | No | Yes |

Housekeeping status updates will be implemented in Increment 9. This increment only defines the shared room data needed for that feature.

All permissions must be enforced by the backend.

## 7. Relationship Between Room Type and Room

The database uses a one-to-many relationship:

**RoomType (1) → (Many) Room**

One Room Type can contain multiple physical rooms, but each physical room belongs to exactly one Room Type.

For example:

- Deluxe Room Type: Rooms 201, 202, and 203.
- Standard Room Type: Rooms 101, 102, 103, and 104.

The number of eligible physical rooms helps determine room-type inventory.

## 8. Validation and Data Integrity

The system must reject invalid operations, including:

- Creating a room type with zero or negative guest capacity.
- Entering a negative nightly rate.
- Creating duplicate room numbers.
- Assigning a physical room to a nonexistent room type.
- Removing or changing room records in ways that create invalid references.
- Deactivating a room when doing so would violate existing stay or reservation constraints.

Permanent deletion is avoided where it could destroy historical relationships.

## 9. Dependencies on Other Increments

**Increment 1 — Authentication:** Provides staff accounts, authentication, and permission checks.

**Increment 3 — Availability:** Uses room types and eligible physical-room inventory to calculate availability for requested dates. Confirmed reservations and unexpired five-minute Pending holds reduce availability.

**Increment 5 — Reservations:** Creates room-type reservations without assigning specific physical rooms.

**Increment 7 — Check-in/Check-out:** Assigns eligible physical rooms during check-in.

**Increment 9 — Housekeeping:** Maintains the cleaning and operational status of physical rooms.

## 10. Open Design Decisions

The following details require coordination with other increment owners:

- How Out-of-Order room periods reduce future availability.
- Which housekeeping statuses make rooms ready for assignment.
- How room deactivation affects future confirmed reservations and Pending holds.
- Which room-type changes are allowed when reservations already exist.

These decisions will be finalized with the relevant team members before implementing the affected features.

## 11. Acceptance Criteria

The design will be ready for review when:

1. RoomType and Room attributes are defined.
2. Their one-to-many relationship is documented.
3. Manager permissions are identified.
4. Capacity, rate, and uniqueness rules are defined.
5. Unsafe room changes and deactivation are addressed.
6. The Class Diagram and ERD represent the agreed entities.
7. The design is consistent with future availability, reservation, and housekeeping requirements.

The working implementation and automated tests will be completed in the working version of Increment 2.