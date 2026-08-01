# Backend Technical Design Document (TDD) - PG Management System

## TECHNOLOGY STACK
- **Language**: Python 3.10+
- **Framework**: FastAPI (Core APIs), Django (Admin Panel & Built-in ORM features if used alongside)
- **Database**: PostgreSQL 15+
- **ORM**: SQLAlchemy (Async) / Django ORM
- **Authentication**: JWT (JSON Web Tokens)
- **Architecture**: Layered Architecture (Controller -> Service -> Repository)
- **Validation**: Pydantic v2

---

## MODULE 1: AUTHENTICATION

### 1. Module Overview
The Authentication module is responsible for securing the API endpoints, managing user identities (Admin/Staff), issuing JWT tokens, and handling role-based access control (RBAC).

### 2. Database Design

**Table: `users`**
- **Purpose**: Stores administrator and staff credentials.
- **Primary Key**: `id` (UUID)
- **Columns**:
  - `id` (UUID, Not Null, Default: uuid4)
  - `email` (Varchar(255), Not Null, Unique)
  - `password_hash` (Varchar(255), Not Null)
  - `first_name` (Varchar(100), Not Null)
  - `last_name` (Varchar(100), Nullable)
  - `role` (Varchar(50), Not Null, Default: 'ADMIN')
  - `is_active` (Boolean, Not Null, Default: True)
  - `created_at` (Timestamp, Not Null, Default: now())
  - `updated_at` (Timestamp, Not Null, Default: now())
- **Indexes**: `idx_users_email`

### 3. Entity Relationship
- Currently standalone. Future: One-to-Many with `Roles` or `Permissions`.

### 4. API Endpoints
| Method | Endpoint | Description | Authentication Required |
| :--- | :--- | :--- | :--- |
| POST | `/api/v1/auth/login` | Authenticate and issue JWT tokens | No |
| POST | `/api/v1/auth/refresh` | Refresh access token | Yes (Refresh Token) |
| GET | `/api/v1/auth/me` | Get current user profile | Yes |

### 5. Request Payload
**POST `/api/v1/auth/login`**
```json
{
  "email": "admin@pg.com",
  "password": "SecurePassword123!"
}
```

### 6. Response Payload
**Success (200 OK)**
```json
{
  "access_token": "eyJhbGciOiJIUzI1...",
  "refresh_token": "eyJhbGciOiJIUzI1...",
  "token_type": "bearer",
  "expires_in": 3600
}
```
**Error (401 Unauthorized)**
```json
{
  "detail": "Invalid email or password."
}
```

### 7. Validation Rules
- **Required**: Email and Password are required.
- **Format**: Email must be a valid email format.
- **Business**: Account must have `is_active=True`.

### 8. Business Logic
Receive Credentials -> Validate Email Format -> Fetch User from DB -> Check `is_active` -> Verify bcrypt password hash -> Generate Access & Refresh JWTs -> Return Response.

---

## MODULE 2: PG MANAGEMENT

### 1. Module Overview
Manages the physical properties, including PG buildings, rooms, and individual bed inventory.

### 2. Database Design

**Table: `pg_properties`**
- **Purpose**: Stores PG building details.
- **Primary Key**: `id` (UUID)
- **Columns**: `name` (Varchar), `address` (Text), `total_rooms` (Int), `status` (Varchar).

**Table: `rooms`**
- **Purpose**: Stores rooms within a PG.
- **Primary Key**: `id` (UUID)
- **Foreign Key**: `pg_id` references `pg_properties(id)`
- **Columns**: `room_number` (Varchar), `capacity` (Int), `floor` (Int).

**Table: `beds`**
- **Purpose**: Stores individual beds.
- **Primary Key**: `id` (UUID)
- **Foreign Key**: `room_id` references `rooms(id)`
- **Columns**: `bed_identifier` (Varchar), `is_occupied` (Boolean).

### 3. Entity Relationship
- **One-to-Many**: `pg_properties` (1) to `rooms` (N). Cascade Delete.
- **One-to-Many**: `rooms` (1) to `beds` (N). Cascade Delete.

### 4. API Endpoints
| Method | Endpoint | Description | Authentication Required |
| :--- | :--- | :--- | :--- |
| GET | `/api/v1/pgs` | List all PGs | Yes |
| POST | `/api/v1/pgs` | Create a PG | Yes |
| GET | `/api/v1/pgs/{id}/rooms` | List rooms for a PG | Yes |

### 5. Request Payload
**POST `/api/v1/pgs`**
```json
{
  "name": "Sunshine PG",
  "address": "123 Main St",
  "total_rooms": 10
}
```

### 6. Response Payload
**Success (201 Created)**
```json
{
  "id": "uuid",
  "name": "Sunshine PG",
  "status": "Active"
}
```

