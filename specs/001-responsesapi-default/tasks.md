---
description: "Task list for ResponsesAPI Default Enablement implementation"
---

# Tasks: ResponsesAPI Default Enablement

**Input**: Design documents from `/specs/001-responsesapi-default/`
**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/

**Tests**: Tests are OPTIONAL - not explicitly requested in feature specification

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Web app structure**: `api/` for backend, `client/` for frontend
- **Shared packages**: `packages/` for cross-project code

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure verification

- [X] T001 Verify existing ResponsesAPI implementation in api/app/clients/OpenAIClient.js
- [X] T002 Check current default values in packages/data-schemas/src/schema/defaults.ts
- [X] T003 Locate UI endpoint components in client/src/components/Endpoints/
- [X] T004 Review conversation model structure in api/models/Conversation.js

## Phase 2: Foundational Changes (Blocking Prerequisites)

**Purpose**: Core changes that all user stories depend on

- [X] T005 Update default ResponsesAPI value to true in packages/data-schemas/src/schema/defaults.ts
- [X] T006 Ensure MongoDB conversation schema supports responsesAPI field in api/models/Conversation.js
- [X] T007 Add deployment date constant for existing conversation detection in api/config/constants.js

## Phase 3: User Story 1 - Default ResponsesAPI for New Conversations (P1)

**Story Goal**: New conversations automatically use ResponsesAPI format
**Independent Test**: Create new conversation, verify ResponsesAPI enabled in UI and backend

- [ ] T008 [US1] Modify conversation initialization to use ResponsesAPI default in api/server/services/Conversations/initializeConversation.js
- [ ] T009 [P] [US1] Update OpenAI endpoint handler to respect ResponsesAPI default in api/server/controllers/endpoints/openai/initializeClient.js
- [ ] T010 [P] [US1] Update Azure endpoint handler to respect ResponsesAPI default in api/server/controllers/endpoints/azure/initializeClient.js
- [ ] T011 [P] [US1] Update Custom endpoint handler to respect ResponsesAPI default in api/server/controllers/endpoints/custom/initializeClient.js
- [ ] T012 [US1] Ensure UI reads ResponsesAPI default from data-schemas in client/src/hooks/useNewConversation.ts
- [ ] T013 [P] [US1] Update endpoint settings UI to show ResponsesAPI enabled by default in client/src/components/Endpoints/Settings/ResponsesAPIToggle.tsx
- [ ] T014 [US1] Add logic to detect new vs existing conversations in api/server/services/Conversations/getConversation.js
- [ ] T015 [US1] Preserve existing conversation ResponsesAPI state when loading in api/server/services/Conversations/loadConversation.js

## Phase 4: User Story 2 - UI and Backend Synchronization (P1)

**Story Goal**: UI toggle immediately reflects in backend processing
**Independent Test**: Toggle ResponsesAPI in UI, verify next message uses correct format

- [ ] T016 [US2] Add ResponsesAPI state synchronization to conversation update handler in api/server/routes/conversations/update.js
- [ ] T017 [P] [US2] Ensure UI toggle sends update request immediately in client/src/components/Endpoints/Settings/ResponsesAPIToggle.tsx
- [ ] T018 [US2] Update backend message handler to read current ResponsesAPI state in api/server/controllers/chat/sendMessage.js
- [ ] T019 [P] [US2] Add ResponsesAPI flag to API request builder in api/app/clients/OpenAIClient.js
- [ ] T020 [US2] Ensure state persistence across page refreshes in client/src/store/conversations.ts
- [ ] T021 [P] [US2] Add optimistic UI update for toggle with rollback on error in client/src/hooks/useResponsesAPIToggle.ts
- [ ] T022 [US2] Validate toggle response time < 100ms in client/src/components/Endpoints/Settings/ResponsesAPIToggle.tsx

## Phase 5: User Story 3 - Azure Endpoint API Version Handling (P2)

