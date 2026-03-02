# 🚀 ServiceHub
Production-grade, multi-tenant service marketplace platform with booking state machine, AI-powered copilot (streaming), and real-time analytics dashboard.
## 📌 Overview
ServiceHub is a scalable marketplace platform connecting customers with verified service professionals (carpenters, plumbers, electricians, etc.).

This project is designed as an industrial-grade system, focusing on:

- Clean architecture

- Role-based access control

- Booking lifecycle state modeling

- Multi-tenant isolation

- AI-driven operational insights

- Real-time data visualization
## ✨ Core Features
### 🏠 Marketplace
- Multi-role authentication (Customer, Worker, Admin)

- Worker profile & service categories

- Booking creation and tracking

- Review & rating system

- Status-based UI permissions

### 🔁 Booking State Machine
Booking lifecycle is strictly modeled to prevent invalid transitions:
```txt
PENDING → ACCEPTED → STARTED → COMPLETED
PENDING → REJECTED
PENDING/ACCEPTED → CANCELLED
```
Rules:

- Workers can accept/reject only in PENDING

- Customers can cancel only in PENDING or ACCEPTED

- Reviews allowed only after COMPLETED

- Invalid transitions are blocked at API level

This ensures predictable and enforceable business logic.

### 📊 Analytics Dashboard (Recharts)
Admin dashboard includes:

- Bookings per day (Line Chart)

- Acceptance rate by service (Bar Chart)

- Cancellation breakdown (Pie Chart)

- Worker leaderboard

- Completion performance trends

Supports real-time updates (polling / streaming-ready).

### 🤖 AI Copilot (LLM Streaming)
AI-powered assistant integrated into the admin panel:

- Streaming chat responses (SSE-based)

- Operational insights (e.g., booking trends)

- Review summarization

- Tool-based UI interaction (e.g., “Show bookings chart”)

Designed with explainable outputs and structured responses.

### 🏢 Multi-Tenancy
Supports tenant isolation using tenantId strategy:

- Each tenant has isolated workers, bookings, and analytics

- Role-based access scoped to tenant

- Prevents cross-tenant data access

- Designed for future subdomain-based tenancy

## 🏗 Architecture
### Frontend
- Vite + React + TypeScript

- Feature-based modular architecture

- TanStack Query for server-state management

- React Router for routing

- Recharts for data visualization

- MSW for mock backend simulation

### Backend (Planned / In Progress)
- NestJS

- PostgreSQL

- JWT Authentication

- Tenant-based filtering

- SSE for AI streaming

## 📂 Project Structure
```txt
src/
  app/
  api/
  features/
    auth/
    services/
    workers/
    bookings/
    analytics/
    copilot/
  shared/
  mocks/
```
Features are isolated and domain-driven.

## 🔐 Role-Based Access Control
Roles:

- CUSTOMER

- WORKER

- ADMIN

Access is controlled via:

- Route guards

- Conditional UI rendering

- API-level enforcement

## 📦 Installation

```txt
git clone https://github.com/your-username/servicehub.git
cd servicehub
npm install
npm run dev
```
## 🧪 Testing
- Unit tests (Vitest)

- Component tests (React Testing Library)

- Mock API integration via MSW

Run tests:
```txt
npm run test
```

## 🚀 Deployment

## 📈 Future Improvements

- Stripe payment integration

- Real-time booking updates via WebSocket

- AI-based worker recommendation engine

- Push notifications

- Production-ready NestJS backend

- Load testing and performance benchmarking

## 📷 Screenshots
## 🌐 Live Demo
## 👤 Author
Vishoodi Cooray

