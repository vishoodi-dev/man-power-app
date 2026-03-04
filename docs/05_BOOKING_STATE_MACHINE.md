# Booking State Machine
## Purpose
The booking lifecycle is modeled as a finite state machine to ensure:
- predictable behavior
- no invalid transitions
- clear responsibilities per role
- consistent UI + API enforcement

This document is the source of truth for booking state behavior.

## 1) Status Enum
### Core statuses (MVP)
- PENDING
- ACCEPTED
- STARTED
- COMPLETED
- REJECTED
- CANCELLED

### Optional (phase 2)
- ON_THE_WAY
- DISPUTED
MVP should start with core statuses only.

## 2) Actors (Who triggers events)
- CUSTOMER
- WORKER
- ADMIN (limited actions, e.g., force cancel)

## 3) Events (Actions that change state)
### Customer events
- CREATE_BOOKING
- CANCEL_BOOKING
- SUBMIT_REVIEW (doesn’t change booking state, but depends on it)

### Worker events
- ACCEPT_BOOKING
- REJECT_BOOKING
- MARK_STARTED
- MARK_COMPLETED

### Admin events (optional MVP-lite)
- FORCE_CANCEL
- FORCE_COMPLETE (avoid in MVP unless required)

## 4) Transition Table (State → Event → New State)
### From PENDING
- ACCEPT_BOOKING (WORKER) → ACCEPTED
- REJECT_BOOKING (WORKER) → REJECTED
- CANCEL_BOOKING (CUSTOMER) → CANCELLED

### From ACCEPTED
- MARK_STARTED (WORKER) → STARTED
- CANCEL_BOOKING (CUSTOMER) → CANCELLED

### From STARTED
- MARK_COMPLETED (WORKER) → COMPLETED

### Terminal states (no transitions)
- COMPLETED
- REJECTED
- CANCELLED

## 5) State Diagram (Text)
```txt
PENDING
  ├── (worker accepts) ──> ACCEPTED ── (worker starts) ──> STARTED ── (worker completes) ──> COMPLETED
  ├── (worker rejects) ──> REJECTED
  └── (customer cancels) ──> CANCELLED

ACCEPTED
  └── (customer cancels) ──> CANCELLED
```

## 6) Role Permissions Matrix

| Action/Event    | CUSTOMER                     | WORKER               | ADMIN          |
|-----------------|------------------------------|----------------------|---------------|
| Create booking  | ✅                           | ❌                   | ✅ (optional) |
| Cancel booking  | ✅ (PENDING / ACCEPTED only) | ❌                   | ✅ (force)    |
| Accept booking  | ❌                           | ✅ (PENDING only)    | ❌            |
| Reject booking  | ❌                           | ✅ (PENDING only)    | ❌            |
| Mark started    | ❌                           | ✅ (ACCEPTED only)   | ❌            |
| Mark completed  | ❌                           | ✅ (STARTED only)    | ❌            |
| Submit review   | ✅ (COMPLETED only)          | ❌                   | ❌            |


## 7) Invariants (Rules that must always be true)
### Ownership invariants
- Only the booking’s customerId can cancel (as CUSTOMER)
- Only the assigned workerId can accept/start/complete

### State invariants
- COMPLETED/REJECTED/CANCELLED are terminal
- Review can only be created if booking is COMPLETED
- Booking cannot be moved backwards (e.g., STARTED → ACCEPTED)

### Data invariants
- Booking must always have: id, customerId, workerId, serviceId, status, scheduledAt
- updatedAt must change on every transition

## 8) API Enforcement Rules
The backend (or MSW mock) must enforce:
- valid state transitions
- valid actor permissions

### Recommended error for invalid transitions
- HTTP 409 Conflict
- Code: BOOKING_INVALID_TRANSITION

Example response:
```txt
{
  "error": {
    "code": "BOOKING_INVALID_TRANSITION",
    "message": "Cannot move from STARTED to ACCEPTED",
    "details": { "from": "STARTED", "to": "ACCEPTED" }
  }
}
```
### Recommended error for forbidden actor
- HTTP 403 Forbidden
- Code: FORBIDDEN

## 9) UI Rules (Front-end behavior)
**Button visibility by status**
### Customer View
- PENDING: show Cancel
- ACCEPTED: show Cancel
- STARTED: show No actions
- COMPLETED: show Leave Review
- REJECTED/CANCELLED: show No actions

### Worker View
- PENDING: show Accept, Reject
- ACCEPTED: show Start Job
- STARTED: show Complete Job
- terminal: show No actions

### UX Requirements
- Disable action buttons while mutation is in-flight
- Prevent double-click / duplicate submission
- After status update, refetch or invalidate booking queries

## 10) Concurrency & Edge Cases
### Edge Case A: Two workers try to accept the same booking

In our model, each booking is created with a single workerId.
So this cannot happen.

If later you add “broadcast booking to multiple workers”, you must implement:
- first accept wins
- others receive 409 conflict

### Edge Case B: Customer cancels while worker accepts

Backend must ensure **only one final outcome.**

Rules:
- If cancel request arrives first → status becomes CANCELLED
- If accept request arrives first → status becomes ACCEPTED
- Second request gets 409 BOOKING_INVALID_TRANSITION

### Edge Case C: Worker completes without starting
Must be blocked (ACCEPTED → COMPLETED is invalid)

### Edge Case D: Late cancellation policy
If later needed:
- enforce cancellation only if scheduledAt - now > X minutes

## 11) Audit & History (Optional but Professional)
To support debugging and admin visibility, maintain a history log:

booking_events:
- id
- bookingId
- actorRole
- actorId
- event
- fromStatus
- toStatus
- createdAt
This is not required for MVP UI but is excellent for backend later.

## 12) Implementation Notes
### Frontend
- Keep transitions in a single file:
features/bookings/bookingStateMachine.ts
- UI calls getAvailableActions(booking, role)
- UI never hardcodes transition rules in multiple places

### Backend (later)
- Validate transitions server-side
- Use optimistic locking or transactions to prevent race issues
- Return 409 for invalid transitions

## 13) Acceptance Criteria
- UI never shows invalid actions for a given role/status
- Backend/MSW blocks invalid transitions even if UI is bypassed
- Booking list and details always show consistent status
- Review submission is only possible after COMPLETED