### 7. Validation Rules
- **Duplicate**: PG Name must be unique.
- **Business**: Cannot delete a room if beds are occupied.

### 8. Business Logic
Receive PG Data -> Validate Unique Name -> Insert PG Record -> Return PG Object.

---

## MODULE 3: MEMBER MANAGEMENT (Including Rent)

### 1. Module Overview
Handles the tenant lifecycle, demographics, document storage, room allocation, and monthly rent generation.

### 2. Database Design

**Table: `members`**
- **Purpose**: Tenant profile.
- **Primary Key**: `id` (UUID)
- **Foreign Key**: `bed_id` references `beds(id)`
- **Columns**: `first_name`, `last_name`, `mobile`, `email`, `status` (Active, Notice, Inactive), `base_rent` (Decimal).

**Table: `documents`**
- **Purpose**: KYC Documents.
- **Foreign Key**: `member_id` references `members(id)`
- **Columns**: `doc_type` (Aadhaar, PAN), `file_path` (Varchar).

**Table: `rent_records`**
- **Purpose**: Monthly billing.
- **Foreign Key**: `member_id` references `members(id)`
- **Columns**: `billing_month` (Date), `amount` (Decimal), `due_date` (Date), `payment_status` (Pending, In Review, Paid, Rejected), `transaction_id` (Varchar), `screenshot_path` (Varchar).

### 3. Entity Relationship
- **One-to-One**: `members` (1) to `beds` (1).
- **One-to-Many**: `members` (1) to `documents` (N).
- **One-to-Many**: `members` (1) to `rent_records` (N).

### 4. API Endpoints
| Method | Endpoint | Description | Authentication Required |
| :--- | :--- | :--- | :--- |
| GET | `/api/v1/members` | List members with filters | Yes |
| POST | `/api/v1/members` | Onboard new member | Yes |
| GET | `/api/v1/members/{id}/rents` | Get payment history | Yes |

### 5. Request Payload
**POST `/api/v1/members`**
```json
{
  "first_name": "Rahul",
  "mobile": "9876543210",
  "bed_id": "uuid",
  "base_rent": 6000.00
}
```

### 6. Response Payload
**Success (201 Created)**
```json
{
  "id": "uuid",
  "status": "Active"
}
```

### 7. Validation Rules
- **Duplicate**: Mobile number must be unique globally for active members.
- **Relationship**: `bed_id` must exist and `is_occupied` must be False.

### 8. Business Logic
Member Registration -> Validate Mobile Unique -> Validate PG/Room/Bed -> Mark Bed as Occupied -> Insert Member -> Generate First Rent Record -> Upload KYC Async -> Return Response.

---

## MODULE 4: PAYMENT VERIFICATION

### 1. Module Overview
Facilitates the member-facing payment submission and admin-facing manual verification workflow.

### 2. Database Design
- Utilizes the `rent_records` table defined in Member Management.

### 3. Entity Relationship
- Updates state on `rent_records`.

### 4. API Endpoints
| Method | Endpoint | Description | Authentication Required |
| :--- | :--- | :--- | :--- |
| POST | `/api/v1/payments/submit` | Member submits payment proof | No (Uses secure token) |
| POST | `/api/v1/payments/{id}/verify` | Admin approves/rejects payment| Yes |

### 5. Request Payload
**POST `/api/v1/payments/{id}/verify`**
```json
{
  "action": "APPROVE",
  "remarks": "Verified via Bank Statement"
}
```

### 6. Response Payload
**Success (200 OK)**
```json
{
  "id": "uuid",
  "payment_status": "Paid"
}
```

### 7. Validation Rules
- **Business**: Can only verify records currently in 'In Review' status.
- **Required**: Remarks required if action is 'REJECT'.

### 8. Business Logic
Fetch Rent Record -> Check Status is 'In Review' -> If APPROVE, set status='Paid', update 'payment_date' -> If REJECT, set status='Rejected', append remarks -> Return Updated Record.

---

## MODULE 5: DASHBOARD

### 1. Module Overview
Aggregates real-time KPIs, operational metrics, and alerts for the admin interface.

### 2. Database Design
- No dedicated tables; uses aggregation queries across `members`, `rooms`, `beds`, and `rent_records`.

### 3. Entity Relationship
- N/A

### 4. API Endpoints
| Method | Endpoint | Description | Authentication Required |
| :--- | :--- | :--- | :--- |
| GET | `/api/v1/dashboard/kpis` | Get high-level stats | Yes |
| GET | `/api/v1/dashboard/alerts` | Get pending actions | Yes |

### 5. Request Payload
- N/A (GET requests)

### 6. Response Payload
**Success (200 OK)**
```json
{
  "total_members": 150,
  "occupancy_rate": 85.5,
  "pending_verifications": 12
}
```

### 7. Validation Rules
- **Auth**: User must have Admin role.

