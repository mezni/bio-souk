# API Contracts: Farm and Certification Management

## Base URL
`POST /api/v1/farms` / `GET /api/v1/farms/{id}` / `PATCH /api/v1/farms/{id}`

## Contract: Create Farm

**Method**: POST  
**Endpoint**: `/api/v1/farms`  
**Description**: Register a new farm for an authenticated farmer.

**Request Schema**:
```json
{
  "name": "string (max 200 chars, required)",
  "location": "string (required, geographic region or location description)",
  "size": "number (required, decimal-safe area measurement, e.g., 15.5 for hectares)"
}
```

**Response (201 Created)**:
```json
{
  "id": "uuid",
  "name": "string",
  "location": "string",
  "size": "number",
  "status": "string (enum: active | inactive | suspended)",
  "owner_id": "uuid",
  "created_at": "ISO 8601 timestamp",
  "updated_at": "ISO 8601 timestamp"
}
```

**Errors**:
- 400: Validation error (missing required fields, invalid format)
- 401: Authentication required (invalid/missing JWT)
- 403: Farmer attempting to update another farmer's farm (access denied)

---

## Contract: Update Farm Profile

**Method**: PATCH  
**Endpoint**: `/api/v1/farms/{id}`  
**Description**: Update farm profile details for an authenticated owner.

**Request Schema**:
```json
{
  "name": "string (max 200 chars, optional)",
  "description": "string (optional)",
  "location": "string (optional)"
}
```

**Response (200 OK)**:
Same as Create Farm response schema.

**Errors**:
- 400: Validation error (invalid field format)
- 401: Authentication required
- 403: Access denied (farmer cannot update another's farm)
- 404: Farm not found

---

## Contract: Submit Certification

**Method**: POST  
**Endpoint**: `/api/v1/certifications`  
**Description**: Submit a new certification for a farm.

**Request Schema**:
```json
{
  "farm_id": "uuid (required, must belong to authenticated farmer)",
  "certification_type": "string (required, e.g., organic, sustainable, fair-trade)",
  "expiration_date": "string (ISO 8601 date, required, must be future date)"
}
```

**Response (201 Created)**:
```json
{
  "id": "uuid",
  "farm_id": "uuid",
  "certification_type": "string",
  "status": "string (enum: pending | approved | rejected | expired)",
  "expiration_date": "ISO 8601 date",
  "status": "pending",
  "submission_date": "ISO 8601 timestamp",
  "audit_trail": [
    {
      "event": "submission",
      "timestamp": "ISO 8601 timestamp",
      "details": "Certification submitted by farmer"
    }
  ]
}
```

**Errors**:
- 400: Validation error (farm not owned by farmer, invalid certification type, expired date)
- 401: Authentication required
- 403: Farmer attempting to certify farm they don't own

---

## Contract: Review Certification

**Method**: PATCH  
**Endpoint**: `/api/v1/certifications/{id}/review`  
**Description**: Administrator reviews and evaluates a pending certification.

**Request Schema**:
```json
{
  "decision": "string (required, enum: approved | rejected)",
  "feedback": "string (optional, free text feedback from reviewer)"
}
```

**Response (200 OK)**:
```json
{
  "id": "uuid",
  "farm_id": "uuid",
  "certification_type": "string",
  "status": "string (enum: approved | rejected)",
  "decision_date": "ISO 8601 timestamp",
  "reviewer_id": "uuid",
  "feedback": "string (optional)",
  "audit_trail": [
    ...existing entries...,
    {
      "event": "review",
      "timestamp": "ISO 8601 timestamp",
      "decision": "approved | rejected",
      "reviewer_id": "uuid"
    }
  ]
}
```

**Errors**:
- 400: Validation error (certification not in pending state, invalid decision)
- 401: Authentication required (admin privileges needed)
- 403: Non-admin user attempting to review certification
- 404: Certification not found

---

## Contract: Get Certification Status

**Method**: GET  
**Endpoint**: `/api/v1/certifications/{id}`  
**Description**: Retrieve certification status and details.

**Response (200 OK)**:
```json
{
  "id": "uuid",
  "farm_id": "uuid",
  "certification_type": "string",
  "status": "string (pending | approved | rejected | expired)",
  "expiration_date": "ISO 8601 date",
  "submission_date": "ISO 8601 timestamp",
  "decision_date": "ISO 8601 timestamp (nullable)",
  "reviewer_id": "uuid (nullable)",
  "feedback": "string (nullable)",
  "audit_trail": [
    {
      "event": "submission | review | expiration",
      "timestamp": "ISO 8601 timestamp",
      "details": "free text",
      "decision": "approved | rejected (nullable)",
      "reviewer_id": "uuid (nullable)"
    }
  ]
}
```

**Errors**:
- 404: Certification not found

---

## Contract: Farmer Association Management

**Method**: POST / DELETE  
**Endpoint**: `/api/v1/farmer-associations` / `/api/v1/farmer-associations/{id}/members/{farmer_id}`  
**Description**: Manage farmer group memberships.

**Request (POST - join association)**:
```json
{
  "association_id": "uuid (required)"
}
```

**Response (200 OK)**:
```json
{
  "association_id": "uuid",
  "farmer_id": "uuid",
  "joined_at": "ISO 8601 timestamp",
  "role": "string (member, organizer)"
}
```

**Errors**:
- 400: Farmer already member of association, association not found, association inactive
- 401: Authentication required
- 403: Farmer cannot join inactive association

---

## Contract: Health Check

**Method**: GET  
**Endpoint**: `/api/v1/health`  
**Description**: Service health check with domain availability.

**Response (200 OK)**:
```json
{
  "status": "healthy",
  "domains": {
    "postgresql": "connected",
    "jwt": "valid",
    "outbox": "operational"
  }
}
```