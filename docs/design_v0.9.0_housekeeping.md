# Increment 9 Design — Housekeeping & Room Readiness (`v0.9.0-design`)

**Owner:** Zein A. Sunoqrot  
**Requirements:** `FR-OP-01`, `FR-OP-02`, `FR-OP-03`  
**Diagram:** `uml/state_room.puml` (rendered in `uml/rendered/state_room.svg`)

## 1. Operational Room States
- **`Dirty`**: Default state immediately after guest check-out or maintenance completion. Cannot be assigned to guests.
- **`Clean`**: Cleaned by housekeeping staff, pending supervisor verification. Still cannot be assigned to guests.
- **`Inspected`**: Verified ready for occupancy. **Only** rooms in `Inspected` state (with `is_active = true` and no overlapping active stay) are eligible for check-in assignment.
- **`Occupied`**: Currently assigned to an active `CheckedIn` stay.
- **`Out-of-Order`**: Unavailable due to maintenance or damage; requires a mandatory reason.

## 2. Allowed State Transitions & Assignment Consistency
| From State | To State | Trigger / Actor | Validation & Assignment Rule |
|---|---|---|---|
| `Dirty` | `Clean` | `Housekeeping` / `Manager` | Standard cleaning completion |
| `Clean` | `Inspected` | `Supervisor` / `Manager` | Quality check passed; room becomes assignable |
| `Clean` | `Dirty` | `Supervisor` / `Housekeeping` | Failed inspection or room re-soiled before check-in |
| `Inspected` | `Occupied` | `Front Desk` (Check-in) | Allowed **only** from `Inspected` (`FR-OP-03`) |
| `Occupied` | `Dirty` | `System` (Check-out) | Automatic transition upon stay check-out |
| `Dirty` / `Clean` / `Inspected` | `Out-of-Order` | `Manager` / `Maintenance` | Mandatory `reason` and optional `expected_end_date` |
| `Out-of-Order` | `Dirty` | `Manager` / `Maintenance` | Must go through `Dirty -> Clean -> Inspected` before reassignment |

*Invalid transitions (e.g., `Dirty -> Inspected`, `Clean -> Occupied`, or `Out-of-Order -> Inspected`) are rejected with `400 Bad Request`.*

## 3. `Out-of-Order` Impact on Availability (Current vs. Future Scope)
- **Immediate Operational Impact:** Rooms in `Out-of-Order` status are strictly excluded from immediate check-in room assignment (`FR-OP-03`).
- **Future Availability & Existing Reservations (Open Design Coordination):** Current requirements leave the exact handling of future `Out-of-Order` periods unresolved. Specifically, whether setting a room to `Out-of-Order` immediately reduces future room-type inventory blocks, and how conflicting pre-existing `Confirmed` reservations are flagged or reassigned, remains open and requires cross-increment coordination with **Increment 3** (Availability & Reservation Management) and **Increment 7** (Check-in / Room Assignment).

## 4. Audit Model (`RoomStatusHistory`)
Every state transition—whether triggered manually by staff or automatically by the system—creates an immutable audit record in `RoomStatusHistory`:
- `id`, `room_id`, `from_status`, `to_status`, `changed_at` (UTC timestamp)
- `actor_type`: Enum (`STAFF` | `SYSTEM`)
- `changed_by_staff_id`: Foreign key to `Staff` (**required** when `actor_type = STAFF`; **`NULL`** when `actor_type = SYSTEM`, such as automatic `Occupied -> Dirty` transitions on check-out)
- `trigger_event` / `reason`: System event code (e.g., `AUTO_CHECKOUT`) or mandatory staff note when transitioning to `Out-of-Order`.
