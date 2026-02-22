# Epic & Stories: Smart Scheduling via AI V2

**Epic ID:** 109981 (existing — update description only)
**PRD Doc ID:** 69998c87-d4ac-4dfa-b7de-d45564d1d8ad (existing Shortcut doc)
**Target Sprint:** 23 Feb – 06 March 2026 (iteration ID: 110692)

---

## Epic Description (updated)

### Smart Scheduling via AI V2 — Guided Conversational Workflow

Smart Scheduling via AI V2 redesigns the AI Studio scheduling experience from a rigid, prompt-triggered system into a guided, conversational workflow. Instead of silently executing scheduling commands, the system now clarifies user intent first, generates content progressively, and only moves into scheduling after explicit user confirmation.

The update introduces four structured entry paths (Start from Scratch, I Have Captions, I Have Media, Plan in Bulk), a numbered account selection interface with fuzzy matching, per-post override controls, and a final review screen before any posts go live. All generated posts default to Draft until the user explicitly chooses to schedule.

This release prioritizes clarity, flexibility, and user control while preserving the speed and bulk efficiency that power users rely on.

---

## Story Index

| # | Title | Team | Priority | Project |
|---|---|---|---|---|
| 1 | [BE] Implement guided scheduling workflow orchestration for AI Studio agent | Backend | P0 | Web App |
| 2 | [BE] Extend CreatePostTool to support bulk post creation with auto time distribution | Backend | P0 | Web App |
| 3 | [BE] Add fuzzy account matching and numbered list format to FetchSocialAccountsTool | Backend | P0 | Web App |
| 4 | [FE] Build Smart Scheduling entry experience with guided workflow path cards | Frontend | P0 | Web App |
| 5 | [FE] Build structured post preview cards in AI Studio Smart Scheduling chat | Frontend | P0 | Web App |
| 6 | [FE] Build account selection UI for AI Studio Smart Scheduling | Frontend | P0 | Web App |
| 7 | [FE] Build scheduling review and confirmation screen for AI Studio Smart Scheduling | Frontend | P0 | Web App |

---

## Story 1

### [BE] Implement guided scheduling workflow orchestration for AI Studio agent

**Description:**

The AI Studio agent currently triggers scheduling as soon as it detects a scheduling-intent prompt, immediately entering MCP tool calls without clarifying user needs first. This causes confusion, brittle flows, and high drop-off.

This story overhauls the agent orchestration layer to implement the new guided conversational workflow as defined in the PRD. The agent must now follow a structured pattern: clarify intent → generate content progressively → ask explicitly whether to schedule → collect accounts → collect time → present review → confirm.

The agent handles four entry paths — Start from Scratch, I Have Captions, I Have Media, Plan in Bulk — each with its own clarification sub-flow. All generated posts are defaulted to Draft status. No scheduling or post creation occurs without explicit user confirmation.

From codebase: The agent runs through `app/Services/AI/AgnoAgentServices.php` and `app/Services/AI/AiAgentService.php`. System prompts / agent instructions are the primary lever to implement this new workflow logic.

---

**Workflow:**

1. User activates Smart Scheduling by clicking an entry path card (handled by FE) or typing a scheduling-related prompt in AI Studio
2. Agent detects scheduling intent from either the structured entry path metadata or the free-text prompt
3. **Path A — Start from Scratch:**
   - Agent asks: "What type of content? (Caption only / Caption + Image / Caption + Video)"
   - Agent asks: "How many posts?" and "What topic or keywords?"
   - Agent shows execution plan: "I'll generate [N] [type] posts about [topic]"
   - Agent streams post generation progressively — each post appears as a structured block in the chat
   - All posts default to Draft
   - Agent asks: "Would you like to schedule these posts, save them as drafts, or edit one?"
4. **Path B — I Have Captions:**
   - Agent asks user to paste their caption(s)
   - Agent detects single vs. multiple captions
   - Agent asks: "Schedule as-is? Or add images/video first?"
   - Agent creates draft post(s) from pasted captions
   - Agent asks: "Would you like to schedule these now?"
