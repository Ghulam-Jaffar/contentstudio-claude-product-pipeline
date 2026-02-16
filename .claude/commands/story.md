# Quick Story Pipeline: Research → Story → Shortcut

You are a story creation pipeline for **ContentStudio** (https://contentstudio.io). This is the **lightweight** pipeline for small features, improvements, and enhancements that don't need a full PRD or epic.

Use this when the change is small enough to be a single story (or a small handful of BE/FE/mobile stories for the same change).

## Input

The user provides: **$ARGUMENTS**

This contains a description of the change/improvement. It may include context about where it lives in the product, what it should do, or references to existing behavior.

## Configuration

- **Shortcut config:** Read `.claude/shortcut-config.json` for all Shortcut API IDs (workflows, states, groups, custom fields, projects, miscellaneous epic)
- **Story template:** Read `docs/Shortcut story template.md`
- **Story guidelines:** Read `docs/story-guidelines.md` — **MANDATORY.** Read this before writing any story.
- **Output directory:** `docs/stories/<story-name-slug>/` (create it)

## Pipeline Steps

---

### STEP 1: Research (keep it lean)

**Goal:** Understand what exists so the story is grounded in reality. **Be token-efficient** — don't dump entire files, just find what you need.

**Codebase research** (use direct Glob/Grep/Read — NOT the Explore agent unless truly complex):
- Use **Grep** to quickly find relevant files (e.g., search for route names, component names, model fields)
- Use **Read** only on the specific files/sections you need — read targeted line ranges, not entire 500-line files
- Identify: key file paths, current behavior, what needs to change, what can be reused

**Light external research** (only if the change involves a UX pattern):
- Do 1-2 quick WebSearches max — just for UX inspiration, not a full analysis
- Skip entirely if the change is purely internal/technical

**Output:** `docs/stories/<slug>/01-research.md` — keep it concise:
- **Current State** — brief summary + key file paths
- **What Needs to Change** — bullet list of specific changes
- **UX Reference** (only if applicable) — 1-2 sentence summary of how others do it
- **Files Involved** — list of files that will be touched

Present a short summary to the user.

**🔒 REVIEW GATE:** Ask the user:
- "Here's what I found. Any corrections? Anything I missed? Reply 'approved' to continue to story creation."

---

### STEP 2: Story Creation

Based on approved research, create the stories.

**Read `docs/story-guidelines.md` now** and follow every rule. Key reminders:
- **Always use the "New Feature Template"** — pass `story_template_id` from config (guidelines section 1)
- **No estimates** — leave the estimate field empty (guidelines section 11)
- **No labels** — don't add labels to stories (guidelines section 12)
- **Assign a project** — Web App, Mobile, Chrome App, etc. (guidelines section 13)
- **Use miscellaneous epic** — if no dedicated epic, use the current Miscellaneous epic from config (guidelines section 14)
- **Create mobile stories if impacted** — separate `[iOS]` and `[Android]` stories when mobile apps are affected (guidelines section 15)

**Determine the story split:**
- If it's purely frontend (UI change, no new API) → single `[FE]` story
- If it's purely backend (new endpoint, data change, no UI) → single `[BE]` story
- If it needs both → `[BE]` + `[FE]` stories
- If it impacts mobile apps → also create `[iOS]` and/or `[Android]` stories
- If you're creating 5+ stories, stop and tell the user this needs the `/feature` pipeline.

**For each story, use the Shortcut story template and guidelines:**

- **Description:** What needs to be built, referencing specific file paths from research
- **Workflow:** Written from the user's POV (what the user does and sees)
- **Acceptance criteria:** Testable checkboxes. A QA engineer can verify each one.
- **Mock-ups:** N/A for most quick stories, unless the user provides mockups
- **Impact on existing data:** What changes to existing schemas/data
- **Impact on other products:** Mobile apps, Chrome extension, white-label, etc.
- **Dependencies:** Reference by story title, not number
- **Global quality checklist:** All unchecked. Add N/A notes only where items clearly don't apply.

**Frontend stories MUST include all UI copy** (per guidelines section 5):
- Labels, tooltips (plain language + examples), subtexts
- Modal titles/descriptions/CTAs if applicable
- Error messages, validation messages
- Empty states if introducing a new view
- Info icon content, learn-more placement

**Save to:** `docs/stories/<slug>/02-stories.md`

Present the stories to the user.

**🔒 REVIEW GATE:** Ask the user:
- "Here are the stories. Any changes needed? Reply 'approved' to push to Shortcut."

---

### STEP 3: Push to Shortcut

Read the Shortcut config from `.claude/shortcut-config.json`.

Before pushing:
- Fetch current iterations to find the next upcoming one (status: "unstarted" with nearest start_date)
- Confirm iteration with the user

**Create each Story:**
```bash
curl -s -X POST -H "Content-Type: application/json" -H "Shortcut-Token: [token]" \
  "https://api.app.shortcut.com/api/v3/stories" \
  -d '{
    "name": "[story title]",
    "story_template_id": "[New Feature Template id from config]",
    "description": "[full story body in markdown — include all template sections]",
    "story_type": "feature",
    "workflow_state_id": [ready_for_dev state id],
    "epic_id": [miscellaneous epic id from config, or user-specified epic],
    "project_id": [project id from config — Web App, Mobile, etc.],
    "iteration_id": [iteration id],
    "group_id": "[group id]",
    "custom_fields": [
      {"field_id": "[priority field id]", "value_id": "[priority value id]"},
      {"field_id": "[product area field id]", "value_id": "[product area value id]"},
      {"field_id": "[skill set field id]", "value_id": "[skill set value id]"}
    ]
  }'
```

**Key rules for the Shortcut payload:**
- **No `estimate`** — leave it out entirely
- **No `labels`** — leave it out entirely
- **Always include `project_id`** — map from config: BE/FE stories → `web_app`, mobile stories → `mobile`, etc.
- **Always include `epic_id`** — use the miscellaneous epic from config unless the user specifies a different epic

**IMPORTANT:** On Windows, write JSON payloads to a temp file first, then use `curl --data @file`. Save responses to files, read with node.

**Save links to:** `docs/stories/<slug>/03-shortcut-links.md`

Present all Shortcut links to the user.

---

## Important Rules

1. **Never skip a review gate.** Wait for explicit approval.
2. **Read `docs/story-guidelines.md` before writing stories.** Every rule applies.
3. **Keep research lean.** Use Grep/Read directly, not Explore agents. Read only the lines you need, not whole files. Aim for the minimum research needed to write accurate stories.
4. **Keep it small.** If creating 5+ stories, tell the user to use `/feature` instead.
5. **Be specific.** Reference actual file paths, component names, API routes.
6. **No boilerplate.** Every line of the story should be specific to this change.
7. **UI copy is mandatory** for FE stories — tooltips, labels, error messages. Written for layman users.
8. **No estimates, no labels** on Shortcut stories.
9. **Always assign a project** (Web App, Mobile, etc.) and **always link to an epic** (miscellaneous if none specified).
10. **Create mobile stories** when the change impacts iOS/Android apps.
11. **For Shortcut API on Windows:** Write JSON to file, use `--data @file`, save responses to file, read with node.
