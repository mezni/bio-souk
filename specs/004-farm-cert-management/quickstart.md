# Quickstart: Farm and Certification Management Validation

## Prerequisites

- Rust 1.75+ with `cargo` installed
- Node.js 20+ with `npm` installed
- PostgreSQL 15+ running and accessible
- Environment variables configured (see `.env.example`)

## Setup

### 1. Repository Setup

```bash
# Clone and initialize submodules
git clone <repo-url>
cd bio-souk

# Install backend dependencies
cd backend
cargo build --release

# Install frontend dependencies
cd ../frontend
npm install
```

### 2. Database Setup

```bash
# Create database and user
createdb biosouk

# Apply migrations (domain schemas per constitution)
cd backend
cargo run -- migrate --create_schema

# Verify schemas created
psql -d biosouk -c "\dn"
```

### 3. Environment Configuration

Copy `.env.example` to `.env` and set:

```bash
# Database
DATABASE_URL="postgres://biosouk_user:password@localhost:5432/biosouk?sslmode=disable"

# JWT Configuration
JWT_SECRET="change-this-in-production-minimum-32-bytes"
JWT_EXPIRATION="1d"
ARGON2ID_SALT="change-this-in-production-16-bytes"

# Server
HOST="0.0.0.0"
PORT="8080"
```

### 4. Start the Server

```bash
# Backend
cd backend
cargo run

# Frontend (separate terminal)
cd frontend
npm run dev
```

The server will be available at `http://localhost:8080`.

## Validation Scenarios

### Scenario 1: Farm Registration

**Goal**: Verify a farmer can register a new farm.

**Steps**:
1. Authenticate as farmer (obtain JWT via `/api/v1/auth/login` with test credentials)
2. POST `/api/v1/farms` with valid payload:
   ```json
   {"name": "Sunny Valley Farm", "location": "Region A", "size": 15.5}
   ```
3. Verify response contains farm with `status: "active"`
4. Verify farm appears in GET `/api/v1/farms` list

**Expected Outcome**: Farm successfully created and visible in listing. Returns 201 Created.

---

### Scenario 2: Invalid Farm Registration

**Goal**: Verify validation rejects missing required fields.

**Steps**:
1. POST `/api/v1/farms` with incomplete payload (e.g., missing `size`)
2. Verify response returns 400 Bad Request

**Expected Outcome**: Validation error returned. Farm not created.

---

### Scenario 3: Certification Submission

**Goal**: Verify a farmer can submit a certification for their farm.

**Steps**:
1. Create a farm (Scenario 1)
2. Obtain JWT for the farm's owner
3. POST `/api/v1/certifications` with payload:
   ```json
   {"farm_id": "<farm-uuid>", "certification_type": "organic", "expiration_date": "2027-12-31"}
   ```
4. Verify certification created with `status: "pending"`
5. Check audit trail includes submission event

**Expected Outcome**: Certification entered review queue. Returns 201 Created with audit trail.

---

### Scenario 4: Certification Review by Administrator

**Goal**: Verify administrator can review and decide on a certification.

**Steps**:
1. Have a pending certification (from Scenario 3)
2. Obtain JWT with admin role
3. PATCH `/api/v1/certifications/{id}/review` with payload:
   ```json
   {"decision": "approved", "feedback": "All requirements met"}
   ```
4. Verify certification status updated to "approved"
5. Verify decision_date populated
6. Verify audit trail includes review event with decision

**Expected Outcome**: Certification reaches final state "approved" with complete audit trail.

---

### Scenario 5: Expired Certification Detection

**Goal**: Verify expired certifications are flagged as invalid.

**Steps**:
1. Create a certification with a past expiration date
2. Query certification status via GET `/api/v1/certifications/{id}`
3. Verify status flagged as "expired"
4. Verify system excludes expired certifications from valid results

**Expected Outcome**: Certification marked expired and excluded from valid certification queries.

---

### Scenario 6: RBAC Access Control

**Goal**: Verify role-based access control works correctly.

**Steps**:
1. Login as farmer, attempt to review another farmer's certification → should return 403
2. Login as admin, attempt to create a farm → should succeed (admin privileges)
3. Login as producer, attempt to administer certification → should return 403

**Expected Outcome**: Access denied for unauthorized roles; authorized roles succeed.

## Development Commands

### Run Tests

```bash
# Backend unit and integration tests
cd backend
cargo test

# frontend tests
cd frontend
npm test

# linting
cd backend
cargo run -- check
cd frontend
npm run lint
```

### Run Specific Validation

```bash
# Quick validation of farm/certification workflow
cd backend
cargo test -- farm certification

# Run with debug logging
RUST_LOG=debug cargo run
```

## Expected Outcomes Summary

All validation scenarios should complete successfully with:
- Farm registration: 201 Created with valid farm data
- Certification submission: 201 Created, enters pending review
- Certification review: 200 OK, reaches approved/rejected state
- Expired certification: Flagged as expired, excluded from valid results
- RBAC: Correct access denied/granted per role
- Audit trail: Complete decision history preserved