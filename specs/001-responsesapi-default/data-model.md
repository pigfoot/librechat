# Data Model: ResponsesAPI Default Enablement

**Date**: 2025-10-22
**Feature**: ResponsesAPI Default Enablement

## Entities

### 1. EndpointConfiguration

**Description**: Configuration for OpenAI-compatible endpoints
**Location**: Backend configuration and UI state

**Fields**:
- `endpoint`: string - Type of endpoint (openai, azure, custom)
- `responsesAPI`: boolean - Whether to use ResponsesAPI format (DEFAULT: true)
- `apiVersion`: string | null - Azure API version (omitted when responsesAPI is true)

**Validation Rules**:
- `responsesAPI` must be boolean
- When `endpoint === 'azure' && responsesAPI === true`, `apiVersion` should be null
- Default value for new configurations: `responsesAPI = true`

### 2. ConversationSettings

**Description**: Per-conversation endpoint settings
**Location**: MongoDB conversations collection

**Fields**:
- `conversationId`: string - Unique conversation identifier
- `endpoint`: string - Selected endpoint type
- `responsesAPI`: boolean - ResponsesAPI enablement state
- `createdAt`: Date - Conversation creation timestamp
- `isNewConversation`: boolean - Flag for new vs existing conversations

**Validation Rules**:
- Existing conversations (`createdAt < deploymentDate`) maintain current `responsesAPI` value
- New conversations (`isNewConversation === true`) initialize with `responsesAPI = true`
- `responsesAPI` state persists across endpoint switches within same conversation

**State Transitions**:
- NEW → CONFIGURED: When first message sent, lock in ResponsesAPI setting
- CONFIGURED → UPDATED: When user manually toggles ResponsesAPI
- No automatic migration for existing conversations

### 3. APIRequest

**Description**: Request object sent to OpenAI-compatible endpoints
**Location**: Runtime object in backend API calls

**Fields**:
- `messages`: Array - Conversation messages
- `model`: string - Selected model
- `responsesAPI`: boolean - Format flag from conversation settings
- `azureApiVersion`: string | undefined - Conditionally included for Azure

**Transformation Rules**:
- If `responsesAPI === true && endpoint === 'azure'`: Remove `azureApiVersion` field
- If `responsesAPI === false && endpoint === 'azure'`: Include `azureApiVersion`
- For all endpoints: Include `responsesAPI` flag in request

## Relationships

```
EndpointConfiguration (1) ←→ (N) ConversationSettings
    - Each conversation has one active endpoint configuration
    - Configuration can be shared across multiple conversations

ConversationSettings (1) → (N) APIRequest
    - Each conversation generates multiple API requests
    - Settings determine request format

EndpointConfiguration → APIRequest
    - Configuration determines request structure
    - Azure-specific logic applied during transformation
```

## Data Flow

1. **New Conversation Creation**:
   - User selects endpoint type
   - System applies default `responsesAPI = true`
   - Settings stored in ConversationSettings

2. **Existing Conversation Load**:
   - Retrieve ConversationSettings from MongoDB
   - Display current `responsesAPI` state (no migration)
   - Allow manual toggle if desired

3. **API Request Generation**:
   - Read ConversationSettings
   - Apply EndpointConfiguration rules
   - Transform to appropriate APIRequest format

4. **Settings Toggle**:
   - User changes ResponsesAPI in UI
   - Update ConversationSettings immediately
   - Next APIRequest uses new format

## Migration Strategy

**No automatic migration** - Per requirements:
- Existing conversations keep their current settings
- Only new conversations get the new default
- Users can manually update if desired

## Persistence

**MongoDB Collections**:
- `conversations`: Stores ConversationSettings with responsesAPI field
- No new collections needed

**In-Memory**:
- EndpointConfiguration cached for performance
- APIRequest objects are transient

## Constraints

1. **Backward Compatibility**: Must not break existing conversations
2. **Synchronization**: UI and backend must always reflect same state
3. **Performance**: Toggle effect must be < 100ms
4. **Forced Enablement**: Continue using ResponsesAPI even on incompatible endpoints