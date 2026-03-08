# Stories: AI Studio Inline Brand Voice Selection

---

## Story 1: [BE] Accept per-message brand voice and style overrides in AI chat endpoint

### Description:
As a ContentStudio user, I want to pass a specific brand voice and/or style with each AI chat message so that my selection applies only to that conversation — without changing my global defaults.

---

### Workflow:
1. User sends a message in AI Studio with a brand voice and/or style selected in the chatbox
2. The frontend includes `brand_voice_id` and/or `style_id` in the `chatWithStreaming` request payload
3. The backend receives these optional fields and uses them to look up the corresponding brand voice and style from the workspace's AI Content Library Profile
4. The AI agent receives the selected brand voice/style data in the `metadata.brand_guidelines` field — same format as today
5. If neither `brand_voice_id` nor `style_id` is provided, the backend falls back to reading the user's persisted defaults from `post_generation_settings` (existing behavior, no change)
6. The user's global `post_generation_settings` is **never modified** by this flow

---

### Acceptance criteria:
- [ ] `AiChatRequest.php` accepts optional `brand_voice_id` (nullable string) and `style_id` (nullable string) fields
- [ ] When `brand_voice_id` is provided in the payload, `extractBrandVoicePrompt()` looks up that specific brand voice from the workspace profile instead of reading from `post_generation_settings`
- [ ] When `style_id` is provided in the payload, `extractBrandVoicePrompt()` looks up that specific style from the workspace profile instead of reading from `post_generation_settings`
- [ ] When `brand_voice_id` is explicitly set to `null` in the payload, no brand voice is applied (user chose "None") — even if a default is set in `post_generation_settings`
- [ ] When `style_id` is explicitly set to `null` in the payload, no style is applied (user chose "None") — even if a default is set
- [ ] When neither field is present in the payload (key absent), fallback to existing `post_generation_settings` behavior (backwards compatible)
- [ ] The `brand_guidelines` data sent to the Agno agent follows the exact same format as today — no change to the Python agent contract
- [ ] The user's `post_generation_settings` document is **never** written to or modified by this endpoint
- [ ] Invalid `brand_voice_id` or `style_id` values (not found in profile) gracefully fall back to no brand voice/style rather than erroring

---

### Mock-ups:
N/A — backend-only story.

---

### Impact on existing data:
- No schema changes. The `brand_voice_id` and `style_id` are looked up from the existing `ai_content_library_profiles` collection at request time.
- No changes to persisted `post_generation_settings` — existing behavior is the fallback.

---

### Impact on other products:
- No impact on mobile apps, Chrome extension, or white-label — this is an additive optional parameter on an existing endpoint.
- AI Content Library continues to use `post_generation_settings` as before (unaffected).

---

### Dependencies:
None — this story can be developed independently. The frontend story **[FE] Add inline brand voice/style selector to AI Studio chatbox** will consume these new fields.

---

### Global quality & compliance (wherever applicable)
- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support — N/A, no user-facing strings
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 2: [FE] Add inline brand voice/style selector to AI Studio chatbox

### Description:
As a ContentStudio user, I want to select or change my brand voice and style directly from the AI Studio chatbox so that I can quickly apply brand guidelines to my AI-generated content without navigating away or changing my global defaults.

---

### Workflow:
1. User opens AI Studio (Home dashboard or `/ai-chat` modal)
2. In the ChatInput toolbar row (next to Upload, Saved Prompts, Audio, and Generation Type controls), user sees a **Brand Voice button** — a compact pill showing a `Lightbulb` icon and the active brand voice name (or "Brand Voice" if none selected)
3. User clicks the button — a **lightweight popover panel** appears anchored above the button (not a full modal). The popover contains:
   - A **"Style"** label with an `ℹ` tooltip icon, and a single-select dropdown listing all available styles from the user's AI profile, plus a "None" option at the top
   - A **"Brand Voice"** label with an `ℹ` tooltip icon, and a single-select dropdown listing all available brand voices from the user's AI profile, plus a "None" option at the top
   - A divider line, then a **"Manage brand knowledge"** link at the bottom that navigates to Settings → Brand Knowledge
4. User selects a style and/or brand voice — selections apply **immediately** on click (no save/cancel buttons needed). The popover stays open so the user can adjust both fields.
5. The toolbar button label updates to show the selected brand voice name (e.g., "Casual Brand")
6. User closes the popover by clicking outside it or pressing Escape
7. User types a message and sends it — the selected brand voice ID and style ID are included in the chat request payload
8. The selection persists for the duration of the chat session (new messages in the same chat keep the selection) but **does not update the user's global defaults**
9. When user starts a new chat, the selector resets to the user's global default (from `post_generation_settings`)
10. User can re-open the popover and change the selection at any time mid-conversation — the next message uses the new selection
11. **If user has no brand voices set up:** Clicking the button shows an empty-state popover with a heading, description, and a "Set up now" CTA button that navigates to Settings → Brand Knowledge

