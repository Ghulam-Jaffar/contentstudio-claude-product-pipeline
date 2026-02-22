# Codebase Analysis: Smart Scheduling via AI V2

> Note: Research/competitor analysis step skipped — PRD already exists. This file covers codebase analysis only.

---

## Existing AI Studio Infrastructure

### Backend

| File | Purpose |
|---|---|
| `app/Http/Controllers/AI/AIController.php` | Handles chats, messages, streaming via `processAgnoAgent()` |
| `app/Services/AI/AiAgentService.php` | AI agent orchestration |
| `app/Services/AI/AgnoAgentServices.php` | Agno framework integration (streaming) |
| `app/Helpers/Ai/AiChatHelper.php` | Maps chat messages to provider-agnostic format; supports TEXT, IMAGE_URL, VIDEO_URL, AI library posts |
| `routes/web/ai.php` | Chat endpoints: fetchChats, chatMessage, processAgnoAgent (streaming) |

### MCP Tools (app/Mcp/Tools/)

| Tool | Key Params | Notes |
|---|---|---|
| `FetchSocialAccountsTool.php` | workspace_id, auth_key, platform, page, per_page | Returns paginated accounts with platform, account_name, status |
| `CreatePostTool.php` | workspace_id, auth_key, text_content, accounts, publish_type, scheduled_at, images, video | Supports draft/scheduled/now; single post only currently |
| `FetchPostsTool.php` | auth_key, workspace_id, status[], date_from, date_to | Paginated post list |
| `DeletePostTool.php` | Post deletion | — |
| `ValidateTokenTool.php` | Token validation via /me | — |
| `FetchWorkspacesTool.php` | List workspaces | — |

### Frontend

| File/Module | Purpose |
|---|---|
| `src/modules/AI-tools/AIChatMain.vue` | Main wrapper; state: activeChat, chats, promptInput, modalToggler |
| `src/modules/AI-tools/ChatBox.vue` | Message input/output |
| `src/modules/AI-tools/ChatHistory.vue` | Message chronological display |
| `src/modules/AI-tools/BotChatTemplate.vue` | Bot response rendering |
| `src/modules/AI-tools/composables/chat.js` | Chat state management |
| `src/modules/planner_v2/` | Planner; patterns for scheduling UI (AccountDetailItem, DataTable, CalendarEvent) |

---

## Integration Points for V2

### Backend
1. **Extend AI agent system prompt / instructions** in `AgnoAgentServices.php` to understand 4 entry paths and guided flow
2. **Add bulk post creation** to `CreatePostTool.php` (currently single post only)
3. **Extend `FetchSocialAccountsTool.php`** to return numbered list format + support fuzzy name matching
4. **Add scheduling state to chat sessions** — store in-progress draft data (workflow step, posts, accounts, time) with TTL

### Frontend
1. **Extend `AIChatMain.vue`** — add `isSchedulingFlow` state, current draft post data
2. **New component: SmartSchedulingEntry.vue** — entry cards, shown in chat when scheduling mode starts
3. **New component: SchedulingPostCard.vue** — structured post preview card (replaces markdown output)
4. **New component: AccountSelectionTable.vue** — numbered account list for inline selection
5. **New component: SchedulingReviewPanel.vue** — full review table + confirm/make-changes
6. **Extend `ChatHistory.vue`** — render new message types (post cards, account tables, review panel)

---

## Technical Considerations

- **Chat persistence**: MongoDB (AI chat sessions). Scheduling workflow state can be stored in the session document (TTL 24h)
- **Streaming**: Already implemented via `fetchEventSource` — use for progressive post generation
- **Account data**: Already available via `FetchSocialAccountsTool` and `GET /api/v1/workspaces/{id}/accounts`
- **CreatePostTool**: Currently single-post only — needs array support for bulk scheduling
- **Auto-time distribution**: If no time provided, distribute evenly across date range at 10:00 AM — implement in backend tool
- **Draft default**: All generated posts default to `publish_type: draft` until user explicitly chooses to schedule
