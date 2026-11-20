# Increment 10 Design — Operations Dashboard (`v0.10.0-design`)

**Owner:** Zein A. Sunoqrot  
**Requirements:** `FR-OP-04`, `NFR-03`  
**Diagram:** `uml/seq_dashboard.puml` (rendered in `uml/rendered/seq_dashboard.svg`)

## 1. Access Control & Filters
- **Authorized Role:** `Hotel Manager` only (enforced server-side by FastAPI RBAC middleware).
- **Date Filter:** `start_date` to `end_date` (inclusive of start, exclusive of end for nights; defaults to **Today**). Invalid ranges where `end_date` is before `start_date` return `400 Bad Request`.

## 2. Dashboard Metric Definitions & Sources
| Metric | Definition / Formula | Source Tables |
|---|---|---|
| **Occupancy Rate (%)** | `(Occupied Room-Nights in period / Available Active Room-Nights in period) * 100` (excludes inactive and Out-of-Order rooms from denominator) | `Stay`, `Room` |
| **Expected Arrivals** | Count of `Confirmed` reservations with `check_in_date` within selected period | `Reservation` |
| **Expected Departures** | Count of active `CheckedIn` stays with `check_out_date` within selected period | `Stay`, `Reservation` |
| **Recognized Revenue** | Sum of successful `Payment` amounts minus completed `Refund` amounts recorded within selected period | `Folio`, `Payment`, `Refund` |
| **Room Readiness Summary** | Real-time count of active physical rooms grouped by status (`Inspected`, `Clean`, `Dirty`, `Out-of-Order`) | `Room` |

## 3. API Endpoint Contract (Planned)
- **Endpoint:** `GET /api/dashboard/metrics?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`
- **Response:** JSON object returning `occupancy_rate`, `expected_arrivals`, `expected_departures`, `recognized_revenue`, and `room_readiness` counts.
