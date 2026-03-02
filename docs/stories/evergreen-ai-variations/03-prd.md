# PRD: AI-Powered Variations in Evergreen Automation

**Author:** Product
**Last Updated:** 2026-03-03
**Status:** Approved
**Target Release:** Q1 2026

---

## 1. Overview

Evergreen automation (Recycle Posts) lets users build a library of posts that ContentStudio rotates and re-publishes automatically over time. Today, users can create multiple variations of each post manually — different caption wordings that the system cycles through to keep content fresh. Creating those variations is tedious: users write every version by hand.

This feature adds AI-powered variation generation directly inside the Compose Posts step (Step 2) of the Evergreen wizard. Users click a single button, set how many variations they want, and the AI generates them instantly — one post at a time or across all posts in one go. Generated variations are pushed straight to the post as regular variations; users can edit or delete them like any other. There is no approval flow.

---

## 2. Problem Statement

**What problem are we solving?**

Creating meaningful content variations for evergreen automations is time-consuming and often results in users publishing the same caption verbatim across cycles, defeating the purpose of the rotation feature. Users either skip creating variations (leaving only 1 copy per post) or duplicate the post and tweak manually, which doesn't scale.

**Who has this problem?**

- Social media managers running long-term evergreen strategies (content that repeats for weeks or months)
- Agencies managing multiple brand automations — high volume, low time per account
- Small business owners who understand the value of rotation but lack the copywriting bandwidth

**What happens if we don't solve it?**

- Evergreen automations remain underused — users set up 1 variation per post and the "rotation" value prop is wasted
- We lose differentiation vs. competitors who offer AI-assisted content scheduling
- Users churn from the automation module citing "too much work to set up properly"

---

## 3. Goals & Success Metrics

| Goal | Metric | Target | How We'll Measure |
|---|---|---|---|
| Increase average variations per evergreen post | Avg variations per post | From ~1.2 → 2.5+ | Product analytics on automation drafts |
| Drive AI credit consumption in the automation module | AI text credits used in evergreen context | 15% of total AI text credit usage | Credit deduction logs |
| Reduce time-to-complete for Step 2 | Session duration on Step 2 | -30% for automations with 3+ posts | Session timing |

---

## 4. Target Users

**Primary Persona:**
Social media manager or content creator — runs 3–10 evergreen automations, publishes to 5–20 accounts. Comfortable with AI tools, wants speed without sacrificing quality. Has an active ContentStudio plan with AI credits.

**Secondary Persona:**
Agency owner or account manager — managing multiple client automations. Uses bulk actions wherever available. Values "set and forget" over fine-grained control.

**Non-Users (explicitly out of scope):**
Users on free plans without AI text credits. Mobile app users (AI features are web-only).

---

## 5. User Stories / Jobs to Be Done

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| US-1 | Social media manager | generate multiple caption variations per post with one click | I don't have to write each version manually | Must Have |
| US-2 | Content creator | generate variations from an image-only post | AI writes the caption first, then creates variations | Must Have |
| US-3 | Agency owner | generate variations across all posts in one action | I can set up evergreen automations in minutes, not hours | Must Have |
| US-4 | User | see AI-generated variations instantly in my post cards | I can review and edit before the automation goes live | Must Have |
| US-5 | User | generate AI images alongside each variation | each variation has a unique visual, not just a different caption | Should Have |
| US-6 | User | give the AI custom instructions | the tone, length, and style match my brand | Should Have |
| US-7 | User | see how many credits I have before generating | I can plan my usage and know when to upgrade | Should Have |
| US-8 | User | continue working while generation runs in the background | I'm not blocked waiting for AI to finish | Must Have |

---

## 6. Requirements

### 6.1 Must Have (P0)

- **AI generate button per post**: `✨ Generate with AI` button in each post card header (alongside existing `+ Add variation` button)
- **Bulk generate button**: `✨ Generate variations for all` button above the post list (top-right), applies same settings to every post
- **Generate Variations modal (per post)**: source auto-detection, variation count stepper (1–10, default 3), optional AI image toggle, optional custom instructions textarea, credit state handling
- **Bulk Generate modal**: same controls + dynamic `{posts} × {variations} = {total}` label + warning notice about existing variations
- **AI generates caption first for image-only posts**: if a post has an image but no caption, AI analyses the image and generates a caption before creating variations; caption replaces the placeholder in Variation 1
- **AI variation badge**: generated variations display a small AI badge so users know which were AI-generated; styling is otherwise identical to manual variations
- **Variation count badge**: live pill in each post card header showing `{N} Variation / {N} Variations`; updates as variations are added or deleted
- **Loading card**: dashed border, spinner, appended to bottom of post card during generation; removed when complete
- **Background async generation**: generation runs as a backend job — closing the modal or navigating away does not cancel it; results are auto-saved to draft on completion
- **Credit validation — no text credits**: generation controls hidden, Generate button disabled, amber alert with upgrade link shown
- **Credit validation — partial credits (runs out mid-generation)**: generates as many variations as credits allow; shows a warning toast for partial results
- **Toast notifications**: full set covering single-post success, partial credits, zero credits, bulk success, bulk partial, bulk failure (see spec §13)

