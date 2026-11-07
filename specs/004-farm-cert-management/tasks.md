# Tasks: Farm and Certification Management

**Input**: Design documents from `/home/dali/WORK/bio-souk/specs/004-farm-cert-management/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tasks include test tasks where appropriate; tests are OPTIONAL but recommended for independent story verification

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story. Each user story can be implemented, tested, and delivered independently as an MVP increment.

**Path Conventions**: Single project with modular monolith structure - `src/`, `tests/` at repository root. Backend: Rust with axum/sqlx. Frontend: TypeScript with React/Vite.

---

## Format: `- [X] T### [P?] [Story] Description with file path`

**Format Components**:
1. **Checkbox**: ALWAYS start with `- [ ]` (markdown checkbox)
2. **Task ID**: Sequential number (T001, T002, T003...) in execution order
3. **[P] marker**: Include ONLY if task is parallelizable (different files, no dependencies on incomplete tasks)
4. **[Story] label**: REQUIRED for user story phase tasks only
   - Format: [US1], [US2], [US3], etc. (maps to user stories from spec.md)
   - Setup phase: NO story label
   - Foundational phase: NO story label
   - User Story phases: MUST have story label
   - Polish phase: NO story label
5. **Description**: Clear action with exact file path

**Examples**:
- ✅ CORRECT: `- [X] T001 Create project structure per implementation plan`
- ✅ CORRECT: `- [X] T005 [P] Implement authentication middleware in src/middleware/auth.py`
- ✅ CORRECT: `- [X] T012 [P] [US1] Create User model in src/models/user.py`
- ✅ CORRECT: `- [X] T014 [US1] Implement UserService in src/services/user_service.py`
- ❌ WRONG: `- [ ] Create User model` (missing ID and Story label)
- ❌ WRONG: `T001 [US1] Create model` (missing checkbox)
- ❌ WRONG: `- [ ] [US1] Create User model` (missing Task ID)
- ❌ WRONG: `- [X] T001 [US1] Create model` (missing file path)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [X] T001 Create project structure per implementation plan
- [X] T002 Initialize Rust project with axum, sqlx, tokio dependencies
- [X] T003 [P] Configure linting and formatting tools (oxlint, rustfmt)
- [X] T004 Setup PostgreSQL database and domain schemas per BioSouk Constitution (platform, commerce, order, traceability schemas)
- [X] T005 Configure environment variables and .env.example (DATABASE_URL, JWT_SECRET, ARGON2ID_SALT)
- [X] T006 Initialize git repository with proper .gitignore and Conventional Commits setup
- [X] T007 [P] Create Cargo.toml with modular monolith workspace structure
- [X] T008 [P] Setup CI pipeline (fmt, clippy, build, test jobs)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T009 [P] Setup database connection pool with sqlx (Rust backend)
- [X] T010 [P] Implement JWT authentication middleware (Argon2id password hashing per constitution)
- [X] T011 [P] Implement RBAC authorization middleware (Admin, Producer, Customer, Support roles per constitution)
- [X] T012 [P] Setup Outbox Pattern infrastructure for domain events (minimum fields: event_id, event_type, event_version, aggregate_id, payload, created_at, processed_at, processing_status, retry_count)
- [X] T013 [P] Configure audit logging for all certification decisions (mandatory per constitution)
- [X] T014 [P] Setup Clean Architecture layer structure (domain, application, infrastructure)
- [X] T015 [P] Create shared error handling infrastructure (custom errors, error mapping)
- [X] T016 [P] Setup structured logging (tracing/subscriber) with correlation IDs

---

## Phase 3: User Story 1 - Farm Registration (Priority: P1) 🎯 MVP

**Goal**: A farmer can register a new farm through the system.

**Independent Test**: Can be fully tested by attempting to register a new farm with valid farmer credentials and verifying the farm is created and associated with the farmer.

- [X] T017 [P] [US1] Create Farm model in src/models/farm.rs (fields: id, name, description, location, size, status, owner_id, created_at, updated_at)
- [X] T018 [P] [US1] Implement farm registration handler in src/handlers/farm_handler.rs (POST /api/v1/farms)
- [X] T019 [P] [US1] Create farm validation schema with axum (name: required string max 200 chars, location: required string, size: required decimal-safe number)
- [X] T020 [US1] Write integration test for farm registration in tests/integration/farm_test.rs (given authenticated farmer, when valid details provided, then farm registered successfully)
- [X] T021 [US1] Add farm registration acceptance criteria documentation in spec.md (SC-001: Farmers can complete farm registration successfully with all required fields validated)
- [X] T022 [US1] Verify farm appears in GET /api/v1/farms listing after creation

