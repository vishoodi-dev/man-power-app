# Technical Design Document (TDD)
## Project
ServiceHub — ManPower / Minor Services Marketplace (Customer ↔ Worker ↔ Admin)

## Version
v1.0 (Frontend-first MVP)

## Author
Vishoodi Cooray

## Date
2026-03-04

## 1) Objectives
### 1.1 Goals (MVP)
- Build a production-grade frontend architecture with realistic workflows
- Implement role-based navigation and authorization patterns
- Model booking lifecycle using a state machine (strict transitions)
- Use a replaceable API layer: MSW now → real backend later
- Build an “admin-grade” experience: dashboards, states, failures, edge cases

### 1.2 Non-Goals (MVP)
- Payments
- Real-time GPS tracking
- Full dispute workflow
- Microservices
- DB-per-tenant multi-tenancy

## 2) System Context
### Actors
**Customer:** Browse workers, create bookings, track status, review
**Worker:** Accept/reject bookings, update status
**Admin:** Verify workers, view bookings, see analytics (optional)

### High-Level Flow
Customer → selects service & worker → creates booking → worker accepts/rejects → service progresses → customer reviews.

## 3) Tech Stack
### Frontend (MVP)
- Vite + React + TypeScript
- React Router for routes
- TanStack Query (React Query) for server state
- React Hook Form + Zod for forms and validation (recommended)
- MSW (Mock Service Worker) for fake API
- Recharts for analytics (phase 2)

### Backend (later)
- NestJS, PostgreSQL, Redis
- JWT auth (access + refresh)
- SSE for streaming AI responses (optional)
- Tenant isolation middleware (optional)

## 4) Architecture Overview
### Layered Frontend Architecture
- **UI Layer:** pages + components
- **Feature Layer:** domain modules (auth/services/workers/bookings/admin)
- **Data Layer:** API client + React Query hooks
- **Mock Layer:** MSW handlers + mock DB
- **Shared Layer:** common UI, utilities, types
**Key Principle:** Feature modules should be isolated; shared utilities live in /shared.

## 5) Repository Structure
```txt
servicehub/
  README.md
  docs/
    01_PRD.md
    02_TDD.md
    03_API_CONTRACT.md
    04_ARCHITECTURE.md
    05_BOOKING_STATE_MACHINE.md
  src/
    app/
      App.tsx
      routes.tsx
      layout/
        RootLayout.tsx
        ProtectedRoute.tsx
      providers/
        QueryProvider.tsx
        AuthProvider.tsx
    api/
      http.ts
      error.ts
      endpoints/
        auth.api.ts
        services.api.ts
        workers.api.ts
        bookings.api.ts
        reviews.api.ts
        analytics.api.ts (phase 2)
        ai.api.ts (phase 3)
    features/
      auth/
      services/
      workers/
      bookings/
      admin/
      analytics/ (phase 2)
      copilot/ (phase 3)
      tenancy/ (optional)
    shared/
      ui/
      lib/
      hooks/
      types/
      config/
    mocks/
      db.ts
      browser.ts
      handlers/
  .github/
    workflows/ci.yml
```

## 6) Domain Model (Types)
### Roles
- CUSTOMER | WORKER | ADMIN

### Core Entities
- **User:** { id, role, name, email, phone, city }
- **ServiceCategory:** { id, name, icon }
- **WorkerProfile:** { id, userId, serviceIds[], rating, jobsDone, verified, startingPrice, bio, city }
- **Booking:** { id, tenantId?, customerId, workerId, serviceId, scheduledAt, address, notes, status, createdAt, updatedAt }
- **Review:** { id, bookingId, workerId, customerId, stars, comment, createdAt }
IDs are strings (UUID-like) for future backend compatibility.

## 7) Routing & Access Control
### Routes
### Customer
- /services
- /workers/:workerId
- /bookings
- /bookings/:bookingId

### Worker
- /worker/bookings
- /worker/profile 

### Admin
- /admin/workers
- /admin/bookings
- /admin/analytics (phase 2)
- /admin/copilot (phase 3)

### Guards
- ProtectedRoute: checks authentication
- RoleGuard: checks role authorization
- TenantGuard (optional): ensures tenant context is chosen

## 8) Booking State Machine (Core Business Logic)
### Status Enum
PENDING | ACCEPTED | STARTED | COMPLETED | REJECTED | CANCELLED
(Optional later: ON_THE_WAY)

### Valid Transitions
- PENDING → ACCEPTED | REJECTED | CANCELLED
- ACCEPTED → STARTED | CANCELLED
- STARTED → COMPLETED
- Terminal: COMPLETED | REJECTED | CANCELLED

### Role Permissions
Customer:
- cancel only in PENDING or ACCEPTED
- review only after COMPLETED

Worker:
- accept/reject only in PENDING
- start only after ACCEPTED
- complete only after STARTED

### Implementation
features/bookings/bookingStateMachine.ts exports:
- canTransition(from, to, actorRole)
- getAvailableActions(booking, actorRole)
- isTerminal(status)

## 9) API Layer Design (Replaceable)
### API Client
src/api/http.ts:
- base URL from env (VITE_API_BASE_URL)
- request wrapper
- inject token header
- parse error format consistently

src/api/error.ts:
- maps API error to typed ApiError:
code, message, details, status

Endpoint Modules

services.api.ts

workers.api.ts

bookings.api.ts

reviews.api.ts

(future) analytics.api.ts

(future) ai.api.ts