---

### Acceptance criteria:
- [ ] A brand voice button appears in the ChatInput toolbar row, between the Saved Prompts button and the Audio Input button
- [ ] The button uses the existing `useSetup` composable to access `AIUserProfile` data (brand voices and styles list)
- [ ] Clicking the button opens a **popover panel** (not a modal) anchored above the button
- [ ] The popover contains two independent single-select dropdowns: one for Style and one for Brand Voice
- [ ] Each dropdown lists the respective items from `AIUserProfile.styles` / `AIUserProfile.brand_voices` with a "None" option at the top
- [ ] Selections apply **immediately** on click — no save/cancel buttons in the popover
- [ ] The popover stays open after selecting a value so the user can adjust both Style and Brand Voice without re-opening
- [ ] The popover closes on click outside or Escape keypress
- [ ] Selected brand voice name is shown on the toolbar button as a truncated label (max 20 characters + ellipsis)
- [ ] When no brand voice is selected, the button shows "Brand Voice" as the label with a `Lightbulb` icon
- [ ] When a brand voice is selected, the button shows the brand voice name with a `Lightbulb` icon in the primary color (`text-primary-cs-500`)
- [ ] Selection state is maintained per chat session — switching chats or starting a new chat resets to the user's global default
- [ ] The `sendChatMessageStream` function in `AIChatMain.vue` includes `brand_voice_id` and `style_id` in the payload when available
- [ ] Selecting "None" for brand voice sends `brand_voice_id: null` explicitly (tells backend to skip brand voice)
- [ ] Selecting "None" for style sends `style_id: null` explicitly
- [ ] The popover does **not** call `savePostSettings()` — it never persists the selection to the user's AI profile
- [ ] For users without an AI profile (no brand voices/styles set up), clicking the button shows an empty-state popover with a heading, description, and "Set up now" CTA
- [ ] The "Manage brand knowledge" link at the bottom of the popover navigates to `{ name: 'brand-settings' }` route
- [ ] The existing header "Brand Knowledge" button in `ChatHeader.vue` continues to work as before (manages global defaults)
- [ ] The popover renders correctly at z-index 1200 or higher (same level as the Generation Type dropdown) to display above the chat modal
- [ ] On mobile viewports, the brand voice label text is hidden (icon only), consistent with how the Generation Type dropdown hides its label text on small screens
- [ ] Each dropdown item shows a checkmark icon next to the currently selected option

---

### UI Copy:

**Brand Voice selector button:**
- Default label (nothing selected): `"Brand Voice"`
- With selection: `"{brand voice name}"` (truncated to 20 chars)
- Icon: `Lightbulb` (from Lucide)

**Dropdown header:**
- Section label for styles: `"Style"`
- Section label for brand voices: `"Brand Voice"`
- Style tooltip (on `ℹ` icon): `"Choose a visual style for AI-generated images. This sets the look and feel — colors, logo, and design — so images match your brand."`
- Brand voice tooltip (on `ℹ` icon): `"Choose a brand voice to shape how AI writes for you. This sets the tone, personality, and language style — so content sounds like your brand."`

**None option:**
- Label: `"None"`

**Manage link:**
- Label: `"Manage brand knowledge"`
- Tooltip: `"Go to Settings → Brand Knowledge to add or edit your brand voices and styles"`

**Empty state (no brand voices set up):**
- Heading: `"Set up your brand voice"`
- Description: `"Add your brand voice so AI-generated content matches your tone and style. It only takes a minute."`
- CTA button: `"Set up now"`

**Dropdown item tooltips (on hover, for long names):**
- Show full name if truncated in the dropdown item

---

### Mock-ups:
N/A — no designer mockups provided. Follow the existing dropdown pattern used by the Generation Type selector in `ChatInput.vue` for visual consistency. The Brand Voice dropdown should match the same styling: `bg-[#E6F0FC]` button background, same border-radius, same dropdown item sizing.

---

### Impact on existing data:
- No data changes. The selector reads from the existing `AIUserProfile` data (already fetched by `useSetup` composable).
- Session-level selection is stored in component/composable state only — not persisted anywhere.

---

### Impact on other products:
- **Mobile apps:** No impact — AI features are web-only.
- **Chrome extension:** No impact — AI Studio is not in the Chrome extension.
- **White-label:** The selector uses theme-aware classes (`text-primary-cs-500`, `bg-primary-cs-50`) for the active state indicator. No hardcoded colors.
- **AI Content Library:** The existing `BrandVoiceSettingsModal` and global `post_generation_settings` flow remain unchanged.

---

### Dependencies:
- Depends on **[BE] Accept per-message brand voice and style overrides in AI chat endpoint** for the backend to honor the `brand_voice_id` and `style_id` fields in the payload.

---

### Global quality & compliance (wherever applicable)
- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