---

## Phase 4: User Story 2 - Farm Profile Management (Priority: P2)

**Goal**: A farmer can manage their farm profile details after registration.

**Independent Test**: Can be fully tested by updating farm profile fields and verifying the changes are persisted and reflect in related operations.

- [X] T023 [P] [US2] Extend Farm model with update methods (name, description, location fields)
- [X] T024 [P] [US2] Implement farm profile update handler in src/handlers/farm_handler.rs (PATCH /api/v1/farms/{id})
- [X] T025 [P] [US2] Implement farm profile ownership validation (farmer can only update their own farm)
- [X] T026 [P] [US2] Create farm profile update validation schema (optional name/description/location fields)
- [X] T027 [US2] Write integration test for farm profile management in tests/integration/farm_profile_test.rs (given authenticated farmer owns farm, when profile updated, then changes reflected across system)
- [X] T028 [US2] Add farm profile management acceptance criteria documentation in spec.md (SC-002: Farm profile management operations complete within expected time and reflect changes across the system)
- [X] T029 [US2] Verify farm profile changes appear in dashboard and related operations

---

## Phase 5: User Story 3 - Farmer Association (Priority: P2)

**Goal**: A farmer can associate with farmer associations or groups.

**Independent Test**: Can be fully tested by associating a farmer with an association and verifying the association is linked to the farmer's profile.

- [X] T030 [P] [US3] Create FarmerAssociation model in src/models/farmer_association.rs (fields: id, name, description, purpose, created_at, member_count, is_active)
- [X] T031 [P] [US3] Implement association join handler in src/handlers/farmer_handler.rs (POST /api/v1/farmer-associations)
- [X] T032 [P] [US3] Implement association leave handler in src/handlers/farmer_handler.rs (DELETE /api/v1/farmer-associations/{id}/members/{farmer_id})
- [X] T033 [P] [US3] Implement association membership validation (farmer can only join active associations; cannot join own association twice)
- [X] T034 [US3] Write integration test for farmer associations in tests/integration/farmer_association_test.rs (given authenticated farmer, when associates with group, then linked to profile and visible in dashboard)
- [X] T035 [US3] Add farmer association acceptance criteria documentation in spec.md
- [X] T036 [US3] Verify association membership visible in farmer dashboard and affects certification bulk processes

---

## Phase 6: User Story 4 - Farm Location Management (Priority: P3)

**Goal**: A farmer can manage farm locations (add/remove) for their farms.

**Independent Test**: Can be fully tested by adding and removing farm locations and verifying they are correctly associated with the farm.

- [X] T037 [P] [US4] Add location management fields to Farm model (relationship to location entities)
- [X] T038 [P] [US4] Implement farm location add handler in src/handlers/farm_handler.rs
- [X] T039 [P] [US4] Implement farm location remove handler in src/handlers/farm_handler.rs
- [X] T040 [P] [US4] Implement location validation (geographic format, associated with valid farm)
- [X] T041 [US4] Write integration test for farm location management in tests/integration/farm_location_test.rs (given authenticated farmer owns farm, when adds/removes location, then correctly associated)
- [X] T042 [US4] Add farm location management acceptance criteria documentation in spec.md
- [X] T043 [US4] Verify locations correctly associated and affect logistics/inventory operations

---

## Phase 7: User Story 5 - Certification Submission (Priority: P1)

**Goal**: A farmer can submit a certification for their farm.

**Independent Test**: Can be fully tested by submitting a certification for a farm and verifying it enters the review queue.

- [X] T044 [P] [US5] Create Certification model in src/models/certification.rs (fields: id, farm_id, certification_type, status, expiration_date, submission_date, review_date, decision_date, reviewer_id, feedback, audit_trail, created_at, updated_at)
- [X] T045 [P] [US5] Implement certification submission handler in src/handlers/certification_handler.rs (POST /api/v1/certifications)
- [X] T046 [P] [US5] Implement certification validation (farm ownership verification, certification_type validation, expiration_date must be future date)
- [X] T047 [P] [US5] Initialize certification audit_trail on submission (add submission event to audit trail)
- [X] T048 [US5] Write integration test for certification submission in tests/integration/certification_test.rs (given authenticated farmer owns farm, when submits certification, then enters review queue with pending status)
- [X] T049 [US5] Add certification submission acceptance criteria documentation in spec.md (SC-003: Certifications can be submitted, reviewed, and reached a final state)
- [X] T050 [US5] Verify certification created with correct audit_trail entry and visible in farm's certifications list

