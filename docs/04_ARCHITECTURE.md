# Architecture Overview
## Project
### ServiceHub – ManPower Service Marketplace

## 1. Introduction
This document describes the technical architecture of ServiceHub, a multi-role service marketplace platform connecting customers with skilled workers.

The architecture is designed to support:
- scalable frontend architecture
- modular domain separation
- multi-tenant data isolation
- replaceable backend infrastructure
- future AI integrations
- analytics and data visualization

The current implementation focuses on a frontend-first architecture using mock APIs, allowing rapid development before integrating the real backend.

## 2. High-Level Architecture
```txt
Client (React + Vite)
        │
        ▼
API Client Layer
        │
        ▼
Mock API (MSW)
        │
        ▼
Mock Database
```
Future architecture will introduce a backend layer:
```txt
Client (React + Vite)
        │
        ▼
API Client Layer
        │
        ▼
NestJS Backend
        │
        ▼
PostgreSQL Database
        │
        ├── Redis (Caching)
        ├── AI Service (LLM Copilot)
        └── Analytics Engine
```
## 3. Frontend Architecture
The frontend follows a **feature-based modular architecture.**
- Design Principles
- Domain-driven feature modules
- Separation between UI and data logic
- Replaceable API layer
- Minimal global state
- Strong typing with TypeScript

## 4. Frontend Layer Structure
The frontend is divided into several layers:

### UI Layer
Contains presentational components and pages.

Responsibilities:
- render UI
- display data
- handle user interactions

Examples:
- worker list page
- booking dashboard
- analytics dashboard

### Feature Layer
Contains domain-specific logic.

Each feature module manages:
- UI components
- data fetching hooks
- local business logic
- types related to the feature

Example feature modules:
- auth
- services
- workers
- bookings
- analytics
- copilot (AI assistant)

### Data Layer
The data layer manages communication with APIs.

Responsibilities:
- API requests
- error normalization
- request configuration
- response typing

Technologies used:
- Fetch API
- TanStack Query

### Mock API Layer
Mock APIs simulate backend services during frontend development.

Implemented using:

### MSW (Mock Service Worker)
Advantages:
- realistic API behavior
- simulated network delays
- simulated server errors
- testable UI flows

## 5. Project Folder Structure
```txt
src/
  app/
    routes.tsx
    providers/
    layout/

  api/
    http.ts
    error.ts
    endpoints/

  features/
    auth/
    services/
    workers/
    bookings/
    analytics/
    copilot/

  shared/
    ui/
    hooks/
    types/
    lib/

  mocks/
    handlers/
    db.ts
```
## 6. Domain Modules
### Authentication Module
Responsibilities:
- user login
- user registration
- role management
- session management

User roles:
- CUSTOMER
- WORKER
- ADMIN

### Services Module
Responsibilities:
- display service categories
- allow service filtering
- support service browsing

Example services:
- carpentry
- plumbing
- electrical repair

### Workers Module
Responsibilities:
- display worker profiles
- show ratings and reviews
- worker verification status

Worker profile includes:
- skills
- experience
- location
- rating
- jobs completed

### Booking Module
The booking module is the core business logic of the platform.

Responsibilities:
- booking creation
- booking tracking
- booking lifecycle management
Booking states follow a strict **state machine**.

### 7. Booking State Machine
Booking lifecycle states:
```txt
PENDING
ACCEPTED
STARTED
COMPLETED
REJECTED
CANCELLED
```
Allowed transitions:
```txt
PENDING → ACCEPTED
PENDING → REJECTED
PENDING → CANCELLED

ACCEPTED → STARTED
ACCEPTED → CANCELLED

STARTED → COMPLETED
```
This ensures:
- predictable business logic
- prevention of invalid actions
- clear workflow for workers and customers

## 8. Server State Management
Server data is managed using:

### TanStack Query

Advantages:
- automatic caching
- background refetching
- request deduplication
- simplified loading/error states

Example query keys:
```txt
['services']
['workers']
['worker', workerId]
['bookings']
['booking', bookingId]
```

## 9. Error Handling Strategy
All API errors are normalized into a single format:
```txt
ApiError {
  code: string
  message: string
  status: number
  details?: object
}
```
Benefits:
- consistent error handling
- simplified UI logic
- improved debugging

## 10. Multi-Tenant Architecture
ServiceHub supports a multi-tenant architecture.
Each tenant represents a separate organization or service provider network.

All core entities include:
```txt
tenantId
```

Example entities:
- workers
- bookings
- services
- reviews

Benefits:
- tenant data isolation
- scalability for SaaS deployment
- easier enterprise onboarding

## 11. Analytics Architecture
Analytics features will provide insights for administrators.

Metrics include:
- bookings per day
- worker acceptance rate
- cancellation trends
- worker performance
  
Visualization library:
- Recharts

Charts used:
- line charts
- bar charts
- pie charts

## 12. AI Copilot Architecture
The platform includes an optional AI assistant to help administrators analyze platform data.

Capabilities:
- review summarization
- operational insights
- booking trend explanations

Streaming responses will be implemented using:

### Server-Sent Events (SSE)

Example workflow:
```txt
Admin Prompt
      │
      ▼
AI Service
      │
      ▼
Streaming Response
      │
      ▼
Copilot Chat Interface
```
## 13. Future Backend Architecture
The backend will be implemented using:
- NestJS
- PostgreSQL
- Redis
- JWT authentication

Responsibilities:
- API endpoints
- business logic enforcement
- booking state transitions
- analytics aggregation
- AI service integration

## 14. Scalability Considerations
The system is designed to support:
- multi-city service expansion
- high worker counts
- increased booking volume

Potential future improvements:
- microservices architecture
- message queues
- event-driven booking updates

## 15. Deployment Architecture
Frontend deployment:
- Vercel
- Netlify

Backend deployment:
- Docker containers
- Cloud infrastructure

Environment configuration will use:
```txt
.env
.env.production
```

## 16. Summary
The architecture prioritizes:
- modular design
- scalability
- clear domain separation
- maintainable frontend structure

This foundation allows the platform to evolve into a fully scalable marketplace system while maintaining strong engineering practices.
