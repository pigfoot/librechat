# Research: ResponsesAPI Default Enablement

**Date**: 2025-10-22
**Feature**: ResponsesAPI Default Enablement
**Research Focus**: Understanding current implementation and identifying modification points

## Key Findings

### 1. Current ResponsesAPI Implementation

**Decision**: ResponsesAPI is currently implemented as an optional feature flag
**Rationale**: Found in multiple configuration files and client implementations
**Key Files**:
- `/api/app/clients/OpenAIClient.js` - Main OpenAI client with ResponsesAPI support
- `/packages/api/src/endpoints/openai/config.ts` - Configuration handling
- `/packages/api/src/endpoints/openai/llm.ts` - LLM integration with ResponsesAPI
- `/packages/data-schemas/src/schema/defaults.ts` - Default values

### 2. Default Value Handling

**Decision**: Defaults are set in data-schemas package
**Rationale**: Centralized schema definitions ensure consistency
**Current Behavior**:
- ResponsesAPI is NOT enabled by default in backend logic
- UI may show it as enabled but backend doesn't process it correctly
- This mismatch is the core issue to fix

### 3. Endpoint-Specific Handling

**OpenAI Endpoint**:
- Handled in `/api/app/clients/OpenAIClient.js`
- Uses `responsesAPI` flag in conversation parameters

**Azure Endpoint**:
- Currently requires API version parameter
- When ResponsesAPI is enabled, API version should be omitted per Microsoft guidance
- Configuration in same OpenAI client (Azure uses OpenAI SDK)

**Custom Endpoints**:
- Inherit from OpenAI client behavior
- Support ResponsesAPI through the same mechanism

### 4. UI-Backend Synchronization

**Decision**: State management uses React hooks and stores
**Key Components**:
- `/client/src/hooks/Conversations/useSetIndexOptions.ts` - Conversation options
- Client-side state management for endpoint settings
- Backend receives settings through API calls

**Current Issue**:
- UI default doesn't match backend processing
- Need to ensure both initialize with same default value

### 5. Azure API Version Handling

**Decision**: Conditionally omit API version when ResponsesAPI is enabled
**Rationale**: Microsoft Azure AI Foundry documentation specifies this behavior
**Implementation Point**:
- Modify Azure endpoint configuration logic
- Check responsesAPI flag before adding API version parameter

## Implementation Strategy

### Modification Points

1. **Backend Default Values**:
   - `/packages/data-schemas/src/schema/defaults.ts`
   - Set `responsesAPI: true` as default for OpenAI-compatible endpoints

2. **Frontend Default Sync**:
   - Ensure UI components use same default from data-schemas
   - Update conversation initialization hooks

3. **Azure API Version Logic**:
   - Modify OpenAI client to conditionally include API version
   - Add logic: `if (isAzure && responsesAPI) { omitApiVersion() }`

4. **Existing Conversation Handling**:
   - Keep current ResponsesAPI state for existing conversations
   - Only apply new default to new conversations

### Testing Requirements

1. New conversation initialization with ResponsesAPI enabled
2. UI toggle synchronization with backend
3. Azure endpoint without API version when ResponsesAPI enabled
4. Existing conversation preservation

## Alternatives Considered

1. **Server-side override**: Force ResponsesAPI regardless of UI
   - Rejected: Would break user control and transparency

2. **Migration script**: Update all existing conversations
   - Rejected: Against clarified requirements, preserve existing state

3. **Separate Azure client**: Create dedicated Azure handling
   - Rejected: Violates Brownfield-first principle, unnecessary complexity

## Dependencies

- OpenAI SDK (already in use)
- MongoDB for conversation persistence
- React state management (existing)

## Risks & Mitigations

1. **Risk**: Breaking existing conversations
   - **Mitigation**: Explicit check for existing vs new conversations

2. **Risk**: Custom endpoints incompatibility
   - **Mitigation**: Per clarification, force enable and accept errors

3. **Risk**: Azure API failures
   - **Mitigation**: Thoroughly test Azure endpoint behavior

## Conclusion

The implementation requires minimal changes to existing code:
1. Update default values in schema
2. Ensure UI uses same defaults
3. Add conditional Azure API version logic
4. Preserve existing conversation states

All modifications follow Brownfield-first principle by changing existing behavior rather than creating new systems.