# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command; its definition describes the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

**Language/Version**: Rust 1.75+ (backend), TypeScript 5.x (frontend)

**Primary Dependencies**: axum, sqlx, tokio (Rust backend); React 18+, Vite, @radix-ui/shadcn/ui (frontend)

**Storage**: PostgreSQL (single database, separate domain schemas per BioSouk Constitution)

**Testing**: cargo test (Rust unit/integration), oxlint (Rust linting), vitest (frontend), Playwright (e2e)

**Target Platform**: Linux server (production), development containers, CI environments

**Project Type**: modular monolith (single repository, single backend application, single PostgreSQL database, separate domain schemas per constitution)

**Performance Goals**: Standard web application expectations for agricultural marketplace; inventory operations under 2s response; certification workflow under 5s end-to-end

**Constraints**: Clean Architecture boundaries (domain layer must not depend on Axum, SQLx, PostgreSQL, React, or external implementations); Modular monolith structure (no premature microservice decomposition per constitution); Constitution compliance (Traceability Domain ownership, JWT/Argon2id auth, RBAC authorization); Synthetic data required for dev/testing; No production secrets in source control; Audit logging mandatory for all certification decisions

**Scale/Scope**: Agricultural marketplace supporting farmers, products, certifications, and orders; moderate scale with expected growth; domain-driven design with 8 bounded contexts (Platform, Customer, Commerce, Order, Traceability, Operations, Engagement, Analytics)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**GATE 1 - Domain Ownership**: Farm and Certification domains assigned to Traceability Domain per constitution. ✓ PASS

**GATE 2 - Clean Architecture**: Domain layer contains business rules; application layer orchestrates use cases; infrastructure layer implements technical adapters. ✓ PASS

**GATE 3 - RBAC Authorization**: RBAC model enforced on backend with Admin, Producer, Customer, Support roles as per constitution. ✓ PASS

**GATE 4 - Authentication**: JWT-based approach with Argon2id password hashing per constitution. ✓ PASS

**GATE 5 - Outbox Pattern**: All integration events must use Outbox Pattern with minimum fields (event_id, event_type, event_version, aggregate_id, payload, created_at, processed_at, processing_status, retry_count). ✓ PASS

**GATE 6 - Security Requirements**: Input validation, secure password storage, secrets management, API rate limiting, audit logging per constitution. ✓ PASS

**GATE 7 - No Production Secrets**: No hardcoded credentials or tokens in source control. ✓ PASS

**GATE 8 - Audit Trail**: Certification decisions must preserve complete audit trail from submission to final decision. ✓ PASS

**GATE 9 - Modular Monolith**: Single repository, single backend application, single PostgreSQL database, separate domain schemas. ✓ PASS

**GATE 10 - Event-Driven Design**: Domain events for important business activities (CertificationUploaded, CertificationRevoked, OrderCreated, OrderPaid, etc.). ✓ PASS

**Status**: All gates pass. No violations detected.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
