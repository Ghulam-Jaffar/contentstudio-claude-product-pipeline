# **PRD: Composer v2 Refactor**

**Author:** Product Owner
**Last Updated:** 2026-02-17
**Status:** In Review
**Target Release:** Q1–Q2 2026

---

## **1. Overview**

Refactor the ContentStudio Composer v2 module from a monolithic, tightly-coupled system (~51,000 lines, 119 files) into a modular, maintainable architecture with independent, reusable components. The composer is the core content creation tool — every post published through ContentStudio passes through it. Its current architecture makes feature development slow, bug-prone, and impossible to reuse components elsewhere. This refactor establishes a clean state layer (Pinia + provide/inject), decomposes god components into focused units, eliminates prop drilling (80+ props → 0), replaces the EventBus anti-pattern, and moves validations to the backend. No user-facing UI changes — same behavior, new internals.

---

## **2. Problem Statement**

**What problem are we solving?**

The Composer v2 module has accumulated severe architectural debt:
- **SocialModal.vue is 7,110 lines** with 21+ responsibilities (god component)
- **80+ props drilled** through MainComposer, 50+ through EditorBox, 5 levels deep
- **1,576-line utility composable** (`useComposerHelper`) doing 10+ unrelated things
- **48 files** using the EventBus anti-pattern with no type safety
- **Validation logic scattered** across components with no shared schema
- **Zero reusable components** — editor, schedule picker, account selector cannot work independently

**Who has this problem?**

- Frontend developers: every composer change takes 3-5x longer than it should due to prop chain tracing and side-effect debugging
- Product team: feature requests for the composer are consistently under-scoped because complexity is hidden
- QA: testing is unreliable because state flows through 5 levels of props and EventBus events

**What happens if we don't solve it?**

- New composer features continue to take disproportionate dev time
- Bug rate increases as complexity compounds
- Components cannot be reused in automation, bulk scheduling, or other modules
- Developer onboarding for composer work is weeks instead of days
- Technical debt compounds — each new feature adds to the prop chains and composable spaghetti

---

## **3. Goals & Success Metrics**

| Goal | Metric | Target | How We'll Measure |
|---|---|---|---|
| Reduce component complexity | Max props per component | <15 props (from 80+) | Code review / static analysis |
| Enable reusability | Components usable outside composer | Editor, Schedule, AccountSelector each work standalone | Integration test in isolation |
| Improve maintainability | SocialModal.vue line count | <500 lines (from 7,110) | `wc -l` |
| Eliminate EventBus | Files using EventBus in composer_v2 | 0 (from 48) | `grep -r EventBus` |
| Reduce composable bloat | Largest composable file | <300 lines (from 1,576) | `wc -l` |
| No regressions | Existing E2E tests pass | 100% pass rate | CI/CD pipeline |
| Backend validation | Frontend validation code removed | 90%+ of scattered validation logic removed | Code diff |

---

## **4. Target Users**

**Primary Persona:**
Frontend Developer — Works on the composer module daily. Currently spends significant time tracing prop chains, debugging EventBus events, and understanding state flow. Needs clean architecture with clear boundaries to ship features faster.

**Secondary Persona:**
Product Owner / QA Engineer — Needs confidence that composer changes don't cause regressions. Wants components that can be tested in isolation and reused in other modules.

**Non-Users (explicitly out of scope):**
- End users of ContentStudio — this is an internal refactor with zero UI changes
- Backend developers — backend changes are limited to the existing validation story (sc-110785)
- Mobile app developers — mobile has its own composer, not affected

---

## **5. User Stories / Jobs to Be Done**

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| US-1 | Frontend dev | Have a single source of truth for composer state | I don't trace 5 levels of props to find where data lives | Must Have |
| US-2 | Frontend dev | Use the editor text box as a standalone component | I can embed it in automation, bulk scheduling, or other features | Must Have |
| US-3 | Frontend dev | Use the schedule picker independently | I can add scheduling UI to any feature without duplicating code | Must Have |
| US-4 | Frontend dev | Use the account selector independently | I can add account selection to any new feature | Must Have |
| US-5 | Frontend dev | Have focused composables (<300 lines each) | I can understand and modify behavior without reading 1,576 lines | Must Have |
| US-6 | Frontend dev | Not use EventBus for composer communication | I have type safety, no memory leaks, and traceable data flow | Must Have |
| US-7 | QA engineer | Test composer components in isolation | I can verify each component without mocking 80+ props | Should Have |
| US-8 | Frontend dev | Have backend validate all post creation rules | I don't maintain duplicate validation logic in the frontend | Must Have |
| US-9 | Product owner | Add new platform support with minimal code changes | Adding a new social platform doesn't require touching 20+ files | Should Have |

