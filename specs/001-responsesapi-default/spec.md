# Feature Specification: ResponsesAPI Default Enablement

**Feature Branch**: `001-responsesapi-default`
**Created**: 2025-10-22
**Status**: Draft
**Input**: User description: "Using ResponsesAPI as true by default for OpenAI compatible endpoints"

## Clarifications

### Session 2025-10-22

- Q: How should existing conversations handle ResponsesAPI settings? → A: Show current state without migration
- Q: What happens to ResponsesAPI setting when switching endpoints in same conversation? → A: Maintain current settings
- Q: How to handle custom endpoints that don't support ResponsesAPI? → A: Force enable regardless

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Default ResponsesAPI for New Conversations (Priority: P1)

Users starting new conversations with OpenAI-compatible endpoints (OpenAI, Azure, Custom) should automatically use the ResponsesAPI format without manual configuration, ensuring consistent behavior between UI display and backend processing.

**Why this priority**: This addresses the core issue where UI shows ResponsesAPI as enabled but the backend may not process it correctly, causing confusion and inconsistent behavior for users.

**Independent Test**: Can be fully tested by creating a new conversation with any OpenAI-compatible endpoint and verifying that ResponsesAPI is active both in UI and backend without user intervention.

**Acceptance Scenarios**:

1. **Given** a user starts a new conversation with OpenAI endpoint, **When** they send their first message, **Then** ResponsesAPI is automatically enabled in both UI and backend processing
2. **Given** a user starts a new conversation with Azure endpoint, **When** they send their first message, **Then** ResponsesAPI is automatically enabled and API version parameter is omitted
3. **Given** a user starts a new conversation with a Custom endpoint, **When** they send their first message, **Then** ResponsesAPI is automatically enabled

---

### User Story 2 - UI and Backend Synchronization (Priority: P1)

When users toggle the ResponsesAPI setting in the UI, the backend processing must immediately reflect this change, maintaining consistency throughout the session.

**Why this priority**: UI-backend synchronization is critical for user trust and system reliability. Mismatched states can lead to unexpected behaviors and poor user experience.

**Independent Test**: Can be tested by toggling the ResponsesAPI setting in UI and verifying that subsequent messages use the correct API format in backend processing.

**Acceptance Scenarios**:

1. **Given** ResponsesAPI is enabled in UI, **When** user sends a message, **Then** backend processes the request using ResponsesAPI format
2. **Given** user disables ResponsesAPI in UI, **When** they send a message, **Then** backend processes the request using standard API format
3. **Given** user re-enables ResponsesAPI after disabling, **When** they send a message, **Then** backend immediately switches back to ResponsesAPI format

---

### User Story 3 - Azure Endpoint API Version Handling (Priority: P2)

Users working with Azure endpoints should have simplified configuration when ResponsesAPI is enabled, with automatic handling of API version requirements.

**Why this priority**: This improves user experience for Azure users by removing unnecessary configuration complexity while maintaining compatibility with Azure AI Foundry specifications.

**Independent Test**: Can be tested by configuring an Azure endpoint with ResponsesAPI enabled and verifying that API calls succeed without explicit API version parameters.

**Acceptance Scenarios**:

1. **Given** an Azure endpoint with ResponsesAPI enabled by default, **When** user sends a message, **Then** the request is sent without API version parameter and succeeds
2. **Given** an Azure endpoint where user manually enables ResponsesAPI, **When** they send a message, **Then** the API version parameter is automatically removed from the request

---

### Edge Cases

- What happens when switching between different endpoint types (OpenAI to Azure to Custom) within the same conversation? → ResponsesAPI setting is maintained when switching endpoints
- How does the system handle existing conversations created before this default change? → Existing conversations display their current ResponsesAPI state without automatic migration
- What occurs if a custom endpoint explicitly requires non-ResponsesAPI format? → ResponsesAPI is forced regardless of endpoint compatibility (errors are expected and acceptable)

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST enable ResponsesAPI by default for all new conversations using OpenAI-compatible endpoints (OpenAI, Azure, Custom)
- **FR-002**: System MUST synchronize ResponsesAPI state between UI display and backend processing logic at all times
- **FR-003**: UI MUST show ResponsesAPI as enabled by default for OpenAI-compatible endpoints
- **FR-004**: Backend MUST process requests using ResponsesAPI format when UI shows it as enabled
- **FR-005**: System MUST automatically omit API version parameter for Azure endpoints when ResponsesAPI is enabled
- **FR-006**: Users MUST be able to manually toggle ResponsesAPI on/off with immediate effect
- **FR-007**: System MUST maintain ResponsesAPI preference throughout a conversation session, including when switching between different endpoint types
- **FR-008**: System MUST apply ResponsesAPI default to all supported model types without checking individual model compatibility
- **FR-009**: System MUST continue using ResponsesAPI format even if custom endpoints fail due to incompatibility

### Key Entities *(include if feature involves data)*

- **Endpoint Configuration**: Represents the settings for each API endpoint type, including ResponsesAPI enablement status
- **Conversation Settings**: Stores the active ResponsesAPI state for each conversation session
- **API Request**: The formatted request sent to endpoints, which varies based on ResponsesAPI enablement

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of new conversations with OpenAI-compatible endpoints have ResponsesAPI enabled by default
- **SC-002**: UI and backend ResponsesAPI states match in 100% of tested scenarios
- **SC-003**: Azure endpoint requests with ResponsesAPI enabled succeed without API version parameter in 100% of cases
- **SC-004**: ResponsesAPI toggle changes take effect within 100ms of user action
- **SC-005**: Zero reported issues of UI-backend state mismatch after implementation

## Assumptions

- ResponsesAPI format is compatible with all models on OpenAI, Azure, and Custom endpoints
- Users prefer ResponsesAPI as the default for better feature compatibility
- Existing conversations will continue using their current settings (no retroactive changes)
- Azure AI Foundry's guidance on API versioning with ResponsesAPI is accurate and stable