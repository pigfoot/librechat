# Quickstart: ResponsesAPI Default Enablement

**Feature**: Enable ResponsesAPI by default for OpenAI-compatible endpoints
**Branch**: `001-responsesapi-default`

## Overview

This feature ensures ResponsesAPI is enabled by default for all new conversations using OpenAI, Azure, or Custom endpoints, fixing the UI-backend synchronization issue and simplifying Azure endpoint configuration.

## Key Changes

### 1. Default Values
- New conversations: ResponsesAPI = **true** by default
- Existing conversations: Keep current settings (no migration)
- All OpenAI-compatible endpoints affected (OpenAI, Azure, Custom)

### 2. Azure Special Handling
- When ResponsesAPI is enabled: API version parameter is **omitted**
- When ResponsesAPI is disabled: API version parameter is included
- Automatic handling, no user configuration needed

### 3. UI-Backend Sync
- UI displays ResponsesAPI state accurately
- Backend processes requests according to UI state
- Toggle changes take effect immediately (< 100ms)

## Implementation Checklist

### Backend Changes

1. **Update Default Schema** (`/packages/data-schemas/src/schema/defaults.ts`):
   ```typescript
   responsesAPI: true  // Set as default for new conversations
   ```

2. **Modify OpenAI Client** (`/api/app/clients/OpenAIClient.js`):
   - Add Azure API version conditional logic
   - Ensure ResponsesAPI flag is processed correctly

3. **Update Conversation Model** (`/api/models/Conversation.js`):
   - Preserve ResponsesAPI state for existing conversations
   - Apply new default only to new conversations

### Frontend Changes

1. **Update Endpoint Components** (`/client/src/components/Endpoints/`):
   - Use backend default values
   - Ensure toggle reflects actual state

2. **Modify Conversation Hooks** (`/client/src/hooks/Conversations/`):
   - Sync with backend ResponsesAPI state
   - Handle toggle events properly

## Testing Guide

### Test Scenario 1: New Conversation
1. Start a new conversation with OpenAI endpoint
2. Verify ResponsesAPI is enabled by default
3. Send a message
4. Confirm backend uses ResponsesAPI format

### Test Scenario 2: Existing Conversation
1. Open a pre-existing conversation
2. Verify current ResponsesAPI state is preserved
3. No automatic migration should occur
4. Manual toggle should still work

### Test Scenario 3: Azure Endpoint
1. Configure Azure endpoint
2. Enable ResponsesAPI
3. Verify API calls don't include API version parameter
4. Disable ResponsesAPI
5. Verify API version parameter is included

### Test Scenario 4: UI-Backend Sync
1. Toggle ResponsesAPI in UI
2. Send a message immediately
3. Verify backend uses correct format
4. Response time should be < 100ms

### Test Scenario 5: Endpoint Switching
1. Start with OpenAI endpoint (ResponsesAPI enabled)
2. Switch to Azure endpoint
3. Verify ResponsesAPI setting is maintained
4. Switch to Custom endpoint
5. Verify setting persists

## Rollback Plan

If issues occur:
1. Revert schema default to `responsesAPI: false`
2. Clear browser cache to reset UI state
3. Existing conversations remain unaffected
4. No database migration needed for rollback

## Monitoring

After deployment, monitor:
- API error rates (especially for Custom endpoints)
- Azure endpoint success rates
- UI-backend state mismatch reports
- Response time for toggle operations

## FAQ

**Q: Will my existing conversations be affected?**
A: No, existing conversations keep their current ResponsesAPI settings.

**Q: What if my custom endpoint doesn't support ResponsesAPI?**
A: The system will still use ResponsesAPI format. Errors are expected and acceptable per requirements.

**Q: Can I still manually toggle ResponsesAPI?**
A: Yes, manual toggle remains available and takes immediate effect.

**Q: Why is Azure API version removed with ResponsesAPI?**
A: Per Microsoft Azure AI Foundry documentation, ResponsesAPI doesn't require API versioning.

## Support

For issues or questions:
1. Check existing conversation settings in MongoDB
2. Verify UI console for state synchronization errors
3. Review API request logs for format verification