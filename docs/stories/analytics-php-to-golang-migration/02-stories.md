# Stories: Analytics APIs — PHP to Golang Migration

## Epic

**Title:** Analytics APIs — PHP to Golang Migration

**Description:**
Migrate all analytics APIs from the PHP/Laravel monolith to Golang for improved performance, memory efficiency, and independent scalability. The current PHP analytics surface includes 154 endpoints across 24 controllers covering 8 social platforms, cross-platform overview, AI insights, reports, dashboard, campaign/label analytics, and share links. This epic tracks the migration effort across all phases.

---

## Story 1: [BE] Migrate analytics APIs from PHP to Golang

### Description:

Track the migration of ContentStudio's analytics APIs from the Laravel PHP monolith (`contentstudio-backend`) to a Golang service. The PHP analytics module currently contains **154 POST endpoints** across **24 controller files** covering Facebook, Instagram, LinkedIn, YouTube, TikTok, Pinterest, Twitter/X analytics, cross-platform overview, AI insights, reports, dashboard, campaign/label analytics, and share links.

See the attached research document ("Analytics PHP to Golang Migration — Scope & Inventory") for the full endpoint inventory, controller mapping, data layer details (MongoDB, Elasticsearch, Redis/Horizon jobs), and auth/middleware patterns that the Go service must replicate.

**PHP side reference files:**
- Routes: `contentstudio-backend/routes/web/analytics.php` (154 endpoints)
- Controllers: `contentstudio-backend/app/Http/Controllers/Analytics/` (24 files)
- Auth: `auth`, `set.locale`, `shareable.optional`, `shareable.auth`, `PermissionMiddleware`
- Data: MongoDB collections, Elasticsearch indices, Laravel Horizon queue jobs

---

### Workflow:

1. Development team reviews the attached scope & inventory document
2. Team identifies the platform to migrate first as a proof of concept
3. Go service is built with equivalent endpoints, same request/response shapes, same auth model
4. Frontend is updated to point to the new Go endpoints per platform as each is migrated
5. PHP analytics endpoints are deprecated and removed once the Go equivalent is verified in production

---

### Acceptance criteria:

- [ ] All 154 analytics endpoints have a Golang equivalent (or are explicitly descoped)
- [ ] Each Go endpoint accepts the same request payload as its PHP counterpart (or has documented frontend changes)
- [ ] Each Go endpoint returns the same response shape as its PHP counterpart (or has documented frontend changes)
- [ ] Auth/permission model is replicated (session auth, API key auth, share link auth with optional password)
- [ ] MongoDB and Elasticsearch queries produce identical results to the PHP versions
- [ ] Frontend is updated to call Go endpoints for each migrated platform
- [ ] PHP analytics endpoints are removed after migration is verified per platform
- [ ] No regression in analytics data accuracy or page load performance

---

### Mock-ups:

N/A — backend migration, no UI changes (frontend points to new endpoints but the UI stays the same).

---

### Impact on existing data:

None. The Go service reads from the same MongoDB collections and Elasticsearch indices. No data migration needed.

---

### Impact on other products:

- **Web app:** Frontend needs to update API endpoint URLs per platform as each is migrated
- **Mobile apps:** If mobile apps call analytics APIs directly, they will need endpoint URL updates
- **Chrome extension:** No impact (does not use analytics APIs)
- **White-label:** Analytics share links must continue working with white-label domains

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend migration
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend migration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
