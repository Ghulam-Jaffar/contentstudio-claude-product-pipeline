# Shortcut Links — Composer v2 Module Refactor

## Epic
- [Composer v2 Module Refactor](https://app.shortcut.com/contentstudio-team/epic/111468)

## Docs (linked to epic)
- [Composer v2 Refactor — Research](https://app.shortcut.com/contentstudio-team/doc/6994094a-9e6f-49ce-9ae5-0a91ba3ce98a) — Architecture analysis, problem inventory, refactoring best practices
- [Composer v2 Refactor — PRD](https://app.shortcut.com/contentstudio-team/doc/6994094b-5989-4b88-ab85-c69d63d041a7) — Product Requirements Document

## Prerequisite (Existing — Different Epic)
- [Move post creation validations to backend for all creation sources](https://app.shortcut.com/contentstudio-team/story/110785) — sc-110785 (epic 107951, queued)

## Stories (Sequential Phases)

| Phase | Story | Link |
|---|---|---|
| 1 | [FE] Create Pinia composer store and domain composables | [sc-111469](https://app.shortcut.com/contentstudio-team/story/111469) |
| 1 | [FE] Replace EventBus with typed events and store actions | [sc-111475](https://app.shortcut.com/contentstudio-team/story/111475) |
| 2 | [FE] Decompose SocialModal into thin layout shell | [sc-111481](https://app.shortcut.com/contentstudio-team/story/111481) |
| 3 | [FE] Decompose EditorBox into independent reusable components | [sc-111487](https://app.shortcut.com/contentstudio-team/story/111487) |
| 4 | [FE] Extract PostSchedulePicker and platform options | [sc-111493](https://app.shortcut.com/contentstudio-team/story/111493) |
| 4 | [FE] Remove frontend validation logic and wire backend errors | [sc-111499](https://app.shortcut.com/contentstudio-team/story/111499) |
| 5 | [FE] Cleanup legacy code — finalize migration | [sc-111505](https://app.shortcut.com/contentstudio-team/story/111505) |

## Details
- **Iteration:** 23 Feb - 06 March - 2026
- **Priority:** High
- **Product Area:** Composer
- **Story Type:** Chore (technical refactoring)