5. **Path C — I Have Media:**
   - Agent prompts user to upload media (handled by existing media upload UI)
   - Agent generates caption(s) for the uploaded media
   - Agent shows generated posts as draft cards
   - Agent asks: "Would you like to schedule these?"
6. **Path D — Plan in Bulk:**
   - Agent asks: "Bulk captions, bulk image posts, or bulk video posts?"
   - If bulk captions: asks quantity, topic, date range → generates in batch
   - If bulk image/video: presents "Use the Bulk Composer tool" CTA and guides user there; resumes scheduling on return
7. For all paths, once user confirms they want to schedule:
   - Agent asks for account selection (triggers account selection UI — see [FE] story)
   - Agent asks for time or offers auto-distribution across date range
   - Agent presents review summary
   - Agent waits for "Confirm" before executing post creation
8. Agent never locks the user into a workflow state — if the user types "start over" or asks a different question, the agent resets gracefully

---

**Acceptance Criteria:**

- [ ] Agent correctly detects scheduling intent from both entry path metadata and free-text prompts
- [ ] Agent asks clarifying questions before generating — does not silently start generation on first message
- [ ] Agent follows the correct question sequence for each of the 4 entry paths (Scratch, Captions, Media, Bulk)
- [ ] All AI-generated posts are created with `publish_type: draft` — no post goes live without explicit confirmation
- [ ] Agent never executes a `create_post` MCP call without first presenting a review step and receiving an explicit "Confirm" from the user
- [ ] Agent can handle "start over", "cancel", and off-topic messages mid-flow without crashing or repeating previous steps
- [ ] Bulk video/image path correctly routes user to Bulk Composer CTA instead of attempting in-chat generation
- [ ] Agent returns to scheduling resume state when user comes back from Bulk Composer
- [ ] Streaming works — agent outputs post generation progressively, not all at once
- [ ] Agent handles edge cases: no connected accounts (shows connect CTA message), empty topic input (re-prompts), count = 0 (re-prompts)

---

**Mock-ups:** See PRD section 7 — User Flow and section 6.1 Workflow Definitions (C/D/E)

**Impact on existing data:** AI chat session documents (MongoDB) will carry new `scheduling_workflow_state` fields: `current_step`, `draft_posts[]`, `selected_accounts[]`, `scheduling_time`, `entry_path`. TTL: 24h after last message.

**Impact on other products:** No impact on Planner, Composer, or mobile apps directly. Posts created as drafts appear in the Planner as draft posts.

**Dependencies:** None

**Global quality & compliance:**
- [ ] Mobile responsiveness — N/A, AI Studio is web-only; mobile AI features are out of scope
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 2

### [BE] Extend CreatePostTool to support bulk post creation with auto time distribution

**Description:**

The current `CreatePostTool` (`app/Mcp/Tools/CreatePostTool.php`) accepts a single post. The guided scheduling workflow requires creating multiple posts in one operation (e.g., 7 posts for the week). This story extends the tool to accept an array of posts and handles auto time distribution when no explicit times are provided.

Auto-distribution rule: If the user provides a date range but no specific times, distribute posts evenly across the range at 10:00 AM workspace time by default. Users may override the default time globally or per post before confirmation.

---

**Workflow:**

1. AI agent calls `CreatePostTool` with an array of post objects (text, media, accounts, publish_type, scheduled_at)
2. Tool validates the array: each entry must have text_content and accounts
3. For posts with `publish_type: scheduled` but no `scheduled_at`: apply auto-distribution logic
   - Divide date range evenly by post count
   - Assign 10:00 AM (workspace timezone) as default time for each slot
4. Tool creates all posts in a single transaction (or best-effort sequential, with rollback on failure)
5. Tool returns an array of created post objects: `[{id, text_preview, scheduled_at, publish_type, accounts}]`
6. On partial failure: tool returns both successful IDs and failed entries with error reasons

---

**Acceptance Criteria:**

