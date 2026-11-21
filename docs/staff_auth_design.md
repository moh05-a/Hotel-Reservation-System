# Increment 1 — Staff Authentication and Access Design

## 1. Overview

This document describes staff authentication and role-based access control for the Hotel Reservation System.

Only hotel staff have login accounts. Guests can search and book rooms without registration, and later access their confirmed reservations using a booking reference and secret code.

This design covers `FR-AU-01`, `FR-AU-02`, `FR-AU-03`, `NFR-03`, and `NFR-06`.

## 2. Staff Roles and Permissions

The system has three staff roles:

- **Hotel Manager:** Manages room types, physical rooms, nightly rates, and views hotel reports.
- **Receptionist:** Manages guest records, reservations, check-in/check-out, and billing.
- **Housekeeping Staff:** Views room information and updates permitted cleaning and operational statuses.

### Role-Based Access Control (RBAC) Matrix

| Function | Hotel Manager | Receptionist | Housekeeping |
|---|---|---|---|
| Staff Login/Logout | Yes | Yes | Yes |
| View Room Information | Yes | Yes | Yes |
| Manage Room Types and Rates | Yes | No | No |
| Manage Physical Rooms | Yes | No | No |
| Manage Guest Records | No | Yes | No |
| Manage Reservations | No | Yes | No |
| Check-in / Check-out | No | Yes | No |
| Manage Billing | No | Yes | No |
| Update Housekeeping Status | No | No | Yes |
| View Operations Dashboard | Yes | No | No |

Permissions must be checked by the backend, not only by hiding buttons in the frontend.

## 3. Authentication Process

Staff members log in using their username and password.

The authentication process follows these steps:

1. The staff member enters their credentials on the login page.
2. The React frontend sends the credentials to the FastAPI backend.
3. The backend looks up the staff account in the database.
4. The submitted password is checked against the stored password hash.
5. If the credentials are valid and the account is active, the system creates an authenticated session.
6. The frontend opens the appropriate staff interface based on the user's role.
7. If authentication fails, the system rejects the request and displays a general error message.

Guests do not use this authentication process.

## 4. Session Management

The proposed design uses server-side sessions.

After successful login, the backend creates a random session token and sends it through an HttpOnly cookie.

The database stores a secure hash of the session token, along with the staff member's ID and session expiration time.

For protected requests, the backend checks whether the session is valid and whether the staff member has permission to perform the requested operation.

When a staff member logs out, the backend invalidates the session. The browser cookie is also cleared.

Session cookies will use suitable security settings, including Secure in HTTPS environments and SameSite protection. State-changing requests will also have CSRF protection.

This approach allows logout to invalidate access immediately.

## 5. Database Design

The first increment requires two main database entities.

### StaffUser

| Attribute | Description |
|---|---|
| staff_id | Unique staff identifier |
| username | Unique login username |
| password_hash | Securely hashed password |
| role | Manager, Receptionist, or Housekeeping |
| is_active | Whether the staff account can log in |
| created_at | Account creation timestamp |

### StaffSession

| Attribute | Description |
|---|---|
| session_id | Unique session identifier |
| staff_id | Reference to StaffUser |
| token_hash | Hash of the random session token |
| created_at | Session creation timestamp |
| expires_at | Session expiration timestamp |
| revoked_at | Logout/invalidation timestamp, if revoked |

One staff member may have multiple sessions, depending on the final session policy.

Staff accounts for the demonstration will be created through a controlled seed/setup process rather than a public registration page.

## 6. Proposed API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/auth/login` | Authenticate staff and create a session |
| GET | `/api/auth/me` | Return the current authenticated staff identity and role |
| POST | `/api/auth/logout` | Invalidate the current session |
| GET | `/api/health` | Verify that the backend is running |

Future protected endpoints will reuse the same authentication and role-checking mechanism.

## 7. Security Rules

- Staff passwords must be hashed using a suitable password-hashing algorithm, such as Argon2id.
- Plaintext passwords and session tokens must not be stored in the database or application logs.
- Invalid login attempts must return a general error without revealing whether a username exists.
- Repeated failed login attempts must be rate-limited.
- Inactive staff accounts must be denied access.
- Every protected backend operation must verify the authenticated session and required role.
- Unauthorized requests must return an appropriate HTTP error.
- Guest booking credentials must never authorize staff operations.

## 8. Acceptance Criteria

Increment 1 will be considered functionally complete when:

1. The frontend, backend, and database start successfully.
2. Staff can log in using valid credentials.
3. Incorrect login credentials are rejected.
4. Each staff role can access only its permitted operations.
5. Direct unauthorized API requests are rejected.
6. Logging out invalidates the active session.
7. Passwords are stored as secure hashes.
8. Automated authentication and authorization tests pass.
9. Another team member can run the application using the documented setup steps.

## 9. Scope and Dependencies

This increment establishes the authentication and application foundation used by later increments.

Room management, availability, guest records, reservations, and billing will be implemented separately.

The selected session design and proposed permission matrix will be reviewed with the team before implementation.