---

## **6. Requirements**

### **6.1 Must Have (P0)**

* **Backend validation migration** — All post creation validations centralized in backend with structured error responses. Frontend reduced to UX-only hints. (sc-110785)
* **Pinia composer store** — Single source of truth for all composer state: post data, account selection, scheduling, UI state, validation errors, loading flags
* **Domain composables** — Replace `useComposerHelper` (1,576 lines) with 8+ focused composables: `useComposerPost()`, `useAccountSelection()`, `usePostSchedule()`, `useComposerMedia()`, `useComposerValidation()`, `useComposerUI()`, `usePlatformMetadata()`, `useMobileDetection()`
* **SocialModal decomposition** — Reduce from 7,110 lines to <500 lines (thin layout shell). Extract AccountSelectionPanel, ComposerMain, ComposerSidebarPanel as independent components using inject/store
* **Editor decomposition** — Break EditorBox (2,873 lines) into 5 independent components: EditorTextBox, EditorMediaManager, EditorAIActions, EditorToolbar, EditorLinkPreview
* **Prop drilling elimination** — All components use inject/store. No component receives >15 props
* **EventBus replacement** — Replace all 48 EventBus usages in composer_v2 with store actions + `mitt` typed events

### **6.2 Should Have (P1)**

* **PostSchedulePicker extraction** — Standalone scheduling component usable outside composer
* **Platform options simplification** — Each platform option component self-sufficient via store, no props from parent
* **Composable cleanup** — Delete `useComposerHelper`, consolidate remaining composables
* **Entry point updates** — Home.vue, planner routes, publisher routes updated with clean imports

### **6.3 Nice to Have (P2)**

* **Typed event map** for `mitt` — Full TypeScript-style event definitions
* **Component documentation** — JSDoc/README for each extracted component's API
* **Storybook stories** for reusable components (EditorTextBox, PostSchedulePicker, AccountSelector)

### **6.4 Explicitly Out of Scope**

* **UI/UX changes** — Same visual design, same user flows, same behavior
* **Post preview system rebuild** — 17 preview components have their own debt, deferred to v2
* **TypeScript migration** — Deferred to v2
* **Unit test coverage** — Deferred to v2 (though refactored architecture will be more testable)
* **Performance optimization** — Deferred to v2 (virtual scrolling, lazy loading)
* **New features** — No new user-facing features in this epic
* **Mobile app changes** — Mobile has its own composer
* **Backend changes** (beyond sc-110785) — No new API endpoints needed

---

## **7. User Flow (High Level)**

Since this is an internal refactor, the "user flow" is the developer experience:

### Before Refactor (Current)
1. Developer needs to add a new field to the composer
2. Developer adds prop to SocialModal (level 1)
3. Developer threads prop through MainComposer (level 2) — now 81 props
4. Developer threads prop through EditorBox (level 3) — now 51 props
5. Developer adds logic to one of the god components
6. Developer tests — side effects break unrelated features via EventBus
7. Developer spends 2-3x estimated time debugging prop chains

### After Refactor (Target)
1. Developer needs to add a new field to the composer
2. Developer adds field to the Pinia composer store schema
3. Developer updates the relevant domain composable
4. Developer's component uses `inject()` — no prop changes to any parent
5. Developer tests the component in isolation
6. Change is contained — no side effects

### End User Flow (Unchanged)
1. User opens Composer from header / planner / publisher
2. User selects accounts, writes content, adds media, sets schedule
3. User publishes / schedules / saves draft / sends for approval
4. Everything works exactly the same as before

---