---

## Phase 8: User Story 6 - Certification Review (Priority: P1)

**Goal**: An administrator can review and evaluate certifications.

**Independent Test**: Can be fully tested by an administrator reviewing a certification and marking it as approved or rejected.

- [X] T051 [P] [US6] Implement certification review handler in src/handlers/certification_handler.rs (PATCH /api/v1/certifications/{id}/review)
- [X] T052 [P] [US6] Implement decision logic (approved/rejected) with feedback storage
- [X] T053 [P] [US6] Update certification status and decision_date on review completion
- [X] T054 [P] [US6] Update audit_trail with review event (include decision, reviewer_id, timestamp)
- [X] T055 [US6] Write integration test for certification review in tests/integration/certification_review_test.rs (given authenticated administrator, when reviews certification, then marks approved/rejected with feedback)
- [X] T056 [US6] Add certification review acceptance criteria documentation in spec.md (SC-004: Expired certifications properly flagged; SC-005: Certification decisions auditable with complete trail)
- [X] T057 [US6] Verify farmer notified of certification decision and status updated system-wide

---

## Phase 9: User Story 7 - Certification Expiration Management (Priority: P2)

**Goal**: The system manages certification expiration and prevents use of expired certifications.

**Independent Test**: Can be fully tested by submitting a certification, verifying it's valid, then simulating expiration and verifying it's flagged as invalid.

- [X] T058 [P] [US7] Implement certification expiration check logic (compare expiration_date against current date)
- [X] T059 [P] [US7] Add expired certification flagging (status: expired when current_date > expiration_date)
- [X] T060 [P] [US7] Implement exclusion of expired certifications from valid certification results queries
- [X] T061 [US7] Write integration test for expiration management in tests/integration/certification_expiry_test.rs (given certification with expiration date, when current date exceeds expiration, then flagged as expired and excluded from valid results)
- [X] T062 [US7] Add certification expiration acceptance criteria documentation in spec.md (SC-004: Expired certifications properly flagged and excluded from valid certification results)
- [X] T063 [US7] Verify system correctly handles edge case: certification expired on exact date boundary

---

## Phase 10: User Story 8 - Administrative Approval Workflow (Priority: P1)

**Goal**: An administrative approval workflow manages certification decisions end-to-end.

**Independent Test**: Can be fully tested by submitting a certification and tracing it through the full approval workflow from submission to final decision.

- [X] T064 [P] [US8] Implement approval workflow orchestrator in src/workflows/certification_workflow.rs
- [X] T065 [P] [US8] Implement workflow state transitions (submitted → under_review → approved/rejected → expired)
- [X] T066 [P] [US8] Implement notifications for decision changes (emit domain events via Outbox Pattern)
- [X] T067 [US8] Write integration test for approval workflow in tests/integration/workflow_test.rs (given certification submitted, when follows workflow, then reaches final state with complete audit trail from submission to final decision)
- [X] T068 [US8] Add administrative approval workflow acceptance criteria documentation in spec.md
- [X] T069 [US8] Verify complete end-to-end workflow: submission → administrator review → decision → farmer notification → system state consistency

---

## Phase 11: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories; final validation and quality gates

- [X] T070 [P] Documentation updates in specs/ and src/ (update plan.md, spec.md with actual task references)
- [X] T071 [P] Code cleanup and refactoring across all user story implementations
- [X] T072 [P] Performance optimization across all stories (inventory operations under 2s; certification workflow under 5s end-to-end)
- [X] T073 [P] Security hardening (RBAC enforcement verification, JWT token validation, Argon2id password hashing compliance)
- [X] T074 [P] Run quickstart.md validation (all 6 validation scenarios pass)
- [X] T075 [P] Additional unit tests if requested (tests/unit/ for all models and services)
- [X] T076 [P] Generate final API specification from implemented endpoints
- [X] T077 [P] Constitution compliance re-check (all 10 gates pass post-design)
- [X] T078 [P] Deployment configuration (Docker production images, health checks, rate limiting)
- [X] T079 [P] Backup and recovery procedure validation for certification and farm data

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-10)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Phase 11)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1) [US1]**: Can start after Foundational (Phase 2) - No dependencies on other stories (foundation for all subsequent stories)
- **User Story 2 (P2) [US2]**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P2) [US3]**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable
- **User Story 4 (P3) [US4]**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 5 (P1) [US5]**: Can start after Foundational (Phase 2) - Depends on US1 (farm must exist before certification)
- **User Story 6 (P1) [US6]**: Can start after Foundational (Phase 2) - Depends on US5 (certification must exist before review)
- **User Story 7 (P2) [US7]**: Can start after Foundational (Phase 2) - Depends on US5/US6 (expiration applies to existing certifications)
- **User Story 8 (P1) [US8]**: Can start after Foundational (Phase 2) - Depends on US5/US6/US7 (workflow applies to certified, non-expired certifications)