**Story Goal**: Azure endpoints omit API version when ResponsesAPI enabled
**Independent Test**: Enable ResponsesAPI on Azure, verify no API version in requests

- [ ] T023 [US3] Add conditional Azure API version logic in api/app/clients/OpenAIClient.js
- [ ] T024 [P] [US3] Create Azure-specific request builder that checks ResponsesAPI in api/server/services/endpoints/azure/buildRequest.js
- [ ] T025 [US3] Update Azure configuration to handle missing API version in api/server/controllers/endpoints/azure/config.js
- [ ] T026 [P] [US3] Add Azure ResponsesAPI validation in api/server/validators/azureEndpoint.js
- [ ] T027 [US3] Ensure Azure UI shows API version field disabled when ResponsesAPI enabled in client/src/components/Endpoints/Azure/APIVersionField.tsx

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final integration and edge case handling

- [ ] T028 Add ResponsesAPI state preservation during endpoint switching in api/server/services/Conversations/switchEndpoint.js
- [ ] T029 [P] Add error handling for incompatible custom endpoints in api/app/clients/OpenAIClient.js
- [ ] T030 [P] Add ResponsesAPI status to conversation export/import in api/server/services/Conversations/export.js
- [ ] T031 Create ResponsesAPI migration bypass for existing conversations in api/server/services/Conversations/migration.js
- [ ] T032 [P] Add ResponsesAPI telemetry for monitoring adoption in api/server/services/telemetry/responsesAPI.js

## Dependencies & Execution Order

### Story Dependencies
```
Phase 1 (Setup) → Phase 2 (Foundational) → Phase 3-5 (User Stories - can run in parallel) → Phase 6 (Polish)

User Stories can be implemented independently:
- US1 (Default ResponsesAPI): T008-T015
- US2 (UI-Backend Sync): T016-T022
- US3 (Azure API Version): T023-T027
```

### Parallel Execution Examples

**Within Phase 3 (US1)**:
```bash
# Can run in parallel (different endpoints):
T009 (OpenAI handler) & T010 (Azure handler) & T011 (Custom handler)

# Can run in parallel (UI components):
T013 (UI toggle default) & T012 (Hook updates)
```

**Within Phase 4 (US2)**:
```bash
# Can run in parallel:
T017 (UI toggle) & T019 (API request) & T021 (Optimistic update)
```

**Within Phase 5 (US3)**:
```bash
# Can run in parallel:
T024 (Request builder) & T026 (Validator) & T027 (UI field)
```

## Implementation Strategy

### MVP Scope (Recommended)
Complete Phase 1-3 (User Story 1) first for immediate value:
- New conversations get ResponsesAPI by default
- Existing conversations unchanged
- Basic functionality working

### Incremental Delivery
1. **Sprint 1**: Phase 1-3 (Setup + US1) - Core default behavior
2. **Sprint 2**: Phase 4 (US2) - UI-Backend synchronization
3. **Sprint 3**: Phase 5 (US3) - Azure improvements
4. **Sprint 4**: Phase 6 - Polish and edge cases

### Rollback Strategy
If issues occur, revert T005 (default value change) to quickly restore original behavior while keeping other improvements.

## Task Summary
- **Total Tasks**: 32
- **Setup Tasks**: 4 (T001-T004)
- **Foundational Tasks**: 3 (T005-T007)
- **User Story 1 Tasks**: 8 (T008-T015)
- **User Story 2 Tasks**: 7 (T016-T022)
- **User Story 3 Tasks**: 5 (T023-T027)
- **Polish Tasks**: 5 (T028-T032)
- **Parallel Opportunities**: 13 tasks marked with [P]

## Success Metrics
- [ ] All new conversations have ResponsesAPI enabled by default
- [ ] UI and backend states always match
- [ ] Azure endpoints work without API version when ResponsesAPI enabled
- [ ] Toggle response time < 100ms
- [ ] Zero regression in existing conversations