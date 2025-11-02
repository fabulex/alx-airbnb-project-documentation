from pathlib import Path

# Markdown document content
markdown_content = """# 🏠 Airbnb Clone Backend — Features & Functionalities Document

## 1. Overview

The Airbnb Clone backend powers a platform for users to list, discover, and book accommodations. It manages authentication, property listings, booking lifecycle, payments, and admin controls.

---

## 2. Core Modules

### 2.1 User Authentication & Authorization

**Functionalities**
- **User Registration:** Signup (Guests/Hosts) via email/password or OAuth (Google, Facebook, Apple)
- **Login & Logout:** Secure JWT sessions, refresh tokens, and logout functionality
- **Password Management:** Reset via email link, encrypted storage (bcrypt/Argon2)
- **Role Management:** Guest, Host, and Admin roles with RBAC
- **Profile Management:** Update user info, images, verification status

**Technical Notes**
- Auth: JWT/OAuth 2.0
- Tables: `users`, `roles`, `sessions`

---

### 2.2 Property Management

**Functionalities**
- **Create/Edit/Delete Listing:** Title, description, location, price, amenities, availability
- **Image Management:** Upload multiple images (AWS S3, Cloudinary)
- **View Listings:** Publicly available listings with pagination/filtering

**Technical Notes**
- Tables: `properties`, `property_images`, `amenities`
- Cloud integration for image storage

---

### 2.3 Booking System

**Functionalities**
- **Search Listings:** Location, price, dates, amenities, guests
- **Create Booking:** Validate dates, calculate cost, record details
- **Modify/Cancel:** Manage booking lifecycle and availability
- **Booking History:** Past and upcoming reservations

**Booking Lifecycle:** `Pending → Confirmed → Completed → Cancelled`

**Technical Notes**
- Tables: `bookings`, `booking_status`, `calendar_availability`
- Links to payments and notifications

---

### 2.4 Payment System

**Functionalities**
- **Process Payments:** Stripe/PayPal integration with tokenization
- **Refunds:** Automated refunds tied to cancellation policy
- **Host Payouts:** Release funds post-checkout
- **Transaction Records:** Maintain logs and reports

**Technical Notes**
- Tables: `payments`, `payouts`, `refunds`, `transactions`
- PCI-DSS compliant, HTTPS required

---

### 2.5 Reviews & Ratings

**Functionalities**
- **Guest Reviews:** After booking completion
- **Host Responses:** Replies to reviews
- **Moderation:** Admin content control
- **Aggregated Ratings:** Property and host averages

**Technical Notes**
- Tables: `reviews`, `review_responses`

---

### 2.6 Notifications System

**Functionalities**
- **In-App & Email Notifications:** Booking updates, cancellations, payment alerts
- **Push Notifications (optional):** Firebase or WebPush integration

**Technical Notes**
- Queued dispatch (Redis, RabbitMQ)
- Tables: `notifications`, `email_logs`

---

### 2.7 Admin Dashboard

**Functionalities**
- Manage Users, Listings, Bookings, Payments
- Analytics & Reporting: Metrics and logs
- Admin Moderation: Suspend users or remove content

**Technical Notes**
- Role middleware verification
- Audit logs

---

## 3. Technical Architecture

### Database Layer
- PostgreSQL/MySQL with ORM (SQLAlchemy/TypeORM)
- Relationships: Users ↔ Properties ↔ Bookings ↔ Payments ↔ Reviews

### API Layer
- RESTful API (`/auth`, `/properties`, `/bookings`, `/payments`, `/reviews`)
- OpenAPI (Swagger) documentation

### Storage
- Cloud storage (S3/Cloudinary)
- Redis caching for sessions/search

### Logging & Monitoring
- Centralized logs (ELK stack)
- Metrics (Prometheus, Grafana)

### Testing
- Unit & integration testing (Pytest/Jest)
- API testing (Postman/Newman)

---

## 4. Non-Functional Requirements

| Category | Description |
|-----------|--------------|
| Scalability | Modular, supports horizontal scaling |
| Security | Encrypted data, rate limiting, firewalls |
| Performance | <300ms API response time |
| Availability | 99.9% uptime |
| Compliance | GDPR, PCI-DSS |
| Documentation | OpenAPI (Swagger) |
| Backup | Automated daily backups |

---

## 5. Summary

This backend delivers:
- Secure, modular APIs
- Scalable architecture
- Role-based access and workflows
- Integrations for payments, notifications, and file storage
"""