### 6.2 Should Have (P1)

- **AI image generation toggle**: on/off per modal; consumes 1 AI image credit per variation; disabled if image credits exhausted with upgrade prompt
- **Custom instructions textarea**: optional free-text field for tone/length/hashtag guidance; included in AI prompt
- **Navigation warning banner**: non-blocking banner appears at top of page if user tries to navigate away while generation is in progress; banner auto-dismisses when generation completes
- **Bulk processing**: posts processed in parallel where infrastructure allows; each shows its own loading card; engineering decides on parallelism strategy

### 6.3 Nice to Have (P2)

- **Real-time progress via websocket push**: instead of polling, variations appear via push event as soon as the job completes; fallback to polling if websocket unavailable
- **Regenerate individual variation**: re-run AI for a single existing AI variation (post-MVP, when inline variation editing is built out)

### 6.4 Explicitly Out of Scope

- Manual variation editor (`+ Add variation` button — placeholder stays but is not built in this spec)
- Full inline caption editing within the compose view (editing happens in the full post composer modal)
- Video credits (removed from UI entirely)
- Scheduling logic (Step 3)
- Analytics or performance data per variation
- Keep/Discard/Try Again approval flow — variations are pushed directly, no approval step
- Mobile app support — AI features are web-only
- Dark mode — ContentStudio does not support dark mode

---

## 7. User Flows

### 7.1 Generate Variations — Single Post (Happy Path)

1. User is on Step 2: Compose Posts in the Evergreen wizard
2. User sees each post card has a variation count badge and an `✨ Generate with AI` button in the header
3. User clicks `✨ Generate with AI` on a post → Generate Variations modal opens
4. Modal shows: source auto-detected (post caption / image / link + caption), count stepper defaulting to 3, image toggle (off), custom instructions textarea
5. User sets count to 4, optionally types custom instructions, clicks `✨ Generate`
6. Modal closes; a loading card with spinner appears at the bottom of the post card
7. Backend job runs: generates N variations; draft auto-saves when complete
8. Loading card is replaced by the new variation cards (same styling as existing, AI badge in header)
9. Variation count badge updates from "1 Variation" to "5 Variations"
10. Success toast: "4 variation(s) generated successfully."

### 7.2 Image-Only Post

1. User clicks `✨ Generate with AI` on a post that has an image but no caption
2. Modal shows source chip: `🖼 Image (caption will be generated first)` with explanatory hint
3. User sets count and clicks Generate
4. Loading card text: "Analysing image & generating caption first, then creating 4 variation(s)…"
5. On completion: Variation 1 placeholder is replaced with the AI-generated caption; 4 new variations appended with AI badge

### 7.3 Bulk Generate (All Posts)

1. User clicks `✨ Generate variations for all` above the post list
2. Bulk modal opens; shows warning notice about existing variations and image-only posts; shows `{N} posts × {V} = {total} variations` dynamic label
3. User confirms settings and clicks `✨ Generate for All`
4. Posts are processed (parallel where infrastructure allows) — each shows its own loading card during generation
5. Completion toast: "{Total} variations generated across all posts."

### 7.4 Zero Credits

1. User clicks `✨ Generate with AI`
2. Modal opens; count stepper and image toggle are hidden; amber alert shown: "You've run out of AI text credits. Variations can't be generated right now. [Upgrade your plan] to get more credits."
3. Generate button is disabled (opacity 45%)

### 7.5 Background Generation — User Navigates Away

1. User triggers generation and then clicks Next or Previous
2. Non-blocking warning banner appears at top: "Variations are still being generated. You can leave — your results will be saved automatically and will be ready when you return."
3. User can proceed with navigation freely; generation is not cancelled
4. When user returns to Step 2, variations are already populated

---

## 8. Functional Specification

### 8.1 Compose Posts Step — Layout Changes

**Top-level button** (above post list, right-aligned):
- Label: `✨ Generate variations for all`
- Style: outlined with primary-cs border, white background

**Per-post card header** (existing row extended):

| Element | Details |
|---|---|
| Post name | e.g., "Post 1" |
| Variation count badge | `{N} Variation` / `{N} Variations` — indigo pill (primary-cs-500 bg), live-updating |
| `+ Add variation` | Existing button — unchanged |
| `✨ Generate with AI` | New button — AI gradient style; always shown regardless of post content type |
| 🗑 Delete post | Existing icon button — unchanged |