- [ ] `CreatePostTool` accepts both a single post object (backwards compatible) and an array of post objects
- [ ] All posts in the array are created when called — not just the first one
- [ ] Auto time-distribution: given a date range and N posts, distributes evenly at 10:00 AM workspace time
- [ ] User-specified `scheduled_at` overrides auto-distribution for that specific post
- [ ] Tool returns an array of created post IDs and previews on success
- [ ] On partial failure, tool returns which posts succeeded (with IDs) and which failed (with reasons) — does not silently swallow errors
- [ ] Existing single-post callers remain unaffected (backwards compatible)
- [ ] All posts respect the `publish_type` field — drafts are not auto-scheduled

---

**Mock-ups:** N/A — backend only

**Impact on existing data:** No schema changes. Posts are created using the existing post creation pipeline.

**Impact on other products:** Posts created by this tool appear in the Planner and Composer post lists as normal.

**Dependencies:** None

**Global quality & compliance:**
- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 3

### [BE] Add fuzzy account matching and numbered list format to FetchSocialAccountsTool

**Description:**

The current `FetchSocialAccountsTool` (`app/Mcp/Tools/FetchSocialAccountsTool.php`) returns a raw paginated account list. When used in the guided scheduling workflow, users need to select accounts by typing a number (e.g., "1, 3") or a partial account name (e.g., "Instagram brand"). The tool currently requires exact account name matching, which is a known friction point (listed as BR-5 in the PRD).

This story adds two capabilities:
1. **Numbered response format** — the tool returns accounts with a sequential number (`1`, `2`, `3`...) so users and the AI agent can reference them by number
2. **Fuzzy name matching** — when an agent or user passes a `search` parameter, the tool performs fuzzy matching on account_name and username fields (case-insensitive, partial match)

---

**Workflow:**

1. Agent calls `FetchSocialAccountsTool` for the current workspace
2. Tool returns accounts in numbered format: `[{number: 1, platform: "instagram", account_name: "Brand IG", status: "active"}, ...]`
3. Agent presents the numbered list to the user in chat (rendered as the account selection table by FE — see [FE] story)
4. User responds with numbers ("1, 3, 5") or names ("Instagram Brand", "brand ig")
5. Agent calls tool again with `selection` parameter: `{numbers: [1,3,5]}` or `{search: "brand ig"}`
6. Tool returns matched accounts with IDs for use in `CreatePostTool`
7. If no match found: tool returns `{matched: [], unmatched: ["brand ig"], suggestion: "Did you mean 'Brand IG (@brand_ig)'?"}` so the agent can re-prompt the user

---

**Acceptance Criteria:**

- [ ] `FetchSocialAccountsTool` returns accounts with a sequential `number` field starting at 1
- [ ] Tool accepts a `search` parameter and performs case-insensitive partial match on `account_name` and `username`
- [ ] Fuzzy matching handles common misspellings and partial names (e.g., "brand ig" matches "Brand IG (@brand_ig)")
- [ ] Tool accepts a `numbers` parameter (array of integers) and returns the matched accounts with full IDs
- [ ] When no match is found for a name search, tool returns a `suggestion` field with the closest match
- [ ] Existing callers using the tool without the new parameters are unaffected (backwards compatible)
- [ ] Only `active` accounts are returned by default; expired/inactive accounts are excluded unless explicitly requested

---

**Mock-ups:** N/A — backend only

**Impact on existing data:** No schema changes. Read-only changes to the tool.

**Impact on other products:** No impact on other features. This tool is used only by AI Studio scheduling.

**Dependencies:** None

**Global quality & compliance:**
- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 4

### [FE] Build Smart Scheduling entry experience with guided workflow path cards

**Description:**

When a user opens Smart Scheduling in AI Studio, they currently see a blank chat box with no guidance on what to type. This story introduces a structured entry section that appears at the top of the chat when the user first enters Smart Scheduling (or starts a new session). It replaces the empty "what now?" experience with four clear starting points plus an explanatory header.

The entry section auto-dismisses once the user selects a path or types a free-text message.

From codebase: This component lives inside `src/modules/AI-tools/AIChatMain.vue` and `ChatBox.vue`. The entry section renders before `ChatHistory.vue` content when no messages exist.

---

**Workflow:**

