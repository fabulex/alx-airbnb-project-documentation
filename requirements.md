# Airbnb Clone Backend Requirements

## Overview
This document outlines the detailed backend requirements for an Airbnb Clone application. It comprehensively captures both **functional requirements** (what the system does, e.g., user stories, API behaviors) and **technical requirements** (how it's implemented, e.g., database schemas, integrations, security measures) for each core feature. Non-functional requirements (e.g., scalability, performance) are integrated per feature where applicable and summarized globally.

The backend leverages a relational database (PostgreSQL), RESTful APIs (with optional GraphQL for complex queries), JWT-based authentication, role-based access control (RBAC), and third-party integrations like Stripe for payments, AWS S3 for file storage, and SendGrid for emails. All features adhere to error handling (global middleware for HTTP status codes and structured logging) and testing (unit/integration with pytest, 80% coverage).

**Current Date**: November 02, 2025
**Version**: 1.1 (Updated to explicitly delineate functional and technical aspects per feature)
**Authors**: Grok (xAI)

## Core Functionalities

Each feature below details:
- **Functional Requirements**: User-facing behaviors and workflows.
- **Technical Requirements**: Implementation details, including database interactions, integrations, and auth.
- **API Endpoints**: RESTful interfaces.
- **Input/Output Specifications**: JSON formats.
- **Validation Rules**: Constraints and error responses.
- **Performance Criteria**: Non-functional targets (latency, scalability, optimization).

### 1. User Management
#### Functional Requirements
- Users can register, log in via email/password or OAuth (Google), and log out.
- Authenticated users manage profiles (update details, upload photos).
- Roles: Guest (default, can book), Host (create listings), Admin (full access).
- Supports password reset and session management.

#### Technical Requirements
- Database: `Users` table (id: UUID PK, email: unique, password: hashed bcrypt, role: enum, profilePhotoUrl: string).
- Auth: JWT (access: 15min, refresh: 7d) with HS256 signing; OAuth via Auth0/Google API.
- Storage: S3 for photos (resized thumbnails via Lambda).
- Integration: None external beyond OAuth.
- Security: Rate limiting on auth endpoints; email verification on register.

#### API Endpoints
| Method | Endpoint                  | Description                          |
|--------|---------------------------|--------------------------------------|
| POST   | `/api/auth/register`      | Register a new user                  |
| POST   | `/api/auth/login`         | Authenticate and issue JWT           |
| POST   | `/api/auth/oauth/{provider}` | OAuth login (e.g., /google)       |
| POST   | `/api/auth/logout`        | Blacklist JWT (client-side)          |
| GET    | `/api/auth/profile`       | Fetch profile                        |
| PUT    | `/api/auth/profile`       | Update profile                       |

#### Input/Output Specifications
- **Register**: Input `{ "email": "string", "password": "string", "firstName": "string", "lastName": "string", "phone": "string?", "role": "enum?" }`; Output `{ "userId": "uuid", "token": "string", "refreshToken": "string?" }` (201).
- **Login**: Input `{ "email": "string", "password": "string" }`; Output `{ "userId": "uuid", "token": "string", "expiresIn": "number" }` (200).
- **Profile GET**: Output `{ "userId": "uuid", "email": "string", "firstName": "string", ... }` (200).
- **Profile PUT**: Input `{ "firstName": "string?", ... , "profilePhoto": "file" }`; Output `{ "message": "string", "updatedUser": {...} }` (200).

#### Validation Rules
- Email: Unique, regex-validated (400); duplicate (409).
- Password: Min 8 chars, complexity (400).
- Role: Default guest; admin-only override (403).
- File: ≤5MB, JPEG/PNG (413).
- JWT: Middleware validation (401/403).

#### Performance Criteria
- Latency: <200ms (auth ops).
- Scalability: Stateless; Redis blacklist for logout (TTL 15min).
- Optimization: Cache profiles (Redis TTL 1h).

### 2. Property Listings Management
#### Functional Requirements
- Hosts create/edit/delete listings with details (title, desc, location, price, amenities, availability calendar).
- Guests/admins view listings.
- Availability prevents double-bookings via integration with bookings.

#### Technical Requirements
- Database: `Properties` table (id: UUID PK, hostId: FK Users, title: string, desc: text, location: JSON (address, lat/lng), pricePerNight: decimal, amenities: array JSON, availability: JSON array of date ranges); `PropertyImages` table (propertyId: FK, url: string).
- Auth: Host RBAC for CRUD (verify hostId in JWT).
- Storage: S3 for images (multi-upload via presigned URLs).
- Integration: Google Maps API for geocoding on create/update.
- Logging: Audit trails for edits (separate `PropertyAudits` table).

#### API Endpoints
| Method | Endpoint                  | Description                          |
|--------|---------------------------|--------------------------------------|
| POST   | `/api/properties`         | Create listing                       |
| GET    | `/api/properties/{id}`    | Fetch property                       |
| PUT    | `/api/properties/{id}`    | Update property                      |
| DELETE | `/api/properties/{id}`    | Delete property                      |
| GET    | `/api/properties`         | List properties (basic)              |

#### Input/Output Specifications
- **Create**: Input `{ "title": "string", "description": "string", "location": { ... }, "pricePerNight": "number", "amenities": ["string"], "availability": [{ "startDate": "string", "endDate": "string" }], "images": ["file/URL"] }`; Output `{ "propertyId": "uuid", ... }` (201).
- **Fetch**: Output `{ "propertyId": "uuid", ... , "ratings": { "average": "number" } }` (200).
- **List**: Input query params (e.g., `?hostId=uuid`); Output `{ "properties": [...], "total": "number" }` (200).

#### Validation Rules
- Title: 10-100 chars, sanitized (400).
- Price: >0, ≤2 decimals (400).
- Location: Geocode to lat/lng (±0.001 accuracy, 400 if fail).
- Availability: Future dates, no self-overlaps (409).
- Images: 1-10, ≤5MB each (413).
- Auth: Host-owned (403).

#### Performance Criteria
- Latency: <300ms (fetch/create).
- Scalability: Indexes on hostId, location (GIN for JSON).
- Optimization: Redis cache for hot listings (TTL 30min).

### 3. Search and Filtering
#### Functional Requirements
- Users search properties by location, price, guests, amenities, dates.
- Results paginated, sorted (price/rating), with facets (e.g., amenity counts).
- Availability filtered dynamically.

#### Technical Requirements
- Database: Queries on `Properties` + JOIN `Bookings` for availability.
- Auth: Public; personalized via user prefs (optional).
- Integration: PostGIS extension for geospatial; optional Elasticsearch for full-text.
- Logging: Query logs for analytics.

#### API Endpoints
| Method | Endpoint                  | Description                          |
|--------|---------------------------|--------------------------------------|
| GET    | `/api/search/properties`  | Advanced search/filter               |

#### Input/Output Specifications
- **Search**: Input query `?query=string&location=string&lat=number&lng=number&radius=number&priceMin=number&...&page=number&limit=number`; Output `{ "results": [...], "total": "number", "facets": { ... } }` (200).

#### Validation Rules
- Geo: Lat [-90,90], lng [-180,180], radius >0 (400).
- Dates: checkIn < checkOut, future (400).
- Amenities: Match enum (400).

#### Performance Criteria
- Latency: <500ms (complex filters).
- Scalability: Pagination (limit≤50); sharding for >100k properties.
- Optimization: Composite indexes; Redis facets (TTL 5min).

### 4. Booking Management
#### Functional Requirements
- Guests create bookings (validate availability, calc total).
- Track statuses: pending/confirmed/canceled/completed.
- Hosts/guests cancel; auto-complete post-checkOut.
- Prevent double-bookings.

#### Technical Requirements
- Database: `Bookings` table (id: UUID PK, propertyId: FK, guestId: FK, status: enum, checkIn/Out: date, guests: int, totalAmount: decimal); triggers for status updates.
- Auth: Guest create, host/guest view own (RBAC).
- Integration: Transactional (SELECT FOR UPDATE on create).
- Logging: Event logs for status changes.

#### API Endpoints
| Method | Endpoint                  | Description                          |
|--------|---------------------------|--------------------------------------|
| POST   | `/api/bookings`           | Create booking                       |
| GET    | `/api/bookings/{id}`      | Fetch booking                        |
| PUT    | `/api/bookings/{id}/status` | Update status                     |
| DELETE | `/api/bookings/{id}`      | Cancel booking                       |
| GET    | `/api/bookings`           | List bookings                        |

#### Input/Output Specifications
- **Create**: Input `{ "propertyId": "uuid", "guestId": "uuid", "checkIn": "string", "checkOut": "string", "guests": "number", "totalAmount": "number" }`; Output `{ "bookingId": "uuid", "status": "pending" }` (201).
- **List**: Input query `?role=guest/host&status=enum&page=number`; Output `{ "bookings": [...], "total": "number" }` (200).

#### Validation Rules
- Dates: No overlap with existing (409); checkIn < checkOut (400).
- Guests: ≤ property max (400).
- Amount: Auto-calc validated (400).
- Auth: Role-enforced (403).

#### Performance Criteria
- Latency: <250ms (create with lock).
- Scalability: 5k concurrent via queuing.
- Optimization: Indexes on dates/propertyId; Redis lists (TTL 10min).

### 5. Payment Integration
#### Functional Requirements
- Guests pay on booking; hosts get payouts post-stay.
- Support multi-currency, refunds on cancel.
- Handle failures, partial payments.

#### Technical Requirements
- Database: `Payments` table (id: UUID PK, bookingId: FK, amount: decimal, currency: string, status: enum, stripeId: string).
- Auth: Guest pay, host view payouts (RBAC).
- Integration: Stripe/PayPal APIs (intents, webhooks); currency conversion via Stripe.
- Security: PCI via Stripe; encrypt tokens.

#### API Endpoints
| Method | Endpoint                          | Description                          |
|--------|-----------------------------------|--------------------------------------|
| POST   | `/api/payments/intent`            | Create intent                        |
| POST   | `/api/payments/confirm`           | Confirm payment                      |
| POST   | `/api/payments/refund/{bookingId}`| Refund                               |
| GET    | `/api/payments/{bookingId}`       | Fetch details                        |
| GET    | `/api/payments/user`              | List history                         |

#### Input/Output Specifications
- **Intent**: Input `{ "bookingId": "uuid", "currency": "string", "amount": "number" }`; Output `{ "intentId": "string", "clientSecret": "string" }` (200).
- **Fetch**: Output `{ "paymentId": "string", "status": "enum", ... }` (200).

#### Validation Rules
- Amount: Matches booking (400).
- Currency: ISO-supported (400).
- Refund: Canceled bookings only (409).
- Webhook: Signature verify (401).

#### Performance Criteria
- Latency: <300ms (intent).
- Scalability: Idempotent; batch payouts (Celery).
- Optimization: Redis rates (TTL 1h).

### 6. Reviews and Ratings
#### Functional Requirements
- Guests review post-completion (rating 1-5, text, photos).
- Hosts respond; aggregates shown on listings.
- Moderation: Flag/remove inappropriate.

#### Technical Requirements
- Database: `Reviews` table (id: UUID PK, bookingId: FK, rating: int, body: text, images: JSON); `ReviewResponses` (reviewId: FK, response: text). Triggers for avg rating update in Properties.
- Auth: Guest submit, host respond (RBAC).
- Storage: S3 for review images.
- Integration: Profanity filter (e.g., Perspective API).
- Logging: Moderation audits.

#### API Endpoints
| Method | Endpoint                  | Description                          |
|--------|---------------------------|--------------------------------------|
| POST   | `/api/reviews`            | Submit review                        |
| GET    | `/api/reviews/{bookingId}`| Fetch review                         |
| PUT    | `/api/reviews/{reviewId}` | Update review                        |
| POST   | `/api/reviews/{reviewId}/response` | Host response                  |
| GET    | `/api/reviews/property/{propertyId}` | List property reviews        |

#### Input/Output Specifications
- **Submit**: Input `{ "bookingId": "uuid", "rating": "number", "title": "string", "body": "string", "images": ["file"] }`; Output `{ "reviewId": "uuid", ... }` (201).
- **List**: Input query `?page=number&sort=recent`; Output `{ "reviews": [...], "averageRating": "number" }` (200).

#### Validation Rules
- Booking: Completed, one-per (409).
- Rating: 1-5 int (400).
- Body: 10-1000 chars, sanitized (400).
- Images: 0-5, ≤2MB (413).

#### Performance Criteria
- Latency: <200ms (submit).
- Scalability: Denormalized avgs.
- Optimization: Redis avgs (TTL 1h, invalidate on update).

### 7. Notifications System
#### Functional Requirements
- Trigger notifications for events (booking, payment, review).
- Channels: Email, in-app (WebSocket).
- Users configure prefs (e.g., disable emails).

#### Technical Requirements
- Database: `Notifications` table (id: UUID PK, userId: FK, type: enum, title/body: string, read: bool); `UserPreferences` (userId: FK, prefs: JSON).
- Auth: User-specific (RBAC).
- Integration: SendGrid for email; Redis pub/sub for WebSockets.
- Logging: Delivery tracking.

#### API Endpoints
| Method | Endpoint                  | Description                          |
|--------|---------------------------|--------------------------------------|
| POST   | `/api/notifications/preferences` | Update prefs                  |
| GET    | `/api/notifications/user` | Fetch notifications                  |
| DELETE | `/api/notifications/{id}`| Mark read                            |
| POST   | `/api/notifications/trigger` | Internal trigger                |

#### Input/Output Specifications
- **Prefs**: Input `{ "emailBookings": "bool", ... }`; Output `{ "preferences": {...} }` (200).
- **Fetch**: Input query `?type=enum&unread=bool`; Output `{ "notifications": [...], "totalUnread": "number" }` (200).

#### Validation Rules
- Prefs: Bool fields (400).
- Trigger: Valid event (400, internal).

#### Performance Criteria
- Latency: <100ms in-app, <5s email.
- Scalability: Queued (Celery), 10k/day.
- Optimization: Batch fetches; unread cache (TTL 1d).

### 8. Admin Dashboard
#### Functional Requirements
- Admins manage (CRUD) users/properties/bookings/payments/reviews.
- View analytics (revenue, usage metrics).
- Bulk actions (e.g., suspend users).

#### Technical Requirements
- Database: Views for analytics (e.g., revenue SUM over dates); `AdminAudits` table for actions.
- Auth: Admin RBAC only.
- Integration: SQL aggregates; optional BI tool (Metabase).
- Logging: All actions audited.

#### API Endpoints
| Method | Endpoint                          | Description                          |
|--------|-----------------------------------|--------------------------------------|
| GET    | `/api/admin/users`                | List/search users                    |
| DELETE | `/api/admin/users/{userId}`       | Suspend/delete user                  |
| GET    | `/api/admin/properties`           | List/audit properties                |
| PUT    | `/api/admin/properties/{id}/approve` | Approve/reject                    |
| GET    | `/api/admin/bookings`             | List bookings                        |
| GET    | `/api/admin/analytics`            | Metrics dashboard                    |

#### Input/Output Specifications
- **Users List**: Input query `?role=enum&search=string&page=number`; Output `{ "users": [...], "total": "number" }` (200).
- **Analytics**: Input query `?period=month&from=string&to=string`; Output `{ "totalRevenue": "number", "activeListings": "number", ... }` (200).

#### Validation Rules
- Search: Sanitized (400).
- Actions: No active deps (409).
- Dates: Valid range (400).

#### Performance Criteria
- Latency: <500ms (analytics).
- Scalability: Read replicas; paginate.
- Optimization: Pre-compute metrics (Redis TTL 1h).

## Global Technical Requirements
- **Database**: PostgreSQL; tables as noted; migrations (Alembic); constraints (FKs, uniques).
- **API**: REST (OpenAPI spec); GraphQL optional for queries.
- **Auth/Security**: JWT + RBAC middleware; rate limiting (10-100/min); HTTPS, encryption (at-rest via pg_crypto).
- **File Storage**: S3/Cloudinary; presigned URLs (expiry 1h).
- **Integrations**: As per features; env vars for keys.
- **Error/Logging**: Global handler (trace_id); ELK stack.
- **Testing**: Pytest (unit/int); Postman for API; Locust loads.

## Global Non-Functional Requirements
- **Scalability**: Modular (microservices); horizontal via K8s/load balancer; handle 10k users/day.
- **Security**: OWASP Top 10; firewalls; data encryption.
- **Performance**: Redis caching; query opt (EXPLAIN <50ms); <1% error rate.
- **Reliability**: 99.9% uptime; backups daily.
- **Compliance**: GDPR (data deletion); PCI via Stripe.

## Appendix: Implementation Notes
- Deployment: Docker Compose for dev; K8s prod.
- Monitoring: Prometheus/Grafana; pgBadger.
- Changes: Version-controlled; changelog in repo.

This document is the single source of truth, ensuring full capture of functional (behaviors) and technical (implementation) requirements per feature.