## **8. Business Rules & Constraints**

| Rule ID | Rule | Rationale |
|---|---|---|
| BR-1 | Zero UI changes — every pixel, every interaction must remain identical | This is a refactor, not a redesign. No user impact. |
| BR-2 | Each phase must be independently shippable and testable | No big-bang rewrite. Each PR can be merged and deployed. |
| BR-3 | Backend validation (sc-110785) must complete before frontend validation removal | Cannot remove frontend validation before backend enforces rules |
| BR-4 | EventBus removal must be file-by-file with verification | Removing all at once risks breaking cross-module communication |
| BR-5 | Pinia store must support edit mode (loading existing post) and create mode | Store initialization differs based on entry point |
| BR-6 | Extracted components must work with inject/provide pattern | Ensures they work standalone when provided proper context |
| BR-7 | Existing E2E tests must pass after each phase | Regression prevention |

---

## **9. Open Questions**

| Question | Options | Owner | Due Date | Decision |
|---|---|---|---|---|
| Should we create a formal Storybook for extracted components? | Yes / No / Defer to v2 | Frontend Lead | Before Phase 3 | Pending |
| Should we add TypeScript to new composables even though module isn't TS yet? | Yes (gradual) / No (consistent) | Frontend Lead | Before Phase 1 | Pending |
| What E2E test coverage exists today for the composer? | Need audit | QA Lead | Before Phase 1 | Pending — this determines regression test baseline |

---

## **10. Risks & Mitigations**

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Regressions in post creation flow during migration | High | High | Phase each change independently. Run E2E tests after every PR. Keep old and new code paths working during transition. |
| EventBus removal breaks cross-module communication (planner → composer) | Medium | High | Audit all EventBus events before removing. Replace with mitt typed events one at a time. Verify each replacement. |
| State migration causes data loss (drafts, in-progress posts) | Medium | High | Pinia store hydration tested against all entry points (new post, edit post, template, automation). Verify draft auto-save still works. |
| Scope creep — developers want to "fix UI while we're at it" | High | Medium | Strict rule: BR-1 — zero UI changes. PRs that change any user-facing behavior are rejected. |
| Backend validation story (sc-110785) delayed, blocking Phase 1 frontend cleanup | Medium | Medium | Frontend validation removal is the last step of Phase 1, not the first. State layer and composables can proceed without it. |
| Developer resistance to new patterns | Low | Medium | Document the architecture decisions. Pair program first few component migrations. Benefits become obvious after first extraction. |

---

## **11. Dependencies**

* **Internal:**
  - **sc-110785** (Move post creation validations to backend) — must complete before frontend validation logic removal
  - Existing Vuex store (`@state/base`) — must remain functional during migration, gradually replaced by Pinia
  - Existing composables in `src/composables/` (global) — `useComposerPost`, `useComposerHelpers`, `useComposerMediaUpload`, `useComposerLinkPreview` — need to be audited and either migrated or wrapped
  - E2E test suite — needs baseline run before Phase 1 begins

* **External:**
  - `pinia` — Already in project dependencies (used by other modules)
  - `mitt` — Lightweight event emitter (~200 bytes). New dependency, minimal risk.

* **Blockers:**
  - None — Pinia is already available, mitt is trivial to add. sc-110785 blocks only the validation removal step, not the overall refactor.

---

## **12. Appendix**

* Research document: `docs/features/composer-v2-refactor/01-research.md`
* Workflow design: `docs/features/composer-v2-refactor/02-workflow.md`
* Existing backend validation story: [sc-110785](https://app.shortcut.com/contentstudio-team/story/110785)
* Vue 3 provide/inject docs: https://vuejs.org/guide/components/provide-inject
* Vue 3 composables guide: https://vuejs.org/guide/reusability/composables.html
* mitt event emitter: https://github.com/developit/mitt
* Current composer module: `contentstudio-frontend/src/modules/composer_v2/`
* SocialModal entry point: `contentstudio-frontend/src/modules/composer_v2/views/SocialModal.vue`

---

## **Changelog**

| Date | Author | Changes |
|---|---|---|
| 2026-02-17 | Product Owner | Initial draft |
