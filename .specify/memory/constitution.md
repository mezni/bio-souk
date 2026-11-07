<!-- Sync Impact Report:
- Version change: 1.1 → 1.2.0 (MINOR: new principles/sections added via template structure)
- Added sections: Domain Ownership & Traceability, Security, Events & Integrity
- Modified principles: 5 Core Principles restructured from existing Article II content
- Deferred: RATIFICATION_DATE TODO(original adoption date unknown)
-->

# BioSouk Constitution
<!-- Example: Spec Constitution, TaskFlow Constitution, etc. -->

## Core Principles

### Business First
Business capabilities drive architecture.

Correct:

- Product
- Producer
- Farm
- Batch
- Certification
- Inventory
- Order
- Shipment

Incorrect:

- Controller
- DTO
- Database Table

The domain model is the source of truth.

### Bounded Contexts
All functionality must belong to an explicitly defined bounded context.

Approved domains:

1. Platform Domain
2. Customer Domain
3. Commerce Domain
4. Order Domain
5. Traceability Domain
6. Operations Domain
7. Engagement Domain
8. Analytics Domain

No feature may be implemented without assigning domain ownership.

### Explicit Ownership
Each domain owns:

- Business rules.
- Aggregates.
- Domain entities and value objects.
- Persistence schema.
- Domain events.
- Application services.

A domain must not modify another domain's aggregates directly.

### Clean Architecture
BioSouk follows Clean Architecture.

Dependencies must point inward:

- Domain layer contains business rules.
- Application layer orchestrates use cases.
- Infrastructure layer implements technical adapters.
- Presentation layer exposes APIs and user interfaces.

The domain layer must not depend on Axum, SQLx, PostgreSQL, React, or external service implementations.

### Modular Monolith
BioSouk is implemented as a Modular Monolith.

Current architecture:

- Single repository.
- Single backend application.
- Single PostgreSQL database.
- Separate domain schemas.
- Independently structured frontend applications.

Modules communicate through:

- Public application interfaces.
- Domain events.
- Explicit integration contracts.

Direct access to another domain's private implementation or database repositories is forbidden.

Future microservices must remain possible without requiring a fundamental domain redesign.

## Domain Ownership & Traceability

### Platform Domain

Owns:

- User.
- Role.
- Permission.
- Authentication.
- Authorization.
- Audit.

Responsible for:

- JWT.
- RBAC.
- Password security.
- Access control.
- Session management.

### Customer Domain

Owns:

- Customer.
- Address.
- Preferences.
- Consent.
- Loyalty.

### Commerce Domain

Owns:

- Product.
- Category.
- Brand.
- Price.
- Promotion.
- Inventory.

Products support:

- Unit pricing.
- Weight pricing.
- Volume pricing.

### Order Domain
BioSouk's Core Domain.

Owns:

- Cart.
- Order.
- OrderItem.
- Payment.
- Shipment.
- Return.

Responsible for:

- Checkout.
- Purchasing commitment.
- Order lifecycle.
- Payment orchestration.
- Refund orchestration.

### Traceability Domain
Mandatory strategic domain.

Owns:

- Producer.
- Farm.
- Origin.
- Batch.
- Lot.
- Certification.
- Recall.
- QualityInspection.

Responsible for product provenance and traceability integrity.

### Operations Domain

Owns:

- Warehouse.
- Supplier.
- Procurement.
- Fulfillment.
- ColdChain.

Supports:

- Perishable products.
- Temperature constraints.
- Batch tracking.
- Inventory allocation.

### Engagement Domain

Owns:

- Review.
- Campaign.
- Recommendation.
- Interaction.
- SemanticSearch.

### Analytics Domain

Owns:

- Customer360.
- RevenueAnalytics.
- ProductAffinity.
- ProducerAnalytics.

Analytics is read-only with respect to transactional domains.

It must not modify business aggregates.

## Security, Events & Integrity

### Security Principles
Security is mandatory across all applications and domains.

#### Authentication
Approved technologies:

- JWT.
- Argon2id password hashing.
- Refresh tokens.
- Secure token rotation and revocation.

#### Authorization
Authorization model:

RBAC.

Roles:

- Admin.
- Producer.
- Customer.
- Support.

Authorization must be enforced on the backend.

Frontend permissions are not security boundaries.

#### Security Requirements

- Input validation.
- Secure password storage.
- Secrets management.
- API rate limiting.
- Audit logging.
- Secure file uploads.
- Dependency vulnerability scanning.
- Protection against common web application vulnerabilities.
- Secure refresh-token handling.
- Appropriate data access restrictions.

#### Forbidden

- Plaintext passwords.
- Hardcoded credentials.
- Hardcoded tokens.
- Production secrets in source control.
- Trusting frontend authorization.
- Unauthorized cross-domain data access.

### Event-Driven Design
BioSouk uses domain events for important business activities.

An internal event bus is required.

Required components:

- Domain Event.
- Event Dispatcher.
- Event Handlers.
- Outbox Pattern.

Example Events:

- CustomerCreated.
- ProductCreated.
- BatchCreated.
- CertificationUploaded.
- CertificationRevoked.
- OrderCreated.
- OrderPaid.
- ShipmentCreated.
- RecallInitiated.
- InventoryReserved.

Event Rules:

- Events represent completed business facts.
- Events are published after successful business transactions.
- Events must have stable identifiers.
- Events must support schema versioning.
- Events must include correlation identifiers.
- Consumers must be idempotent.
- Event handlers must not assume exactly-once delivery.

### Outbox Pattern
All integration events must use the Outbox Pattern.

Business data and corresponding outbox records must be persisted inside the same database transaction.

#### Minimum Outbox Fields

- event_id.
- event_type.
- event_version.
- aggregate_id.
- payload.
- created_at.
- processed_at.
- processing_status.
- retry_count.

#### Processing Requirements

- At-least-once delivery.
- Idempotent processing.
- Retry with backoff.
- Failed-event monitoring.
- Error recording.
- Correlation tracking.

The system must prevent duplicate business effects when an event is delivered more than once.

## Governance

Constitution supersedes all other practices.

Amendments require documentation, approval, and migration plan.

All PRs/reviews must verify governance compliance.

Complexity must be justified with an ADR.

Use guidance files for runtime development guidance.

Versioning policy: Semantic versioning (MAJOR.MINOR.PATCH) applies to constitution changes.

- MAJOR: Backward incompatible governance/principle removals or redefinitions.
- MINOR: New principle/section added or materially expanded guidance.
- PATCH: Clarifications, wording, typo fixes, non-semantic refinements.

Compliance review expectations:

- Constitution must be referenced in all significant architecture decisions.
- Amendments require review and approval by domain owners.
- Version changes must be documented in changelog.
- Last amended date updates on each modification.

**Version**: 1.2.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date unknown - consider marking when constitution is first formally ratified | **Last Amended**: 2026-11-06