### Within Each User Story

- Tests (if included) MUST be written and FAIL before implementation
- Models before services
- Services before endpoints
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
  - US1, US2, US3, US4 can proceed in parallel (independent, no shared model dependencies beyond foundational)
  - US5 depends on US1 completion for model/entities
  - US6 depends on US5 completion
  - US7 depends on US5/US6 for existing certification handling
  - US8 depends on US5/US6/US7 for full workflow
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

### Parallel Example: Foundational Phase

```bash
# Launch all foundational tasks in parallel (all marked [P]):
Task: "Setup database connection pool with sqlx (Rust backend)"
Task: "Implement JWT authentication middleware (Argon2id password hashing per constitution)"
Task: "Implement RBAC authorization middleware (Admin, Producer, Customer, Support roles per constitution)"
Task: "Setup Outbox Pattern infrastructure for domain events"
Task: "Configure audit logging for all certification decisions"
Task: "Setup Clean Architecture layer structure"
Task: "Create shared error handling infrastructure"
Task: "Setup structured logging with correlation IDs"
```

### Parallel Example: User Story 1

```bash
# Launch all [P] marked tasks for User Story 1 in parallel:
Task: "Create Farm model in src/models/farm.rs"
Task: "Implement farm registration handler in src/handlers/farm_handler.rs"
Task: "Create farm validation schema with axum"

# Launch tests in parallel:
Task: "Write integration test for farm registration in tests/integration/farm_test.rs"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (project initialization)
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Farm Registration)
4. **STOP and VALIDATE**: Test User Story 1 independently - all acceptance criteria pass
5. Deploy/demo if ready (MVP delivered: farmers can register farms)
6. Proceed to subsequent stories incrementally

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready (all infrastructure complete)
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo (farm management added)
4. Add User Story 3 → Test independently → Deploy/Demo (farmer associations added)
5. Add User Story 4 → Test independently → Deploy/Demo (farm locations added)
6. Add User Story 5 → Test independently → Deploy/Demo (certification submission added)
7. Add User Story 6 → Test independently → Deploy/Demo (certification review added)
8. Add User Story 7 → Test independently → Deploy/Demo (expiration management added)
9. Add User Story 8 → Test independently → Deploy/Demo (full approval workflow added)
10. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (all [P] marked tasks in Phase 1-2)
2. Once Foundational is done:
   - Developer A: User Story 1 (Farm Registration) [US1]
   - Developer B: User Story 2 (Farm Profile Management) [US2]
   - Developer C: User Story 3 (Farmer Association) [US3]
   - Developer D: User Story 4 (Farm Location Management) [US4]
   - Developer E: User Story 5 (Certification Submission) [US5] -- depends on US1
   - Developer F: User Story 6 (Certification Review) [US6] -- depends on US5
   - Developer G: User Story 7 (Expiration Management) [US7] -- depends on US5/US6
   - Developer H: User Story 8 (Approval Workflow) [US8] -- depends on US5/US6/US7
3. Stories complete and integrate independently at each checkpoint
4. Final phase: Polish & cross-cutting concerns (all stories complete)

---

## Notes

- [P] tasks = different files, no dependencies on each other (can run in parallel)
- [Story] label maps task to specific user story for traceability to spec.md
- Each user story should be independently completable and testable
- Verify tests fail before implementing (TDD approach recommended)
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- **Avoid**: vague tasks, same file conflicts, cross-story dependencies that break independence
- **Key**: US5 depends on US1 (certification requires existing farm), US6 depends on US5 (review requires existing certification), US7 depends on US5/US6 (expiration applies to certified certifications), US8 depends on US5/US6/US7 (full workflow covers all certification states)
- **MVP Scope**: Just User Story 1 (Farm Registration) - farmers can register farms. This is the foundational entry point; all other stories build upon it.