# Research: AI Studio Inline Brand Voice Selection

## Current State

### How Brand Voice Works Today

**Header-level button (global, persistent):**
- `ChatHeader.vue` (line 77–91) shows a "Brand Knowledge" button in the AI Studio header bar, visible only if the user has an AI profile with styles + brand voices (`hasAIProfile` computed)
- `WelcomeRow.vue` (line 16–26) shows the same button on the Dashboard/Home page
- Both trigger `BrandVoiceSettingsModal.vue` — a popup with two dropdowns: **Style** and **Brand Voice**
- Clicking "Update" calls `savePostSettings()` from `useAIPostGeneration.js` (line 205), which POSTs to `savePostSettingsURL` — **persisting the selection to the user's AI profile on the server** (`post_generation_settings.style` and `post_generation_settings.brand_voice`)
- This means any change updates the user's default across the entire app (AI Content Library, AI Chat, etc.)

**Backend flow:**
- `AIController.php` → `processAgnoAgent()` (line 984) builds the chat payload
- `AiChatHelper::buildWorkflowInputs()` (line 133) calls `extractBrandVoicePrompt()` (line 913) which reads the persisted `post_generation_settings` from `AiContentLibraryProfileRepo::getProfileByWorkspaceId()`
- The brand voice data is sent to the Agno AI agent as `metadata.brand_guidelines`
- There is **no per-message brand voice override** — the backend always reads from the persisted profile

**Visibility problem:**
- The Brand Knowledge button is in the header bar, easy to miss
- Users who haven't set up brand knowledge don't see the button at all (`v-if="hasAIProfile"`)
- No indication near the chatbox about what brand voice is active or how to configure it

### Key Problems
1. **Persistent updates on user level** — changing brand voice in the modal updates it everywhere (AI Content Library, other chats, etc.), not just the current chat session
2. **Low discoverability** — button is tucked in the header; users without brand knowledge setup don't see it at all, no CTA to set it up

## What Needs to Change

### Frontend
- **Add a brand voice/style selector directly in or near the chatbox** (below the text input area, or as a pill/chip above the input), making it immediately visible and accessible
- The selector should be **session-scoped / per-message** — selections apply only to the current chat context and do NOT persist to the user's global profile
- For users who haven't set up brand knowledge yet, show a subtle CTA to set it up (links to Settings → Brand Knowledge)
- Show the currently active brand voice/style as a visible indicator near the input area
- Keep the existing header button as-is for users who want to change their global defaults

### Backend
- Accept optional `brand_voice_id` and `style_id` parameters in the `chatWithStreaming` payload
- In `extractBrandVoicePrompt()`, if these per-message overrides are present, use them instead of reading from the persisted profile
- No schema/database changes needed — the brand voices and styles already exist in the AI Content Library Profile

### UX Reference
Tools like ChatGPT and Jasper allow selecting a "voice" or "persona" directly in the chat input area via a small selector chip. This pattern is well-established for AI chat interfaces — a compact inline control that doesn't require opening a separate settings modal.

## Files Involved

### Frontend
| File | Change |
|---|---|
| `src/components/dashboard/ChatInput.vue` | Add inline brand voice/style selector UI near the input area |
| `src/modules/AI-tools/ChatBox.vue` | Pass brand voice/style selection to `sendChatMessage` |
| `src/modules/AI-tools/AIChatMain.vue` | Include `brand_voice_id` / `style_id` in the streaming payload (line ~1354) |
| `src/modules/publisher/ai-content-library/composables/useSetup.js` | Already provides `AIUserProfile` with brand voices/styles — reuse |
| `src/composables/useAIChat.js` | May need to track per-session brand voice/style selection |

### Backend
| File | Change |
|---|---|
| `app/Http/Controllers/AI/AIController.php` | Pass through new optional fields from request |
| `app/Http/Requests/AI/AiChatRequest.php` | Add optional `brand_voice_id` and `style_id` validation rules |
| `app/Helpers/Ai/AiChatHelper.php` | In `extractBrandVoicePrompt()`, prioritize per-message IDs over persisted defaults |
