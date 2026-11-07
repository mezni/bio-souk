# Data Model: Farm and Certification Management

## Farm

**Description**: Represents an agricultural farm owned/managed by a farmer. Belongs to the Traceability Domain per BioSouk Constitution.

**Fields**:
- `id`: Unique farm identifier (UUID)
- `name`: Farm name provided by farmer (required, max 200 chars)
- `description`: Optional farm description
- `location`: Geographic location or region (required)
- `size`: Farm area measurement (required, decimal-safe)
- `status`: Farm operational status (active, inactive, suspended) (default: active)
- `owner_id`: Reference to Farmer who owns this farm (UUID, foreign key)
- `created_at`: Timestamp when farm was registered
- `updated_at`: Timestamp of last modification
- `certification_count`: Denormalized count of associated certifications (for performance)

**Relationships**:
- **Owner** (1): One Farmer owns the farm (farmer.owns → farm)
- **Certifications** (1-N): One farm has many certifications (farm.has_many → certification)

## Certification

**Description**: Represents a farm certification verifying compliance with standards. Belongs to the Traceability Domain per BioSouk Constitution. Mandatory strategic domain.

**Fields**:
- `id`: Unique certification identifier (UUID)
- `farm_id`: Reference to associated farm (UUID, foreign key)
- `certification_type`: Type of certification (e.g., organic, sustainable, fair-trade) (required)
- `status`: Certification status (pending, approved, rejected, expired) (default: pending)
- `expiration_date`: Date when certification becomes invalid (required)
- `submission_date`: When certification was submitted (default: now)
- `review_date`: When certification was last reviewed (nullable)
- `decision_date`: When approval/rejection decision was made (nullable)
- `reviewer_id`: Administrator who reviewed (UUID, foreign key, nullable)
- `feedback`: Optional feedback from reviewer (text, nullable)
- `audit_trail`: JSON log of all decision changes and timestamps (jsonb, required)
- `created_at`: Timestamp when certification was submitted
- `updated_at`: Timestamp of last status change

**Relationships**:
- **Farm** (N-1): One certification belongs to one farm (certification.belongs_to → farm)
- **Reviewer** (N-1): One administrator reviewed the certification (certification.reviewed_by → admin/user)

## Farmer

**Description**: Represents a user who owns and manages farms. Linked to the Platform Domain per BioSouk Constitution (owns User, Role, Permission, Authentication).

**Fields**:
- `id`: Unique farmer identifier (UUID)
- `email`: Authentication email (unique, required)
- `password_hash`: Hashed password using Argon2id (required, per constitution)
- `full_name`: Farmer's full name (required)
- `role`: RBAC role (Admin, Producer, Customer, Support) (default: Producer)
- `is_active`: Whether the account is active (default: true)
- `created_at`: Account creation timestamp
- `updated_at`: Account last update timestamp
- `last_login`: Last successful authentication timestamp (nullable)

**Relationships**:
- **Farms** (1-N): One farmer owns many farms (farmer.owns → farm)
- **Associations** (N-M): Farmer may membership multiple farmer associations (many-to-many via join table)

## FarmerAssociation

**Description**: Represents a group of farmers organized for shared purposes, resources, or bulk certification processes. Optional grouping; does not affect core farm ownership semantics.

**Fields**:
- `id`: Unique association identifier (UUID)
- `name`: Association name (required, max 200 chars)
- `description`: Optional description of association purpose
- `purpose`: Collective goal (e.g., shared certification, bulk marketing, resource pooling) (required)
- `created_at`: When association was formed
- `member_count`: Denormalized count of current members (for performance)
- `is_active`: Whether association is active (default: true)

**Relationships**:
- **Members** (N-M): Many farmers belong to many associations (join table: association_members)
  - `farmer_id`: Reference to farmer
  - `association_id`: Reference to association
  - `joined_at`: When farmer joined the association
  - `role`: Member role within association (optional)

## Entity Relationship Summary

```
Farmer (1-N) → Farm (N-1) ← (N-M) FarmerAssociation
                     |
                     | (N-1) Certification
                     |
                   Farm
```

**Key Validation Rules** (from Functional Requirements):
- FR-001: Farm requires name, location, size (all mandatory)
- FR-005: Certification requires farm_id, certification_type, expiration_date, status
- FR-007: Certification with expired expiration_date must be flagged as invalid
- FR-008: Certification audit_trail must preserve complete decision history

**Constraints** (from Constitution):
- Farm and Certification belong to Traceability Domain
- Farmer authentication uses JWT + Argon2id per constitution
- Authorization uses RBAC (Admin, Producer, Customer, Support) enforced on backend
- Clean Architecture: domain layer independent of framework dependencies
- Outbox Pattern for all certification-related integration events