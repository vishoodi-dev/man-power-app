# Product Requirement Document (PRD)
### Product Name
**ServiceHub – Manpower Service Marketplace**
### Version
v1.0 (MVP)
### Author
Vishoodi Cooray
### Date
03/2026
## 1. Product Overview
ServiceHub is a digital marketplace platform that connects customers with skilled service professionals such as carpenters, plumbers, electricians, and other minor service providers.

The platform simplifies the process of finding reliable workers, booking services, tracking job progress, and leaving reviews.

ServiceHub aims to provide a **trusted and efficient ecosystem** for both customers and workers.
## 2. Problem Statement
Currently, finding skilled workers for small services relies heavily on informal methods such as personal contacts or local referrals.

Common issues include:
- Difficulty finding reliable workers
- Lack of transparent pricing
- No verification of worker skills
- No booking tracking system
- No accountability for poor service
  
Workers also face problems such as:
- Limited job opportunities
- No digital visibility
- Lack of reputation tracking

## 3. Product Goals
### Primary Goals
- Provide a trusted platform connecting customers with skilled workers
- Simplify service booking and management
- Create a reputation system for workers

### Secondary Goals
- Enable workers to receive more job opportunities
- Provide analytics and insights for platform administrators
- Support scalable multi-tenant architecture

## 4. Target Users
### 4.1 Customers
People who need services such as:
- Home repairs
- Furniture fixes
- Electrical work
- Plumbing work

Customer needs:
- Quick worker discovery
- Transparent reviews and ratings
- Simple booking process
- Job status tracking

### 4.2 Workers
Service professionals including:
- Carpenters
- Plumbers
- Electricians
- Painters
- Technicians

Worker needs:
- Visibility to customers
- Job notifications
- Ability to accept or reject bookings
- Reputation through reviews

### 4.3 Admin
Platform administrators responsible for:
- Managing workers
- Handling disputes
- Monitoring platform performance

## 5. MVP Scope
The first version will focus on core marketplace functionality.
### Customer Features
- User registration and login
- Browse service categories
- View worker profiles
- Create service booking
- Track booking status
- Cancel booking (limited conditions)
- Leave reviews after service completion

### Worker Features
- Worker account registration
- Create worker profile
- View incoming booking requests
- Accept or reject bookings
- Update booking status

### Admin Features
- View workers
- Verify worker profiles
- View bookings
- Basic platform analytics

## 6. Out of Scope (MVP)
The following features will not be included in the initial version:
- Payment gateway integration
- Insurance management
- AI recommendation engine
- Real-time GPS tracking
- Mobile application
(These features may be introduced in later phases.)

## 7. Core User Flows
### Flow 1: Customer Booking Service
- Customer logs into the platform
- Customer selects a service category
- Customer views available workers
- Customer selects a worker
- Customer schedules a service
- Booking request is created
- Booking status becomes PENDING.
- Worker receives notification.

### Flow 2: Worker Accepts Job
- Worker views pending booking requests
- Worker reviews job details
- Worker accepts booking
- Booking status changes to ACCEPTED.
- Customer receives notification.

### Flow 3: Job Completion
- Worker marks job as started
- Worker marks job as completed
- Customer receives completion notification
- Customer leaves review

## 8. Booking Lifecycle (State Machine)
Booking statuses:
```txt
PENDING
ACCEPTED
STARTED
COMPLETED
REJECTED
CANCELLED
```
### Valid Transitions
```txt
PENDING → ACCEPTED
PENDING → REJECTED
PENDING → CANCELLED

ACCEPTED → STARTED
ACCEPTED → CANCELLED

STARTED → COMPLETED
```
Invalid transitions must be blocked by the system.

## 9. Functional Requirements
### Authentication
Users must be able to:
- Register
- Login
- Logout
- Access role-based features

### Worker Profiles
Each worker profile includes:
- Name
- Service categories
- Location
- Years of experience
- Rating
- Reviews
- Verification badge
- Profile picture(Optional)

### Booking System
Customers must be able to:
- Create bookings
- View booking history
- Track booking status

Workers must be able to:
- Accept bookings
- Reject bookings
- Update job progress

### Review System
After job completion:
- Customers can leave rating (1–5)
- Customers can write review comments
- Reviews appear on worker profile

## 10. Non-Functional Requirements
### Performance
- Worker list should load under **2 seconds**
- Booking operations should respond under **1 second**

### Security
- Role-based access control
- Secure authentication
- Data isolation for tenants

### Scalability
Architecture should support:
- Multiple cities
- Large worker databases
- Future microservices expansion

## 11. Key Metrics (KPIs)
The following metrics will measure product success:
- Booking completion rate
- Worker response rate
- Average worker rating
- Customer retention rate
- Booking cancellation rate

## 12. Risks
Potential risks include:
- Fake worker profiles
- Worker no-shows
- Customer-worker communication outside platform
- Low early adoption

## 13. Future Roadmap
Future platform improvements may include:
- AI worker recommendation engine
- Payment gateway integration
- Real-time job tracking
- AI-based review analysis
- Worker subscription plans
- Mobile application

## 14. Technical Stack (Planned)
### Frontend
- React
- Vite
- TypeScript
- React Query
- Recharts

### Backend (future)
- NestJS
- PostgreSQL
- Redis
- JWT Authentication

### AI Features
- LLM-based operational assistant
- Review summarization

## Conclusion

ServiceHub aims to build a reliable digital marketplace for minor services while maintaining scalable architecture and clean system design.

The MVP focuses on validating the marketplace model while establishing strong technical foundations for future expansion.


