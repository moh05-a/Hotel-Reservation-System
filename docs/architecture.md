
# Hotel Reservation System Architecture

## 1. Architecture Overview

The Hotel Reservation System is a web-based application
designed to manage the main operations of a single hotel.

The system supports guests and hotel staff in managing
room availability, guest records, reservations,
check-in/check-out, billing, simulated payments,
and room operational status.

The architecture follows a client-server approach,
separating the frontend, backend, and database.

## 2. Proposed Technology Stack

| Layer | Technology | Responsibility |
|---|---|---|
| Frontend | React + Vite | User interface |
| Backend | Python + FastAPI | Business logic and REST API |
| ORM | SQLAlchemy | Database access |
| Database | SQLite / PostgreSQL | Persistent storage |
| Payment | Simulated Payment Gateway | Simulated transactions |
| Diagrams | PlantUML | Architecture modeling |

Note: The technology stack is provisional until
confirmed by the project team.

## 3. C4 Level 1 - System Context

Source: `uml/c4-level-1-context.puml`

The Level 1 diagram shows the Hotel Reservation System
and its interactions with external users and systems.

### People

- Guest: Searches rooms and manages reservations.
- Receptionist: Handles guest records, reservations,
  check-in, check-out, and billing.
- Hotel Manager: Oversees hotel operations and room rates.
- Housekeeping Staff: Updates room cleaning status.

### External System

- Simulated Payment Gateway: Processes simulated payment
  and refund requests and returns transaction results.

The Hotel Reservation System is the central software
system connecting these actors and the payment service.

## 4. C4 Level 2 - Container Diagram

Source: `uml/c4-level-2-container.puml`

The Level 2 diagram describes the internal technical
structure of the Hotel Reservation System.

### Web Frontend

Technology: React + Vite.

Provides the interfaces used by guests and hotel staff.
It sends HTTP requests to the backend API.

### Backend API

Technology: Python + FastAPI.

Responsible for:
- Authentication and authorization.
- Room and rate management.
- Room availability validation.
- Guest and reservation management.
- Check-in and check-out.
- Billing and tax calculation.
- Payment and refund integration.
- Room operational status.
- Notification logging.

### Application Database

Technology: SQLite / PostgreSQL with SQLAlchemy.

Stores:
- User and guest records.
- Rooms and room types.
- Room rates.
- Reservations.
- Payment and billing records.
- Room operational statuses.
- Notification logs.

### Simulated Payment Gateway

An external system that simulates payment authorization
and refunds. No real card information is processed.

## 5. Communication Between Containers

1. Users interact with the React frontend through HTTPS.
2. The frontend communicates with the FastAPI backend
   through REST API requests using JSON.
3. The backend accesses persistent data through
   SQLAlchemy.
4. The backend sends simulated payment requests to
   the external payment gateway.
5. The payment gateway returns transaction results
   to the backend.

## 6. Architecture Assumptions

- The system manages one hotel.
- Payments are simulated.
- No real card information is stored.
- Notifications are logged in the application database.
- Room prices are based on nightly room-type rates.
- Tax is configurable.
- The frontend does not directly access the database.

## 7. Architecture Decisions

The frontend and backend are separated to allow
independent development and clear responsibilities.

FastAPI provides the backend REST API.
SQLAlchemy handles database operations.

The database is responsible for persistent storage.

The payment gateway is modeled as an external system
because payment processing is simulated independently
of the hotel's internal application.
