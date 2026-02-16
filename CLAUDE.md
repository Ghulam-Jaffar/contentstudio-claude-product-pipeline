# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A **Claude Code-powered product development pipeline** for [ContentStudio](https://contentstudio.io), a social media management platform. It automates the workflow from feature idea → research → PRD → Shortcut stories, with review gates at every step.

This is **not** a code project. There's no package.json or composer.json at root. The `contentstudio-backend/` and `contentstudio-frontend/` directories are **gitignored separate repos** mounted here so the pipeline can analyze the actual codebase when writing stories.

## Two Pipeline Commands

### `/feature` — Full Feature Pipeline (5 steps)
For major features requiring PRDs and dedicated epics.

**Research → Workflow Design → PRD → Epic + Stories → Push to Shortcut**

- Runs parallel competitor research (WebSearch) + codebase analysis (Explore agent) in Step 1
- Creates a dedicated Shortcut epic for the feature
- Outputs saved to `docs/features/<slug>/` (01-research.md through 05-shortcut-links.md)
- Review gate after every step — never proceed without explicit user approval

### `/story` — Quick Story Pipeline (3 steps)
For small improvements that don't need a full PRD. Max 4 stories; if 5+, redirect to `/feature`.

**Research → Stories → Push to Shortcut**

- Lean codebase research using direct Grep/Read (not Explore agents)
- Stories link to the miscellaneous quarterly epic (currently Q1 2026, id: 107163)
- Outputs saved to `docs/stories/<slug>/` (01-research.md through 03-shortcut-links.md)

## Key Files

| File | Purpose |
|---|---|
| `.claude/shortcut-config.json` | All Shortcut API IDs: workflows, states, groups, custom fields, projects, epics, competitors |
| `.claude/commands/feature.md` | `/feature` pipeline definition |
| `.claude/commands/story.md` | `/story` pipeline definition |
| `docs/story-guidelines.md` | **Mandatory** 15-section rulebook — read before writing any story |
| `docs/PRD Feature Template.md` | 12-section PRD template used by `/feature` Step 3 |
| `docs/Shortcut story template.md` | Story body structure (Description, Workflow, AC, Mock-ups, Impact, Dependencies, Quality checklist) |

## Story Rules (Summary)

The full rules are in `docs/story-guidelines.md`. Key points:

- **Always use the "New Feature Template"** (under no team) when creating stories — pass `story_template_id` in the API payload
- **Titles:** `[BE]` / `[FE]` / `[iOS]` / `[Android]` / `[Design]` prefix + action-oriented title
- **Workflow sections:** Written from user's POV, never developer POV
- **FE stories must include all UI copy:** modal titles, labels, tooltips, placeholders, validation errors, empty/error/loading states — written for non-technical users with concrete examples
- **No estimates, no labels** — devs handle these during sprint planning
- **Always assign a Shortcut project** (Web App, Mobile, Chrome App, etc.)
- **No dark mode, no RTL** — ContentStudio doesn't support either
- **AI features are web-only** — no mobile AI stories
- **Color theming:** Use `text-primary-cs-500`, `bg-primary-cs-50`, etc. (CSS variable-backed) — never hardcode colors like `text-blue-600`
- **Reference stories by full title**, never by number
- **Create separate iOS/Android stories** when mobile apps are impacted

## Shortcut API Integration

- API base: `https://api.app.shortcut.com/api/v3`
- Auth token: `SHORTCUT_API_TOKEN` env var (loaded from `.env`)
- Workspace: `contentstudio-team`
- **Windows curl requirement:** Write JSON payloads to a temp file, use `curl --data @file`
- Stories default to `ready_for_dev` workflow state (id: 500000070)
- Epics default to `to_do` state (id: 500000002)

## ContentStudio Product Context

The pipeline analyzes and writes stories for these codebases (mounted but gitignored):

- **`contentstudio-backend/`** — Laravel 10 API (PHP 8.3, MongoDB, Redis, Kafka). Has its own `CLAUDE.md` with project-specific rules.
- **`contentstudio-frontend/`** — Vue 3 SPA (Composition API, Vuex → Pinia). Has docs in `contentstudio-frontend/docs/`.

When the pipeline does codebase analysis, it searches these directories for relevant models, controllers, services, components, routes, and composables to ground stories in the actual implementation.

## Branch & PR Conventions (for code implementation)

When implementing stories in the sub-project codebases:

- **Branch from:** `develop`
- **Branch naming:** `feature/sc-{story-id}/{story-title-slug}`
- **Commit format:** `[sc-{id}] {description}`
- **PR base:** `develop`
- Shortcut auto-links PRs via the `sc-XXXXX` in the branch name