### 8. Business Logic
Receive Request -> Query `COUNT(members)` -> Query `SUM(rent)` where status='Pending' -> Query Occupied Beds / Total Beds -> Format JSON -> Return.

---

## 9. Service Layer
The Service Layer encapsulates pure business logic, keeping controllers lean.
- **AuthenticationService**: Handles password hashing, JWT encoding/decoding, user validation.
- **PGService**: Manages hierarchy of PGs, Rooms, and Beds. Handles capacity logic.
- **MemberService**: Handles tenant lifecycle, bed allocation/deallocation, and profile updates.
- **RentService**: Generates monthly invoices, calculates dues, handles prorated rent logic.
- **PaymentService**: Processes member submissions, file uploads, and admin verifications.
- **DashboardService**: Executes optimized aggregation queries for UI components.

---

## 10. Repository Layer
The Repository Layer abstracts direct database access (SQLAlchemy/Django ORM).
- **UserRepository**: `get_by_email()`, `update_login_time()`
- **PGRepository**: `get_all_pgs()`, `get_available_beds()`
- **MemberRepository**: `find_by_mobile()`, `list_active_members()`
- **RentRepository**: `get_pending_rents()`, `update_payment_status()`
- **Responsibility**: Provides a clean CRUD interface to the Service Layer, handling DB sessions and transactions.

---

## 11. Authentication & Authorization
- **JWT Implementation**: Uses `PyJWT`. Tokens signed using HS256 with a strong secret key.
- **Login Flow**: Client sends credentials -> Server verifies bcrypt hash -> Issues Access Token (15 mins) & Refresh Token (7 Days).
- **Access Token**: Passed in `Authorization: Bearer <token>` header. Contains `user_id` and `role`.
- **Refresh Token**: Stored in a secure, HttpOnly cookie to prevent XSS.
- **Role-Based Access (RBAC)**: Middleware checks the JWT `role` claim. Architecture supports `SUPER_ADMIN`, `PG_MANAGER`, `STAFF`.
- **Session Expiration**: Handled via JWT expiration (`exp` claim). Client automatically attempts refresh when 401 is received.

---

## 12. Exception Handling
Global exception handlers in FastAPI catch and standardize errors.
- **ValidationException** (422): Pydantic input validation failures.
- **NotFoundException** (404): Resource (e.g., Member, PG) does not exist.
- **DuplicateException** (409): Unique constraint violation (e.g., Duplicate Mobile).
- **BusinessException** (400): Logic errors (e.g., Deactivating an already inactive member).
- **AuthenticationException** (401): Invalid/expired tokens.
- **ForbiddenException** (403): User lacks role permissions.
- **DatabaseException** (500): Unhandled SQL errors.
- **FileUploadException** (400): Invalid MIME type or size exceeded.

---

## 13. Logging
- **Strategy**: Asynchronous logging using standard Python `logging` or `Loguru`. JSON formatted logs for Datadog/ELK ingestion.
- **Request/Response Logs**: Middleware logs `method`, `path`, `status_code`, `duration_ms`, `ip`.
- **Database Logs**: Slow query logging enabled in PostgreSQL; ORM warnings logged at application level.
- **Exception Logs**: Full stack traces logged with ERROR level.
- **Authentication Logs**: Log successful logins and failed attempts (for rate-limiting/auditing).
- **Payment Logs**: Dedicated audit trail for all state changes in `rent_records` (e.g., "Admin X verified Txn Y").

---

## 14. File Storage
- **Strategy**: Files are stored in an S3-compatible object store (AWS S3, MinIO) or a secure local `/uploads` volume.
- **Documents**: Aadhaar, PAN, Driving Licence.
- **Payment Screenshots**: Transaction proofs.
- **Naming Convention**: `UUID4.ext` to prevent filename collisions and enumeration attacks.
- **Database Storage**: Store only the relative path (e.g., `uploads/kyc/123e4567-e89b-12d3-a456-426614174000.pdf`) in PostgreSQL. Use pre-signed URLs for frontend access.

---

## 15. Dashboard Data APIs
Optimized APIs for the UI.
- **KPIs (`/api/v1/dashboard/kpis`)**: Total Members, Total Revenue, Outstanding Dues.
- **Charts (`/api/v1/dashboard/charts/revenue`)**: Monthly revenue grouped by PG.
- **Tables (`/api/v1/dashboard/tables/recent-payments`)**: Limit 10 most recent verifications.
- **Alerts (`/api/v1/dashboard/alerts`)**: Count of 'In Review' payments, 'Notice Period' members.
- **Aggregation Strategy**: Use PostgreSQL `GROUP BY`, `SUM()`, `COUNT()` natively to avoid loading large datasets into Python memory.

---

