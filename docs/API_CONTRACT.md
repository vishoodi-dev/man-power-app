# API Contract Document
## Base
- Base URL: /api

- Auth: Authorization: Bearer <token> (MVP can mock token)

- Content-Type: application/json
## Standard Response Rules
- Dates are ISO strings: "2026-03-02T10:30:00.000Z"

- IDs are strings (UUID-like) to avoid future migration pain

## Standard Error Format
All errors must follow
```txt
{
  "error": {
    "code": "SOME_ERROR_CODE",
    "message": "Human readable message",
    "details": { "field": "optional extra info" }
  }
}
```
Common HTTP codes:

- 400 validation error

- 401 unauthenticated

- 403 unauthorized (role not allowed)

- 404 not found

- 409 conflict (invalid transition / duplicate)

- 429 rate limit (future)

- 500 unexpected
## 1) Auth
### 1.1 Register
**POST** /auth/register
### Request
```txt
{
  "role": "CUSTOMER",
  "name": "Kasun Perera",
  "email": "kasun@gmail.com",
  "phone": "+94771234567",
  "password": "Password123!"
}
```
**Response** 201
```txt
{
  "user": {
    "id": "usr_001",
    "role": "CUSTOMER",
    "name": "Kasun Perera",
    "email": "kasun@gmail.com",
    "phone": "+94771234567",
    "city": "Colombo",
    "createdAt": "2026-03-02T10:30:00.000Z"
  },
  "token": "mock.jwt.token"
}
```
### Errors
- 400 VALIDATION_ERROR

- 409 EMAIL_ALREADY_EXISTS

### 1.2 Login
**POST** /auth/login
### Request
```txt
{
  "email": "kasun@gmail.com",
  "password": "Password123!"
}
```
**Response** 200
```txt
{
  "user": {
    "id": "usr_001",
    "role": "CUSTOMER",
    "name": "Kasun Perera",
    "email": "kasun@gmail.com",
    "phone": "+94771234567"
  },
  "token": "mock.jwt.token"
}
```
### Errors
- 400 VALIDATION_ERROR

- 401 INVALID_CREDENTIALS

### 1.3 Profile
**GET** /auth/profile
**Response** 200
```txt
{
  "user": {
    "id": "usr_001",
    "role": "CUSTOMER",
    "name": "Kasun Perera",
    "email": "kasun@gmail.com",
    "phone": "+94771234567"
  }
}
```
### Errors
- 401 UNAUTHENTICATED

## 2) Services
### 2.1 List Service Catergories
**GET** /services
**Response** 200
```txt
{
  "items": [
    { "id": "svc_001", "name": "Carpenter", "icon": "hammer" },
    { "id": "svc_002", "name": "Plumber", "icon": "pipe" },
    { "id": "svc_003", "name": "Electrician", "icon": "bolt" }
  ]
}
```

## 3) Workers
### 3.1 List Workers (with filters + pagination)
**GET** /workers
### Query Params (all optional)
- serviceId string

- city string

- minRating number (0–5)

- verified boolean (true|false)

- sort enum: rating_desc | price_asc | distance_asc | jobs_desc

- page number (default 1)

- pageSize number (default 10, max 50)

**Response** 200
```txt
{
  "items": [
    {
      "id": "wrk_001",
      "name": "Nimal Silva",
      "city": "Colombo",
      "services": ["svc_001", "svc_003"],
      "rating": 4.7,
      "jobsDone": 128,
      "verified": true,
      "startingPrice": 2500,
      "profileImageUrl": "https://example.com/wrk_001.jpg"
    }
  ],
  "pageInfo": {
    "page": 1,
    "pageSize": 10,
    "totalItems": 42,
    "totalPages": 5
  }
}
```
### Errors
- 400 VALIDATION_ERROR

### 3.2 Get Worker Profile
**GET** /workers/:workerId
**Response** 200
```txt
{
  "worker": {
    "id": "wrk_001",
    "name": "Nimal Silva",
    "bio": "10 years experience in carpentry and electrical repairs.",
    "phone": "+94770000000",
    "city": "Colombo",
    "services": [
      { "id": "svc_001", "name": "Carpenter" },
      { "id": "svc_003", "name": "Electrician" }
    ],
    "rating": 4.7,
    "jobsDone": 128,
    "verified": true,
    "startingPrice": 2500,
    "availabilityNote": "Weekdays after 5 PM",
    "profileImageUrl": "https://example.com/wrk_001.jpg",
    "createdAt": "2025-11-10T08:00:00.000Z"
  }
}
```
### Errors
- 404 WORKER_NOT_FOUND