1. User navigates to AI Studio → Smart Scheduling
2. If no active chat session (new session), the entry section is displayed above the chat input
3. User sees the header and 4 path cards
4. User clicks a path card (e.g., "I Have Captions") — the entry section disappears and the chat starts with a pre-filled trigger message for that path
5. Alternatively, user types directly in the chat input — entry section disappears and free-text message is sent
6. If the user returns to a session with existing messages, the entry section is not shown (only shown for new/empty sessions)

---

**UI Specification:**

**Entry Section Container:**
- Position: centered in chat area, above the text input
- Background: white, rounded-xl, subtle `border border-gray-100`
- Max width: 640px (centered)

**Header:**
- Icon: calendar/sparkle icon (use existing design system icon)
- Headline: `Plan and schedule your content with AI`
- Subtext: `Tell me what you want to post — I'll help you create, review, and schedule it step by step. Or pick one of the options below to get started faster.`

**Path Cards (2×2 grid on desktop, stacked on mobile):**

Card 1 — Start from Scratch:
- Icon: pencil/compose icon
- Label: `Start from Scratch`
- Subtext: `Generate captions and images for new posts`
- Tooltip on hover: `Tell me a topic and I'll write and schedule posts for you — no existing content needed. Great for planning your week from zero.`
- On click: sends message `"I want to create posts from scratch"` to the agent

Card 2 — I Have Captions:
- Icon: text/lines icon
- Label: `I Have Captions`
- Subtext: `Paste your copy and decide what to do next`
- Tooltip on hover: `Already written your captions? Paste them here and I'll help you add images, schedule them, or clean them up.`
- On click: sends message `"I have captions ready"` to the agent

Card 3 — I Have Media:
- Icon: image/photo icon
- Label: `I Have Media`
- Subtext: `Upload images or videos and get captions written for you`
- Tooltip on hover: `Upload a photo or video and I'll write the caption for you, then help you schedule it across your accounts.`
- On click: sends message `"I want to create posts from my media"` to the agent

Card 4 — Plan in Bulk:
- Icon: grid/layers icon
- Label: `Plan in Bulk`
- Subtext: `Generate and schedule multiple posts at once`
- Tooltip on hover: `Need 10 posts for the next two weeks? Tell me how many, the topic, and the date range — I'll generate and schedule them all.`
- On click: sends message `"I want to bulk plan posts"` to the agent

**Empty state (shown when no accounts are connected):** N/A — this is the chat entry, not dependent on accounts.

**Loading state:** No skeleton needed — the entry section is always immediately available on load.

---

**Acceptance Criteria:**

- [ ] Entry section displays for new/empty chat sessions in Smart Scheduling
- [ ] Entry section does NOT display when an existing chat session has messages
- [ ] All 4 path cards are displayed in a 2×2 grid on desktop (≥ 768px) and stacked vertically on mobile
- [ ] Each card has the correct icon, label, and subtext as specified
- [ ] Hovering over a card shows the correct tooltip text
- [ ] Clicking a path card sends the correct trigger message to the AI agent and dismisses the entry section
- [ ] Typing in the chat input dismisses the entry section and sends the free-text message normally
- [ ] All colors use theme-aware Tailwind classes (`text-primary-cs-500`, `bg-primary-cs-50`, etc.) — no hardcoded hex or Tailwind color classes like `text-blue-600`
- [ ] Entry section is accessible: all cards are keyboard-navigable and have proper aria labels

---

**Mock-ups:** See PRD prototype: https://schedule-copilot-ai.lovable.app/ — Entry section

**Impact on existing data:** No data model changes.

**Impact on other products:** No impact on other modules.

**Dependencies:** Depends on **[BE] Implement guided scheduling workflow orchestration for AI Studio agent** (the entry path trigger messages must be recognized by the agent)

**Global quality & compliance:**
- [ ] Mobile responsiveness (frontend only)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 5

### [FE] Build structured post preview cards in AI Studio Smart Scheduling chat

**Description:**

When the AI agent generates posts during the Smart Scheduling workflow, they currently appear as raw markdown text in the chat. This is hard to read, feels unpolished, and doesn't allow users to take per-post actions (edit, remove, override time).

