# Increment 9 Design — Housekeeping & Room Operational Status (`v0.9.0-design`)

**Owner:** Zein A. Sunoqrot  
**Requirements:** `FR-OP-01`, `FR-OP-02`, `FR-OP-03`, `NFR-03`, `NFR-04`  
**Diagram:** `uml/state_room.puml` (rendered in `uml/rendered/state_room.svg`)

## 1. Room Operational States
A physical room's housekeeping/operational status is independent of its reservation state and future bookings:
- **Dirty:** Room requires cleaning (automatically triggered on guest check-out or after maintenance).
- **Clean:** Cleaning completed by Housekeeping Staff, awaiting final inspection.
- **Inspected:** Verified clean and ready for guest assignment at check-in.
- **Out-of-Order:** Unavailable due to maintenance or damage; blocked from assignment.

## 2. Allowed State Transitions & Roles
| From State | To State | Allowed Roles | Trigger / Condition |
|---|---|---|---|
| `Dirty` | `Clean` | Housekeeping Staff, Hotel Manager | Cleaning completed |
| `Clean` | `Inspected` | Housekeeping Staff, Hotel Manager | Inspection passed |
| `Clean` | `Dirty` | Housekeeping Staff, Hotel Manager | Inspection failed / re-soiled |
| `Inspected` / `Clean` | `Dirty` | System / Receptionist | Guest check-out (`FR-CI-03`) |
| Any active state | `Out-of-Order` | Housekeeping Staff, Hotel Manager | Maintenance issue reported (requires reason) |
| `Out-of-Order` | `Dirty` | Hotel Manager, Housekeeping Staff | Repair finished (must be cleaned before assignment) |

*Prohibited transitions* (e.g., `Dirty` directly to `Inspected`, or `Out-of-Order` directly to `Inspected`) are rejected server-side with HTTP `400 Bad Request`.

## 3. Readiness & Check-in Rules (`FR-OP-03`)
- A physical room may be assigned at check-in **only if** its status is `Inspected`, it is active (`is_active = true`), and it has no overlapping active stay.
- Rooms in `Out-of-Order`, `Dirty`, or `Clean` (uninspected) states are blocked from check-in assignment.
- `Out-of-Order` rooms reduce effective room-type inventory for availability checks (coordinated with Increments 3 and 7).

## 4. Change-History Data Model (`FR-OP-02`, `NFR-04`)
Every status change records an immutable entry in `RoomStatusHistory`:
- `id` (PK)
- `room_id` (FK to `Room`)
- `previous_status` (Enum)
- `new_status` (Enum)
- `changed_by_staff_id` (FK to `StaffUser`)
- `reason` (Optional text; required when moving to `Out-of-Order`)
- `changed_at` (UTC timestamp)