**Variation cards** (existing on-hover edit/delete unchanged):
- AI-generated variations: identical card styling to manual variations + small AI badge in top-right of card header
- AI badge: pill with "AI" label, primary-cs gradient, small size

### 8.2 Generate Variations Modal (Per Post)

**Trigger:** `✨ Generate with AI` button on any post card

| Element | Details |
|---|---|
| Modal title | `Generate Variations` |
| Modal subtitle | `Post {N}` |
| Source field | Auto-detected chip (read-only): `🖼 Image (caption will be generated first)` / `💬 Post caption` / `🔗 Link content + post caption` |
| Source hint text | See §8.5 |
| Count stepper | −/+ with number input, min 1, max 10, default 3 |
| Count hint | "Max 10 per post. Each variation consumes AI text credits." |
| Image toggle | Off by default; sub-text shows remaining image credits |
| Image toggle (credits exhausted) | Disabled at 50% opacity; sub-text: "You've used all your AI image credits. [Upgrade your plan] to get more." |
| Custom instructions | Textarea, optional; placeholder: "e.g. Keep under 280 chars, use 2 hashtags, match brand tone…" |
| Cancel button | `Cancel` — secondary style |
| Generate button | `✨ Generate` — AI gradient style |

**Zero text credits state:**
- Count stepper row hidden
- Image toggle row hidden
- Amber alert: "You've run out of AI text credits. Variations can't be generated right now. [Upgrade your plan] to get more credits."
- Generate button disabled (opacity 45%)

### 8.3 Bulk Generate Variations Modal

**Trigger:** `✨ Generate variations for all`

| Element | Details |
|---|---|
| Modal title | `Generate Variations for All Posts` |
| Modal subtitle | `Settings apply to every post in this automation` |
| Warning notice | Always shown (amber/yellow block): lists posts that already have variations (additive, not replaced) and image-only posts (caption will be generated first) |
| Variations per post stepper | Same as per-post modal |
| Dynamic label | `{N} posts × {V} = {total} variations` — updates live as count changes |
| Image toggle | Same as per-post modal |
| Custom instructions | Same textarea |
| Cancel / Generate buttons | Cancel + `✨ Generate for All` |
| Zero credits state | Same as per-post: controls hidden, alert shown, button disabled |

### 8.4 Loading Card

Appended to the bottom of the relevant post card during generation:

- Border: dashed, `border-primary-cs-200`
- Background: `bg-primary-cs-50`
- Content (image-only post): "Analysing image & generating caption first, then creating {N} variation(s)…"
- Content (post with caption): "Generating {N} variation(s)…"
- Icon: spinning loader
- Removed when generation completes; replaced by new variation cards

### 8.5 Source Auto-Detection (Modal)

| Post type | Source chip | Hint text |
|---|---|---|
| Image only (no caption) | `🖼 Image (caption will be generated first)` | "This post has no caption yet. AI will analyse the image, generate a caption, then create variations from it." |
| Caption only | `💬 Post caption` | "AI will use the existing post caption as the base for all variations." |
| Caption + link | `🔗 Link content + post caption` | "AI will use both the linked article and the existing caption as context for generating variations." |

### 8.6 Variation Card — AI Badge

Generated variations appear with identical card styling to manual variations. The only differentiator is a small `AI` pill badge in the card header (top right, before the edit/delete action icons). Badge uses primary-cs gradient.

**Badge lifecycle:** The badge is removed automatically when the user edits the variation's caption text. It cannot be manually added or removed — it reflects whether the content is still as the AI wrote it. Once the user modifies the text, the content is theirs, and the badge disappears.

### 8.7 Navigation Warning Banner

Shown non-intrusively at the top of the Step 2 page when generation is in progress and the user attempts to navigate:

> "Variations are still being generated. You can leave — your results will be saved automatically and will be ready when you return."

- Not a blocking dialog — user can navigate freely
- Dismisses automatically when generation completes or when user navigates away

### 8.8 Background Generation & Session Interruption

| Scenario | Behaviour |
|---|---|
| User closes modal during generation | Job continues on backend; variations auto-saved to draft on completion; appear on next page load or via polling |
| User navigates away from Step 2 | Non-blocking banner shown; job unaffected; variations ready when user returns |
| Browser tab closed / crash | Backend job completes; draft auto-saved; user sees completed variations when reopening wizard |
| User returns after job completes | Variations appear as normal cards with AI badge; no pending state; count badge updated |
| Partial generation (credits run out mid-batch) | Completed variations saved; incomplete skipped; partial toast shown |
| Bulk: tab closed mid-batch | Completed variations from fully-processed posts saved; partially-processed post saves whatever it completed; user sees results on return and can trigger more manually |

---

## 9. Credit System

| Credit type | Unit | When deducted |
|---|---|---|
| AI text credits | Per ~60 words generated | After each variation is successfully generated |
| AI image credits | 1 per image | After each image variation is generated (if toggle on) |