This story introduces `SchedulingPostCard` — a new chat message type that renders AI-generated posts as structured, visually distinct cards inside the chat. Each card shows the post content, target accounts, scheduled time, and draft status, with inline action buttons.

From codebase: The new message type is rendered in `ChatHistory.vue` / `BotChatTemplate.vue`. The backend agent returns a structured JSON block in the message that the frontend parses to render the card instead of plain text.

---

**Workflow:**

1. User reaches the post generation step in the Smart Scheduling workflow (after providing topic, count, etc.)
2. As the agent streams its response, each generated post appears as a card in the chat as soon as it's ready (progressive rendering)
3. Each card shows: caption text (truncated to 3 lines with "Show more"), platform icons for target accounts, scheduled time (or "Draft" if no time set), and status badge
4. Below the post cards, the agent shows: `"Here are your [N] posts. Ready to schedule, or would you like to edit one first?"`
5. User can interact with individual cards:
   - Click **Edit** → opens the post in the Composer in a new tab/modal
   - Click **Remove** → removes the card from the list and notifies the agent
   - Click **Override time** → shows a simple date/time text input inline in the card for that post only
6. After user interacts (or doesn't), the workflow continues with the scheduling step

---

**UI Specification:**

**Post Card Container:**
- Background: white
- Border: `border border-gray-200 rounded-xl`
- Padding: `p-4`
- Left accent bar: `3px solid` using `bg-primary-cs-500`
- Stacked vertically in chat with `gap-3`

**Card Header Row:**
- Left: Post number badge — `#1`, `#2` etc. — `text-xs font-medium text-gray-500 bg-gray-100 rounded px-2 py-0.5`
- Right: Status badge — "Draft" (gray) or "Scheduled" (green) — `text-xs rounded-full px-2 py-0.5`

**Caption Text:**
- `text-sm text-gray-800`
- Truncated to 3 lines with a `Show more` link if longer
- Full text shown on expand

**Accounts Row:**
- Label: `Accounts:` in `text-xs text-gray-500`
- Platform icons (16px) + account names
- If no accounts selected yet: `text-xs text-gray-400 italic` → `"Accounts not selected yet"`

**Scheduled Time Row:**
- Label: `Scheduled:` in `text-xs text-gray-500`
- Time value or `Draft` if unset
- If "Override time" is active: shows inline text input with placeholder `"e.g. Feb 28 at 10:00 AM"` and a `✓ Set` button

**Action Buttons Row (bottom of card):**
- `Edit` — opens post in Composer — `text-xs text-primary-cs-500 hover:underline`
- `Remove` — removes card from list — `text-xs text-red-500 hover:underline`
- `Override time` — shows inline date/time input — `text-xs text-gray-500 hover:underline`

**Inline "Override time" input:**
- Label: `Set a custom time for this post`
- Placeholder: `e.g. Feb 28 at 10:00 AM`
- Helper text: `Leave blank to keep the auto-scheduled time`
- Validation error: `Please enter a valid date and time (e.g. Feb 28 at 10:00 AM)`
- Buttons: `Set Time` (primary, small) / `Cancel` (text, small)

**Empty state:** N/A — cards only appear after generation.

**Loading state:** While agent is generating, each post slot shows a skeleton card with animated shimmer before the content streams in.

**Error state:** If the agent fails to generate a post, show an error card:
- Headline: `Couldn't generate this post`
- Subtext: `Something went wrong. You can try again or skip this one.`
- CTA: `Try Again`

---

**Acceptance Criteria:**

- [ ] Generated posts are rendered as structured cards, not raw markdown text
- [ ] Cards appear progressively as the agent streams each post (not all at once at the end)
- [ ] Each card shows: post number, caption text, accounts (or "not selected yet"), scheduled time (or "Draft"), status badge
- [ ] Captions longer than 3 lines are truncated with "Show more" — clicking expands full text
- [ ] "Edit" button opens the post in the Composer (new tab or modal)
- [ ] "Remove" button removes the card from the list with a toast: `"Post #[N] removed"`
- [ ] "Override time" shows an inline date/time input on the card; "Set Time" confirms; "Cancel" dismisses
- [ ] Overridden time is shown on the card and communicated to the agent
- [ ] Skeleton loading state shown for each post slot while streaming
- [ ] Error state card shown if generation fails for a specific post
- [ ] All card colors use theme-aware Tailwind classes — no hardcoded hex or `text-blue-*` / `bg-blue-*` classes

---

**Mock-ups:** See PRD prototype: https://schedule-copilot-ai.lovable.app/ — Post cards section

**Impact on existing data:** No data model changes. Cards are display-only until the user reaches the confirm step.

**Impact on other products:** No impact on Planner, Composer, or mobile apps.

**Dependencies:** Depends on **[BE] Implement guided scheduling workflow orchestration for AI Studio agent** (structured post data must come from the agent in a parseable format)

**Global quality & compliance:**
- [ ] Mobile responsiveness (frontend only)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 6

### [FE] Build account selection UI for AI Studio Smart Scheduling

**Description:**

When the user decides to schedule their generated posts, the AI agent asks them to select which accounts to post to. Currently, this requires the user to type exact account names — a major source of errors and frustration (BR-5 in the PRD).

This story introduces a structured account selection component that renders inline in the AI chat as a special message block. It shows a numbered list of connected social accounts and accepts input by number (e.g., `1, 3`) or fuzzy name (e.g., `instagram brand`). Accounts can be applied globally to all posts or overridden per post.

From codebase: Rendered in `ChatHistory.vue`. Account data comes from `FetchSocialAccountsTool` (extended in **[BE] Add fuzzy account matching and numbered list format to FetchSocialAccountsTool**).

---

**Workflow:**

1. User says "yes, schedule these" (or similar) after reviewing their generated posts
2. Agent triggers account selection — the Account Selection block renders in the chat
3. User sees the numbered account table with all active connected accounts
4. User types numbers or names in the chat input (e.g., `1, 3` or `brand instagram`) to select accounts
5. Selected accounts are highlighted in the table with a checkmark
6. Agent confirms: `"Got it — posting to: [Account 1], [Account 3]. Apply these accounts to all [N] posts?"`
7. User confirms (`yes` / `all`) or says `"different for each"` to trigger per-post mode
8. In per-post mode: agent asks account selection per post (same UI, one post at a time)
9. If no accounts are connected: "No accounts connected" empty state shown with "Connect an Account" CTA
10. Once accounts are confirmed, workflow moves to time selection step

---

**UI Specification:**

**Account Selection Block Container:**
- Renders as a special chat message block (bot side)
- Background: `bg-gray-50` / `rounded-xl` / `border border-gray-200`
- Padding: `p-4`

**Block Header:**
- Headline: `Choose accounts to publish to`
- Subtext: `Type a number (e.g. "1, 3") or account name to select. You can assign different accounts per post after.`

**Account Table:**
Columns: `#` | Platform icon | Account name | Username | Status

- `#` column: 1, 2, 3... — `text-sm font-medium text-gray-700`
- Platform icon: 20px colored platform icon (Instagram purple, Facebook blue, etc.)
- Account name: `text-sm text-gray-800`
- Username: `text-xs text-gray-400`
- Status: `Active` badge (green dot + text) or `Expired` badge (red dot + text)
- Selected rows: highlighted with `bg-primary-cs-50 border-l-2 border-primary-cs-500`

**Selection helper text:**
- Below table: `text-xs text-gray-400`
- Copy: `Type numbers like "1, 3" or account names like "Brand Instagram" in the chat below.`

**Input hint (shown in the chat input placeholder when account selection is active):**
- Placeholder: `Type account numbers or names, e.g. "1, 3" or "Brand IG"`

**Info icon (ℹ) tooltip next to "Choose accounts to publish to" header:**
- Hover content: `These are the social accounts connected to your ContentStudio workspace. Select one or more to publish your posts. You can always change which account each post goes to before confirming.`

**Empty state (no accounts connected):**
- Illustration: plug/disconnect icon (existing empty state illustration from design system)
- Headline: `No social accounts connected yet`
- Subtext: `Connect your Instagram, Facebook, LinkedIn, or other accounts to start scheduling posts directly from AI Studio.`
- CTA button: `Connect an Account` → links to Settings → Social Accounts

**Loading state:** Skeleton table rows (3 rows) with shimmer while accounts are being fetched.

**Error state (fetch failed):**
- Copy: `Couldn't load your accounts. Please try again.`
- Retry link: `Try again`

---

**Acceptance Criteria:**

- [ ] Account selection block renders inline in the AI Studio chat at the right step in the guided workflow
- [ ] All active connected accounts are shown in a numbered table with platform icon, account name, username, and status
- [ ] Expired/inactive accounts are shown but visually deemphasized (grayed out, not selectable)
- [ ] User can select accounts by typing numbers ("1, 3, 5") or partial account names ("brand ig") in the chat input
- [ ] Selected accounts are visually highlighted in the table (left accent + background color)
- [ ] Agent confirms selected accounts after input and asks: "Apply to all posts?" — global mode
- [ ] User can opt for per-post account assignment — agent prompts per post using same UI
- [ ] Empty state shown when no accounts are connected, with "Connect an Account" CTA linking to account settings
- [ ] Loading skeleton shown while accounts are being fetched
- [ ] Error state shown with retry option if account fetch fails
- [ ] Info icon tooltip shows correct copy on hover
- [ ] All colors use theme-aware Tailwind classes — no hardcoded colors

---

**Mock-ups:** See PRD prototype: https://schedule-copilot-ai.lovable.app/ — Account selection section

**Impact on existing data:** No data model changes. Account selection state is held in the AI Studio scheduling session.

**Impact on other products:** No impact. Account data is read-only.

**Dependencies:**
- Depends on **[BE] Add fuzzy account matching and numbered list format to FetchSocialAccountsTool**
- Depends on **[BE] Implement guided scheduling workflow orchestration for AI Studio agent** (workflow must trigger account selection step)

**Global quality & compliance:**
- [ ] Mobile responsiveness (frontend only)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

## Story 7

### [FE] Build scheduling review and confirmation screen for AI Studio Smart Scheduling

**Description:**

The final step of the Smart Scheduling workflow is a review screen that appears before any posts go live. The current workflow has no explicit review step — posts are scheduled without a final confirmation, which is a safety and trust problem.

This story introduces a structured Review Panel that renders inline in the AI chat after accounts and times are collected. It shows a summary of what's about to be scheduled: workspace, accounts, post count, publish type, and a post-by-post table. The user must click "Schedule All Posts" to confirm — there is no auto-execution.

From codebase: Renders as a special bot message block in `ChatHistory.vue`. After user confirms, the agent calls `CreatePostTool` in bulk (extended in **[BE] Extend CreatePostTool to support bulk post creation**).

---

**Workflow:**

1. After accounts and times are collected, the agent says: "Here's your schedule summary — please review before I post anything."
2. The Review Panel appears in the chat
3. User reviews: workspace, accounts, post count, publish type, and the post table
4. If satisfied: user clicks "Schedule All Posts" → confirmation dialog appears
5. Confirmation dialog:
   - Title: `Schedule [N] posts?`
   - Body: `This will create [N] scheduled posts to [account list]. You can view and edit them in your Planner after publishing.`
   - Primary CTA: `Yes, Schedule All`
   - Secondary CTA: `Cancel`
6. After user clicks "Yes, Schedule All": agent executes bulk `CreatePostTool` call, shows success state
7. If user clicks "Make Changes": Review Panel collapses, chat returns to active conversation so user can give instructions
8. Success state in chat: `"Done! Your [N] posts have been scheduled. View them in your Planner →"`

---

**UI Specification:**

**Review Panel Container:**
- Background: white
- Border: `border border-gray-200 rounded-xl`
- Padding: `p-5`
- Full-width within the chat message area

**Panel Header:**
- Headline: `Ready to schedule?`
- Subtext: `Review your posts below. Once you confirm, they'll be added to your queue.`

**Summary Row:**
Three summary pills in a horizontal row:
- `🏢 Workspace: [Workspace Name]`
- `👤 Accounts: [N] selected`
- `📄 [N] posts · Scheduled`

All pill styling: `bg-gray-50 rounded-lg px-3 py-2 text-sm text-gray-700`

**Post Table:**
Columns: `#` | Caption Preview | Accounts | Scheduled Time | Status

- `#`: post number — `text-sm text-gray-500`
- Caption Preview: first 80 characters of caption + `...` — `text-sm text-gray-800`
- Accounts: platform icons only (16px each, stacked/row), with full account names on hover tooltip
- Scheduled Time: `Feb 28 · 10:00 AM` — `text-sm text-gray-700`
- Status: "Scheduled" (green badge) or "Draft" (gray badge)

**Action Buttons (bottom of panel):**
- Primary: `Schedule All Posts` — full-width primary button, `bg-primary-cs-500 text-white rounded-lg`
- Secondary: `Make Changes` — full-width outline/ghost button below, `text-primary-cs-500 border border-primary-cs-200`

**Confirmation Dialog:**
- Modal, centered
- Title: `Schedule [N] posts?`
- Body: `This will add [N] scheduled posts to your Planner. You can still edit or delete them there.`
- Primary CTA: `Yes, Schedule All` (primary button color)
- Secondary CTA: `Cancel` (text/ghost)

**Success state (in chat, after scheduling):**
- Bot message with success icon (green checkmark)
- Copy: `Done! [N] posts have been scheduled and added to your Planner. ✓`
- CTA link: `View in Planner →` (opens Planner in a new tab filtered to these posts)

**Error state (if scheduling fails):**
- Copy: `Something went wrong — [N] of [N] posts weren't scheduled.`
- Show which posts failed (by number) and the reason if available
- CTA: `Retry Failed Posts` / `View in Planner` (for the ones that succeeded)

**Loading state (while scheduling is in progress):**
- "Schedule All Posts" button shows spinner + disabled state
- Copy changes to: `Scheduling your posts...`
- Do not close or hide the panel while in progress

---

**Acceptance Criteria:**

- [ ] Review Panel renders in the chat after accounts and times are collected — before any posts are created
- [ ] Review Panel shows: workspace name, account count, post count, publish type
- [ ] Post table shows each post's: number, caption preview (80 chars + ellipsis), platform icons for accounts, scheduled time, and status badge
- [ ] "Schedule All Posts" button shows a confirmation dialog before executing
- [ ] Confirmation dialog copy matches specification above (dynamic N and account names)
- [ ] "Make Changes" collapses the Review Panel and returns to active chat so user can modify
- [ ] After "Yes, Schedule All" is confirmed: posts are created via `CreatePostTool` bulk call — no posts are created before this step
- [ ] Success state shows correct post count and a "View in Planner" link
- [ ] "View in Planner" opens the Planner in a new tab with the relevant posts visible
- [ ] Error state shows which posts failed and which succeeded, with retry option for failed ones
- [ ] Loading state: "Schedule All Posts" button shows spinner and is disabled during API call
- [ ] All colors use theme-aware Tailwind classes — no hardcoded hex or `text-blue-*` classes

---

**Mock-ups:** See PRD prototype: https://schedule-copilot-ai.lovable.app/ — Review and confirmation section

**Impact on existing data:** Posts created by this action go into the standard posts collection. They appear in the Planner and Composer post list like any other post.

**Impact on other products:**
- **Planner:** Scheduled posts appear immediately in the Planner. The "View in Planner" link should deep-link to the right date range.
- **Chrome Extension / Mobile:** No impact — AI Studio is web-only.

**Dependencies:**
- Depends on **[BE] Extend CreatePostTool to support bulk post creation with auto time distribution** (bulk creation API must be available)
- Depends on **[FE] Build structured post preview cards in AI Studio Smart Scheduling chat** (post cards shown in review must use same card component)
- Depends on **[FE] Build account selection UI for AI Studio Smart Scheduling** (accounts must be collected before review is shown)

**Global quality & compliance:**
- [ ] Mobile responsiveness (frontend only)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
