# Increment 10 Design — Operations Dashboard (`v0.10.0-design`)

**Owner:** Zein A. Sunoqrot  
**Requirements:** `FR-OP-04`, `NFR-03`  
**Diagram:** `uml/seq_dashboard.puml` (rendered in `uml/rendered/seq_dashboard.svg`)

## 1. Access Control & Date Filters
- **Authorized Role:** `Hotel Manager` only (enforced server-side by FastAPI RBAC middleware; returns `403 Forbidden` for other roles).
- **Date Range Semantics:** Half-open interval `[start_date, end_date)` where `start_date` is inclusive and `end_date` is exclusive.
- **Default Period (`Today`):** When omitted, `start_date` defaults to **today** and `end_date` defaults to **tomorrow** (`today + 1 day`, representing 1 operational night/day).
- **Range Validation:** Any request where `end_date <= start_date` (including equal dates representing zero nights) is rejected with `400 Bad Request`.

## 2. Dashboard Metric Definitions & Sources
| Metric | Definition / Formula | Source Tables |
|---|---|---|
| **Occupancy Rate (%)** | `(Occupied Room-Nights in [start_date, end_date) / Available Active Room-Nights in [start_date, end_date)) * 100`. For historical or multi-day ranges, available active room-nights are derived from historical room status timelines (`Room` active flag + `RoomStatusHistory`) rather than current `Room.status` alone, excluding nights where a room was `Out-of-Order` or inactive on those dates. | `Stay`, `Room`, `RoomStatusHistory` |
| **Expected Arrivals** | Count of `Confirmed` reservations with `check_in_date` within `[start_date, end_date)` | `Reservation` |
| **Expected Departures** | Count of active `CheckedIn` stays with `check_out_date` within `[start_date, end_date)` | `Stay`, `Reservation` |
| **Recognized Revenue** | Sum of earned accommodation room-night charges and posted ancillary service charges accrued during `[start_date, end_date)` (net of charge credits/adjustments). Unearned advance deposits collected for future stays outside the period are excluded from recognized revenue. | `Folio`, `FolioCharge`, `Stay` |
| **Room Readiness Summary** | Real-time count of active physical rooms grouped across all five operational states coordinated with Increment 9 (`Inspected`, `Clean`, `Dirty`, `Occupied`, `Out-of-Order`) | `Room` |

## 3. API Endpoint Contract (Planned)
- **Endpoint:** `GET /api/dashboard/metrics?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`
- **Responses:**
  - `200 OK`: JSON object returning `occupancy_rate`, `expected_arrivals`, `expected_departures`, `recognized_revenue`, and `room_readiness` counts (`inspected`, `clean`, `dirty`, `occupied`, `out_of_order`).
  - `400 Bad Request`: Returned when `end_date <= start_date` or date format is invalid.
  - `403 Forbidden`: Returned when caller is not authenticated as `Hotel Manager`.