### 3.3 Update Worker Profile (Worker only)
**PATCH** /workers/profile
### Auth
Role: WORKER
### Request
```txt
{
  "bio": "Updated bio",
  "city": "Colombo",
  "serviceIds": ["svc_001"],
  "startingPrice": 3000,
  "availabilityNote": "Weekends only"
}
```
**Response** 200
```txt
{
  "worker": {
    "id": "wrk_001",
    "bio": "Updated bio",
    "city": "Colombo",
    "services": ["svc_001"],
    "startingPrice": 3000,
    "availabilityNote": "Weekends only"
  }
}
```
### Errors
- 401 UNAUTHENTICATED

- 403 FORBIDDEN

- 400 VALIDATION_ERROR

## 4) Bookings
### Booking Status Enum
PENDING | ACCEPTED | ON_THE_WAY | STARTED | COMPLETED | REJECTED | CANCELLED
### 4.1 Create Booking (Customer)
**POST** /bookings
### Auth
Role: CUSTOMER
### Request
```txt
{
  "serviceId": "svc_001",
  "workerId": "wrk_001",
  "scheduledAt": "2026-03-05T09:30:00.000Z",
  "address": "No 12, Main Street, Colombo",
  "notes": "Need to fix cupboard door"
}
```
**Response** 201
```txt
{
  "booking": {
    "id": "bok_001",
    "serviceId": "svc_001",
    "workerId": "wrk_001",
    "customerId": "usr_001",
    "scheduledAt": "2026-03-05T09:30:00.000Z",
    "address": "No 12, Main Street, Colombo",
    "notes": "Need to fix cupboard door",
    "status": "PENDING",
    "createdAt": "2026-03-02T10:45:00.000Z",
    "updatedAt": "2026-03-02T10:45:00.000Z"
  }
}
```
### Errors
- 400 VALIDATION_ERROR

- 404 WORKER_NOT_FOUND

- 409 BOOKING_DUPLICATE_OR_CONFLICT (optional: same slot)

### 4.2 List Bookings (Customer or Worker)
**GET** /bookings
### Query Params
- role enum: customer | worker
- status optional booking status

- page, pageSize optional
  In real backend, you usually don’t pass role and userId (server uses token).
But for MSW MVP, it’s okay to keep it simple.

**Response** 200
```txt
{
  "items": [
    {
      "id": "bok_001",
      "serviceId": "svc_001",
      "serviceName": "Carpenter",
      "workerId": "wrk_001",
      "workerName": "Nimal Silva",
      "customerId": "usr_001",
      "customerName": "Kasun Perera",
      "scheduledAt": "2026-03-05T09:30:00.000Z",
      "status": "PENDING",
      "createdAt": "2026-03-02T10:45:00.000Z"
    }
  ],
  "pageInfo": { "page": 1, "pageSize": 10, "totalItems": 1, "totalPages": 1 }
}
```
### Errors
- 401 UNAUTHENTICATED
- 400 VALIDATION_ERROR

### 4.3 Get Booking Detail
**GET** /bookings/:bookingId
**Response** 200
```txt
{
  "booking": {
    "id": "bok_001",
    "serviceId": "svc_001",
    "serviceName": "Carpenter",
    "worker": { "id": "wrk_001", "name": "Nimal Silva", "phone": "+94770000000" },
    "customer": { "id": "usr_001", "name": "Kasun Perera", "phone": "+94771234567" },
    "scheduledAt": "2026-03-05T09:30:00.000Z",
    "address": "No 12, Main Street, Colombo",
    "notes": "Need to fix cupboard door",
    "status": "PENDING",
    "createdAt": "2026-03-02T10:45:00.000Z",
    "updatedAt": "2026-03-02T10:45:00.000Z"
  }
}
```
### Errors
- 404 BOOKING_NOT_FOUND

- 403 FORBIDDEN (not owner/assigned worker)

### 4.4 Update Booking Status (Worker or Customer)
**PATCH** /bookings/:bookingId/status
### Request
```txt
{
  "status": "ACCEPTED"
}
```
**Response** 200
```txt
{
  "booking": {
    "id": "bok_001",
    "status": "ACCEPTED",
    "updatedAt": "2026-03-02T11:00:00.000Z"
  }
}
```
### Rules (enforced by backend/MSW)
- Worker can: PENDING → ACCEPTED/REJECTED, then progress to ON_THE_WAY/STARTED/COMPLETED

- Customer can: cancel only in PENDING or ACCEPTED
### Errors
- 409 BOOKING_INVALID_TRANSITION

- 403 FORBIDDEN

- 404 BOOKING_NOT_FOUND

### 4.5 Cancel Booking (optional convenience endpoint)
**POST** /bookings/:bookingId/cancel
### Request
```txt
{ "reason": "Change of plans" }
```
**Response** 200
```txt
{
  "booking": { "id": "bok_001", "status": "CANCELLED" }
}
```
### Errors
- 409 BOOKING_CANNOT_CANCEL

- 403 FORBIDDEN

- 404 BOOKING_NOT_FOUND