## 16. Payment Verification Logic
Complete backend workflow:
1. **Rent Due**: System generates `rent_record` with status `Pending`.
2. **WhatsApp Link Generated**: Secure JWT containing `rent_id` sent via Notification service.
3. **Member Opens Link**: Frontend decodes JWT, displays rent amount.
4. **Member Pays & Uploads**: File is sanitized, saved to storage.
5. **Status Update**: `RentService` updates `rent_records.screenshot_path` and sets status to `In Review`.
6. **Admin Review**: Admin fetches `In Review` records.
7. **Approval**: Admin clicks Approve -> `PaymentService` verifies state -> Sets `Paid`.
8. **Rejection**: Admin clicks Reject -> `PaymentService` sets `Rejected`, triggers notification to member to retry.

---

## 17. Scheduled Jobs (Cron Jobs)
Managed via Celery, APScheduler, or Django Celery Beat.
- **Rent Generation (Monthly, 1st Day)**: Creates new billing cycle records for Active members.
- **Daily Due Reminder (Daily)**: Scans for `Pending` rent with `due_date < now()` and queues WhatsApp reminders.
- **48-Hour Reminder**: Follow-up for unpaid dues.
- **Expired Link Cleanup (Daily)**: Invalidates old payment tokens.
- **Dashboard Cache Refresh (Hourly)**: Pre-computes heavy KPI aggregations into Redis.

---

## 18. Notifications
Abstracted behind a `NotificationService` layer.
- **WhatsApp**: Primary channel (via Twilio/Meta API). Used for Payment Links, Success Receipts, Overdue Reminders.
- **Email (Future)**: Pluggable architecture ready for SMTP/SendGrid integration for Admin reports.
- **SMS (Future)**: Fallback gateway for OTPs and critical alerts.
- **Queueing**: Notifications are pushed to a Redis/RabbitMQ queue and processed asynchronously by background workers to prevent blocking API responses.

---

## 19. Security
- **JWT**: Stateless, cryptographically signed, short-lived access tokens.
- **Password Hashing**: `bcrypt` with appropriate work factor (rounds).
- **SQL Injection Prevention**: Enforced by using SQLAlchemy/Django ORM parameterized queries exclusively.
- **XSS Prevention**: Backend escapes inputs; Pydantic rejects malicious payloads.
- **CSRF Strategy**: If using cookies for Refresh tokens, `SameSite=Strict` and CSRF tokens are implemented.
- **Rate Limiting**: FastAPI `slowapi` or Redis-based rate limiting on sensitive endpoints (e.g., `/login`, `/upload`).
- **File Upload Security**: Strict MIME-type checking, file extension validation via `python-magic`, and 5MB size limits.
- **Input Sanitization**: Pydantic models strictly type-cast and strip trailing whitespaces.
- **API Throttling**: Nginx level or API Gateway limits.
- **CORS**: Configured to only allow requests from authorized frontend domains.
- **Secure Headers**: Implementation of HSTS, X-Content-Type-Options, X-Frame-Options via middleware.

---

## 20. Folder Structure
Production-ready FastAPI/Django layered architecture.

```text
app/
├── api/                # API Routers (v1 endpoints)
│   ├── v1/
│   │   ├── auth.py
│   │   ├── members.py
│   │   └── payments.py
├── core/               # App configuration, security settings, JWT logic
│   ├── config.py
│   └── security.py
├── database/           # DB connection, engine, session maker
│   └── session.py
├── models/             # SQLAlchemy / Django ORM DB Models
│   ├── member.py
│   └── rent.py
├── schemas/            # Pydantic schemas (Request/Response payload validation)
│   ├── member_schema.py
│   └── rent_schema.py
├── repositories/       # Direct DB query abstractions
│   └── member_repo.py
├── services/           # Business logic layer
│   └── payment_service.py
├── controllers/        # Request handlers (if separating from routes)
├── middleware/         # Custom CORS, Request Logging, Auth extraction
├── exceptions/         # Global exception classes and handlers
├── utils/              # Helper functions (e.g., formatting, UUID generation)
├── validators/         # Custom complex validation rules
├── tasks/              # Celery background jobs and crons
├── uploads/            # Local dev storage for KYC/Screenshots
└── tests/              # Pytest suite (Unit and Integration tests)
    ├── conftest.py
    └── api/
```
**Explanation**:
- **api/**: Defines the REST routes handling HTTP requests and responses.
- **models/**: Defines database tables and relationships.
- **schemas/**: Validates data moving in/out of the system using Pydantic.
- **repositories/**: Houses raw database query abstractions ensuring the service layer doesn't write SQL/ORM code.
- **services/**: Orchestrates domain business logic cleanly.
- **middleware/**: Intercepts requests for logging, timing, and security checks.
- **tasks/**: Contains all scheduled cron jobs and queue consumers.
- **tests/**: Contains all test coverage making the backend CI/CD ready.