**Rules:**
- Credits validated before generation starts; if 0 text credits → block with upgrade prompt
- Credits deducted after generation (not before) based on actual variations completed, not requested
- If credits exhausted mid-generation: partial results saved, partial toast shown, remainder skipped
- Credit count shown in modals only (not on the main page)
- Image toggle disabled if image credits = 0

---

## 10. Toast Notifications

| Trigger | Message | Style | Duration |
|---|---|---|---|
| Single post generation success | `{N} variation(s) generated successfully.` | success (green) | 4s |
| Single post — partial credits | `Only {X} of {N} variations generated — you ran out of AI text credits mid-way. Upgrade your plan to get more.` | error (red) | 7s |
| Single post — zero credits, none generated | `Not enough AI text credits to generate any variations. Please upgrade your plan.` | error (red) | 7s |
| Bulk generation success | `{Total} variations generated across all posts.` | success (green) | 4s |
| Bulk generation — partial | `{X} variations generated across all posts. {Y} couldn't be created — AI text credits ran out mid-way. Upgrade to get more.` | warn (orange) | 8s |
| Bulk generation — zero credits | `No variations could be generated — you've run out of AI text credits. Please upgrade your plan.` | error (red) | 7s |
| Variation deleted | `Variation deleted.` | warn (orange) | 4s |
| Post deleted | `Post deleted.` | warn (orange) | 4s |

---

## 11. Business Rules & Constraints

| Rule ID | Rule | Rationale |
|---|---|---|
| BR-1 | Max 10 variations per post (stepper clamps at 10) | Prevents runaway credit usage; keeps UI manageable |
| BR-2 | Generation is additive — existing variations are never replaced | Protects user's existing work |
| BR-3 | Bulk processing: parallel where infrastructure allows; engineering decides strategy | Each post shows its own loading card regardless of parallelism |
| BR-4 | Credits deducted after generation (not upfront) | Users pay only for what they get |
| BR-5 | Image-only posts: AI generates caption first, then variations | Ensures all variations have text; caption saved to Variation 1 |
| BR-6 | AI badge is removed when the user edits the variation's text; it persists if the variation is untouched | Once a user modifies the content, it's no longer purely AI-generated; badge removal signals ownership |
| BR-7 | AI features are web-only — no mobile equivalent | Aligned with ContentStudio's AI product strategy |
| BR-8 | No Keep/Discard approval flow — variations pushed directly | Simpler UX; users edit/delete if they don't like it |

---

## 12. Open Questions

| Question | Options | Owner | Decision |
|---|---|---|---|
| What AI model/prompt drives caption + variation generation? | GPT-4o / Claude / existing AI pipeline | Engineering / AI team | To decide |
| Polling vs. websocket for real-time variation appearance? | Polling (simpler) / Websocket (better UX) | Engineering | To decide (polling acceptable for v1) |
| Should the AI badge be removable by the user? | Yes / No / Auto-remove on edit | Product | **Decided:** Auto-removed when user edits the variation text; not manually removable |
| Max concurrent generation jobs per workspace? | 1 / 3 / unlimited | Engineering | To decide based on infra capacity |

---

## 13. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Poor AI output quality for niche brands | Medium | Medium | Custom instructions field lets users guide tone; easy edit/delete if bad |
| Credits consumed but browser crashes = user loses context | Low | Low | Draft auto-save on job completion; user finds results on return |
| Bulk generation hammers AI pipeline | Medium | Medium | Sequential processing (not parallel) limits load |
| Image-only posts: AI misreads image, bad caption generated | Low | Medium | User can edit Variation 1 in full composer; expected behaviour for v1 |

---

## 14. Dependencies

- **AI text/image credit system** — existing credits API and billing integration
- **Evergreen automation draft save** — existing `saveDraftEvergreenAutomation(step)` action
- **Backend async job infrastructure** — existing job queue (assumed available based on current URL variation fetching pattern)
- **WebSocket or polling mechanism** — for real-time variation appearance (polling acceptable for v1)
- **AI caption generation service** — must support image-to-caption (same service used in Composer)

---

## 15. Appendix

- **Functional spec (full):** User-provided spec document (sections 1–17) — attached to Shortcut epic as a doc
- **Research doc:** `docs/stories/evergreen-ai-variations/01-research.md`
- **Codebase context:** `contentstudio-frontend/src/modules/automation/components/evergreen/create/`
- **Shortcut config:** `.claude/shortcut-config.json`

---

## Changelog

| Date | Author | Changes |
|---|---|---|
| 2026-03-03 | Product (Claude) | Initial PRD from user-provided spec + codebase research |
| 2026-03-03 | Product | Simplified variation flow — removed Keep/Discard/Try Again; variations pushed directly with AI badge |
