# Increment 4 — Guest Records and History: Use Case Descriptions

**Group:** 15  
**Project:** Hotel Reservation System  
**Increment:** `v0.4.0-design` (draft; not yet reviewed or tagged)  
**Primary owner:** Yousef Taha  
**Source requirements:** `docs/requirements.md`, FR-GS-01–04  
**Related diagram:** `uml/use_case_guest.puml`

## UC-GS-01 — Search Guest Records

| Field | Description |
|---|---|
| Primary actor | Receptionist |
| Requirement | FR-GS-03 |
| Goal | Find an existing guest record using permitted identifying information. |
| Preconditions | The receptionist is authenticated and authorized to search guest records. |
| Trigger | The receptionist opens guest search. |

**Main flow**
1. The receptionist opens Guest Management.
2. The receptionist enters supported information such as name, phone, email, or reservation reference.
3. The system validates the search input and searches guest records.
4. The system displays matching guest records.
5. The receptionist selects a candidate record for inspection.

**Alternative flows**
- No matches: the system reports no results.
- Multiple matches: the system presents candidate records without automatically linking or merging them.
- Invalid input: the system requests a correction.
- Unauthorized requester: the system denies access.

**Postconditions:** No guest records are altered by the search.

## UC-GS-02 — View Guest Profile

| Field | Description |
|---|---|
| Primary actor | Receptionist |
| Requirement | FR-GS-02 |
| Goal | View the permitted information stored for a guest. |
| Preconditions | The receptionist is authenticated and authorized; the guest record has been selected. |
| Trigger | The receptionist opens a selected guest profile. |

**Main flow**
1. The receptionist selects a guest record.
2. The system retrieves that guest's permitted profile information.
3. The system displays the profile for review.

**Alternative flows**
- The record does not exist: the system reports that it cannot be found.
- The requester is unauthorized: the system denies access.
- Data retrieval fails: the system shows an error without exposing other guest records.

**Postconditions:** The profile is viewed, with no changes to stored records.

## UC-GS-03 — Update Guest Profile

| Field | Description |
|---|---|
| Primary actor | Receptionist |
| Requirement | FR-GS-02 |
| Goal | Modify permitted guest contact/profile information. |
| Preconditions | The receptionist is authenticated, authorized, and has selected an existing guest record. |
| Trigger | The receptionist selects Edit Guest Profile. |

**Main flow**
1. The system displays editable fields of the selected guest profile.
2. The receptionist changes the permitted information and submits it.
3. The system validates the new values and the requester's authorization.
4. The system saves valid changes and confirms the update.

**Alternative flows**
- Invalid values: changes are rejected with field-level guidance.
- Unauthorized action: the system denies the update.
- Missing record or save failure: the system reports failure and does not falsely confirm an update.

**Postconditions:** On success, permitted changes persist. Linked reservations and stays remain attached to the correct guest. Contact changes never cause automatic record merging.

## UC-GS-04 — View Guest History

| Field | Description |
|---|---|
| Primary actor | Receptionist |
| Requirement | FR-GS-04 |
| Goal | View reservations and completed stays associated with a guest. |
| Preconditions | The receptionist is authenticated and authorized; a guest record has been selected. |
| Trigger | The receptionist selects Guest History. |

**Main flow**
1. The receptionist opens the selected guest's history.
2. The system retrieves reservations linked to the selected internal `guest_id` and related completed stays.
3. The system displays permitted dates, statuses, and stay details.
4. The receptionist reviews the history.

**Alternative flows**
- No linked history: the system displays an empty-state message.
- Unauthorized access: the system denies the request.
- Retrieval error: the system reports failure without showing another guest's information.

**Postconditions:** No records are modified. Booking reference and access-code credentials alone cannot expose a guest's entire history.

## UC-GS-05 — Link Returning Guest

| Field | Description |
|---|---|
| Primary actor | Receptionist |
| Requirements | FR-GS-01, FR-GS-03 |
| Goal | Associate a reservation with a verified existing guest record. |
| Preconditions | The receptionist is authenticated and authorized; a relevant booking is being handled and candidate existing guest records can be inspected. |
| Trigger | The receptionist chooses to link a booking to a returning guest. |

**Main flow**
1. The receptionist searches for and selects a possible existing guest record.
2. The system presents relevant comparison details.
3. The receptionist completes **UC-GS-06 — Verify Guest Match**.
4. When the match is confirmed, the receptionist approves the link.
5. The system checks that the association is permitted, links the reservation with the verified guest's internal ID, and confirms the result.

**Alternative flows**
- Incorrect or inconclusive match: no link is created.
- No suitable existing record: the booking workflow may create a new Guest record instead.
- Unauthorized or invalid reassignment: the system rejects the operation.
- Persistence failure: the system reports failure and does not claim the link was saved.

**Postconditions:** On success, the reservation references the verified existing Guest record. No Guest records are automatically merged or deleted.

**Relationship:** UC-GS-05 **includes** UC-GS-06 because match verification is mandatory.

## UC-GS-06 — Verify Guest Match

| Field | Description |
|---|---|
| Primary actor | Receptionist |
| Requirement | FR-GS-03 |
| Goal | Confirm that an existing guest record belongs to the person associated with the current reservation. |
| Preconditions | The receptionist is authenticated, authorized, and has selected a possible match. |
| Trigger | UC-GS-05 needs verification of a candidate Guest record. |

**Main flow**
1. The system displays permitted existing guest details for comparison.
2. The receptionist compares them against the current guest's supplied information.
3. The receptionist decides whether the available evidence is sufficient.
4. If verified, the receptionist records confirmation for this linking operation.
5. The system accepts that confirmation and allows UC-GS-05 to continue.

**Alternative flows**
- Information conflicts: the receptionist rejects the match.
- Information is insufficient: the match remains unverified.
- Unauthorized action: the system denies the operation.

**Postconditions:** The link can proceed only if the match was confirmed; otherwise, records remain unlinked.

## Requirements Traceability

| Requirement | Design evidence |
|---|---|
| FR-GS-01 | UC-GS-05; Guest–Reservation relationship. Creation of new Guest records is further modeled in Increment 5. |
| FR-GS-02 | UC-GS-02, UC-GS-03; GuestManagementService. |
| FR-GS-03 | UC-GS-01, UC-GS-05, UC-GS-06; explicit verification before linking. |
| FR-GS-04 | UC-GS-04; Guest–Reservation–Stay relationships. |

## Open Design Decisions and Review Notes

- The exact evidence and steps a receptionist must use to verify a returning guest are not finalized. Same name, email, or phone alone is not sufficient.
- The permitted guest profile fields and validation rules should be agreed before implementation.
- The guest-related Class Diagram and ERD are partial views. The shared hotel design will add RoomType, Room, booking access-code hashes, and other fields/entities.
- The model assumes at most one Stay per Reservation. If retained, `STAY.reservation_id` must be required and unique in the database schema.
- This design does not imply a guest website account or permission to view complete guest history using a booking code.
- `v0.4.0-design` should be tagged only after the necessary design artifacts and rules have been reviewed and merged through a pull request.
