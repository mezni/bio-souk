# Research: Farm and Certification Management Feature

## Dependency Research

### 1. axum Web Framework (Rust)
- **Decision**: Use axum as the primary web framework for the Rust backend
- **Rationale**: axum is the recommended framework in the codebase (Cargo.toml dependencies), provides router-based routing, middleware support, and integrates well with Tower services. It aligns with the Clean Architecture requirements by keeping business logic in the domain layer while axum handles HTTP concerns in the infrastructure layer.
- **Alternatives considered**:
  - Actix-web: High-performance alternative but steeper learning curve and more mutable state complexity
  - Rocket: Rapid development but less flexible for modular monolith patterns and requires nightly Rust features

### 2. sqlx PostgreSQL Integration
- **Decision**: Use sqlx for compile-checked PostgreSQL queries
- **Rationale**: sqlx is already a dependency in the Cargo.toml provides compile-time query checking, reducing runtime errors. Supports both synchronous and asynchronous query execution. Fits the Modular Monolith architecture with separate domain schemas.
- **Alternatives considered**:
  - rusqlite: SQLite-only, not suitable for production PostgreSQL requirements
  - diesel: ORM-style but adds significant abstraction and requires compile-time migrations; sqlx preferred for raw SQL control per Clean Architecture

### 3. JWT Authentication with Argon2id
- **Decision**: JWT-based authentication with Argon2id password hashing as per BioSouk Constitution
- **Rationale**: Constitution explicitly approves JWT and Argon2id. The platform domain owns authentication, and the constitution provides specific guidance on password security and token management. This ensures compliance and consistency across the codebase.
- **Alternatives considered**:
  - Bcrypt: Valid password hashing but constitution specifically mandates Argon2id
  - Plain password comparison: Forbidden by constitution security principles

### 4. RBAC Authorization Middleware
- **Decision**: RBAC model with Admin, Producer, Customer, Support roles as defined in constitution
- **Rationale**: Constitution explicitly defines RBAC role hierarchy and mandates backend enforcement. The Traceability Domain, Commerce Domain, and Order Domain all have specific ownership that requires role-based access control. Backend-only enforcement per constitution ("Frontend permissions are not security boundaries").
- **Alternatives considered**:
  - Attribute-based access control (ABAC): More flexible but overkill for current scope; constitution mandates RBAC
  - Simple role checks without formal RBAC: Would violate constitution governance requirements

### 5. Outbox Pattern for Domain Events
- **Decision**: Implement Outbox Pattern with minimum required fields for all integration events
- **Rationale**: Constitution mandates Outbox Pattern for all integration events. Prevents duplicate business effects from event delivery more than once. Ensures at-least-once delivery with idempotent processing. The Outbox Pattern integrates events with business data in the same database transaction.
- **Minimum Outbox Fields (constitution-mandated)**:
  - event_id: Stable event identifier
  - event_type: Type discriminator for routing
  - event_version: Schema versioning support
  - aggregate_id: Source aggregate identifier
  - payload: Event data
  - created_at: Transaction timestamp
  - processed_at: Processing completion timestamp
  - processing_status: State tracking (pending, processed, failed)
  - retry_count: Failure counter for backoff strategy
- **Alternatives considered**:
  - Direct event publishing without outbox: Risk of duplicate effects, harder to ensure at-least-once delivery
  - Event sourcing-only approach: Different pattern; Outbox is the constitution-mandated pattern alongside event sourcing capabilities

### 6. Clean Architecture in Rust Modular Monolith
- **Decision**: Clean Architecture with inward-pointing dependencies as constitution specifies
- **Rationale**: Constitution explicitly states "The domain layer must not depend on Axum, SQLx, PostgreSQL, React, or external service implementations." This requires careful layering: domain (business rules) → application (use cases) → infrastructure (technical adapters). The modular monolith structure with separate domain schemas enforces these boundaries at the database level.
- **Key boundary enforcement**:
  - Domain layer: No external crate dependencies beyond core types (uuid, serde, thiserror)
  - Application layer: Can use axum, sqlx, tokio as framework adapters
  - Infrastructure layer: Implements adapters for PostgreSQL, JWT, Redis, etc.
  - Presentation layer: Frontend (React) and API layer

### 7. Modular Monolith with Separate Domain Schemas
- **Decision**: Single repository, single backend application, single PostgreSQL database with separate domain schemas per constitution
- **Rationale**: Constitution defines 8 bounded contexts (Platform, Customer, Commerce, Order, Traceability, Operations, Engagement, Analytics) each owning their domain. The modular monolith pattern enables shared infrastructure while maintaining domain boundaries. Separate schemas prevent cross-domain data access without explicit integration contracts.
- **Structure**:
  - `platform` schema: Users, roles, permissions, authentication
  - `commerce` schema: Products, categories, brands, pricing
  - `order` schema: Carts, orders, order items, payments
  - `traceability` schema: Farms, certifications, batches, lots, origins
  - `operations` schema: Warehouses, suppliers, procurement, cold chain
  - `engagement` schema: Reviews, campaigns, recommendations
  - `customer` schema: Customers, addresses, preferences, loyalty
  - `analytics` schema: Read-only analytics, revenue, product affinity (must not modify transactional domains)

## Integration Research

### Constitution Compliance Validation
- All design decisions validated against BioSouk Constitution v1.2.0 (last amended 2026-11-06)
- Core Principles: Business First, Bounded Contexts, Explicit Ownership, Clean Architecture, Modular Monolith
- Security Principles: JWT/Argon2id auth, RBAC, input validation, secure password storage, audit logging
- Event-Driven Design: Domain events, event dispatcher, event handlers, Outbox Pattern
- Governance: Semantic versioning, amendment process, ADR complexity justification

### PostgreSQL Schema Design
- Separate schemas per bounded context as defined in constitution
- Schema-level isolation prevents unintended cross-domain data access
- Migration strategy: domain-specific migrations versioned per schema
- Referential integrity maintained within schemas; cross-schema relationships via explicit integration events

### Audit Logging Strategy
- Mandatory for all certification decisions (approve/reject)
- Must preserve complete trail from submission to final decision
- Include: event_id, event_type, aggregate_id, user action, timestamp, outcome, optional feedback
- Stored in traceability schema per constitution audit requirements
- Immutable storage pattern to prevent modification post-creation

### Synthetic Data Guidelines
- Required for development and testing per constitution
- Must respect domain constraints and referential integrity
- Seed-based reproducibility for dataset generation
- Referential integrity validation against domain rules
- Edge case dataset for boundary condition testing
- Production databases must not be targeted accidentally (schema isolation helps)