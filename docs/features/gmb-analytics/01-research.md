# GMB Analytics — Research

## What Is This Feature?

Google Business Profile (GBP, formerly Google My Business / GMB) analytics allow local businesses to understand how their profile performs on Google Search and Maps. Key data includes search impressions, customer actions (calls, direction requests, website clicks), search keywords, and post performance. ContentStudio already supports GMB posting and inbox but has **zero analytics** for the platform.

---

## Google Business Profile Performance API v1

### Available Endpoints

| Endpoint | Granularity | Purpose |
|---|---|---|
| `locations/{id}:fetchMultiDailyMetricsTimeSeries` | Daily | Fetch multiple metrics as daily time series |
| `locations/{id}:getDailyMetricsTimeSeries` | Daily | Fetch single metric as daily time series |
| `locations/{id}/searchkeywords/impressions/monthly` | Monthly | Search keyword impression data |

### Available Metrics (DailyMetric Enum)

| Category | Metric | Description |
|---|---|---|
| **Impressions** | `BUSINESS_IMPRESSIONS_DESKTOP_MAPS` | Profile views on Maps (desktop) |
| | `BUSINESS_IMPRESSIONS_DESKTOP_SEARCH` | Profile views on Search (desktop) |
| | `BUSINESS_IMPRESSIONS_MOBILE_MAPS` | Profile views on Maps (mobile) |
| | `BUSINESS_IMPRESSIONS_MOBILE_SEARCH` | Profile views on Search (mobile) |
| **Actions** | `CALL_CLICKS` | Call button clicks |
| | `WEBSITE_CLICKS` | Website link clicks |
| | `BUSINESS_DIRECTION_REQUESTS` | Direction requests |
| | `BUSINESS_BOOKINGS` | Bookings via Reserve with Google |
| **Engagement** | `BUSINESS_CONVERSATIONS` | Message conversations |
| | `BUSINESS_FOOD_ORDERS` | Food orders |
| | `BUSINESS_FOOD_MENU_CLICKS` | Menu clicks |

### Key Limitations
- **No Direct/Discovery/Chain search split** — Legacy v4 only, being deprecated
- **No photo/video view counts** — Legacy v4 only
- **No review data from this API** — Requires separate Google My Business v4 `accounts.locations.reviews`
- **Search keywords are monthly only** — Cannot get daily keyword data
- **~24h delay** on interaction metrics; search/view metrics updated monthly

---

## Competitor Analysis

### Feature Matrix

| Feature | Metricool | Hootsuite | SocialBee | Sendible | Loomly | Agorapulse | Buffer | Sprout Social |
|---|---|---|---|---|---|---|---|---|
| Impressions (Maps/Search) | ✅ | ✅ | ✅ | ❓ | ❓ | ❓ | ❌ | ❌ |
| Search keywords | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Call clicks | ✅ | ✅ | ✅ | ❓ | ❓ | ❓ | ❌ | ❌ |
| Direction requests | ✅ | ✅ | ✅ | ❓ | ❓ | ❓ | ❌ | ❌ |
| Website clicks | ✅ | ✅ | ✅ | ✅ | ❓ | ❓ | ❌ | ❌ |
| Maps vs Search split | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Device breakdown | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Post performance | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| PDF reports | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| AI insights | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Multi-location compare | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |

### Key Takeaways
- **Metricool** is the benchmark — most comprehensive GBP analytics (keyword analytics, Maps/Search split, PDF/PPT reports)
- **Buffer and Sprout Social** have no GBP analytics at all — competitive gap
- **No competitor offers AI insights for GBP** — differentiation opportunity
- **No competitor shows device breakdown** (desktop vs mobile) — the API provides this natively, easy win
- Only Metricool shows search keywords — high value, low competition

### Recommended Differentiators for ContentStudio
1. **Search keyword analytics** — Only Metricool has this; we should build it prominently
2. **Maps vs Search + Device breakdown** — API provides 4 separate impression metrics natively; show both splits
3. **AI-powered insights** — No competitor does this for GBP
4. **Cross-platform comparison** — Show GBP alongside other social platforms in overview analytics
5. **Multi-location comparison** — Side-by-side analytics for businesses with multiple locations

---

## Existing ContentStudio Architecture

### GMB Integration (Already Built)
- **Model:** `app/Models/Integrations/Platforms/Social/GoogleMyBusinessAccounts.php` — MongoDB `social_integrations` collection
- **Repository:** `app/Repository/Integrations/Platforms/Social/GoogleMyBusinessRepo.php` — CRUD + multi-location support
- **Posting:** `app/Strategy/Planner/GmbPosting.php` — Full posting (STANDARD, EVENT, OFFER, PRODUCT types)
- **Helper:** `app/Libraries/Inbox/HelperClasses/GmbHelper.php` — Token management, API requests
- **Frontend:** `GmbPreview.vue`, `GmbOptions.vue` in both composer_v2 and publish modules

### Analytics Architecture (Pattern to Follow)

**Backend:**
- **Clickhouse** is the primary analytics store — tables per platform (`facebook_posts`, `instagram_*`, `linkedin_*`, etc.)
- **Builders** in `app/Builders/Analytics/Analyze/` — one per platform (FacebookBuilder, InstagramBuilder, etc.) + OverviewV2Builder
- **Controllers** in `app/Http/Controllers/Analytics/` — per-platform + overview + campaign/label + reports
- **Routes:** `routes/web/analytics.php` (335 lines) — platform-specific endpoints under `/analytics/overview/{platform}/`
- **AI Insights:** Per-platform controllers (e.g., `FacebookInsightsController`) using `AiAgentService` (Vellum AI)
- **Data pipeline:** `AnalyticsHelper.php` triggers Argo Python workers → data flows to Clickhouse
- **Campaign/Label analytics:** `CampaignAnalyticsModel` + `LabelAnalyticsModel` in MongoDB, linked by `platform_id` + `posted_ids`

**Frontend:**
- **Module:** `src/modules/analytics_v3/` (v3 architecture) + `src/modules/analytics/` (platform views)
- **Per-platform pattern:** Route → `MainComponent.vue` → FilterBar + TabsComponent + OverviewSection + AIInsightsSection
- **Composable:** `use<Platform>Analytics.js` per platform — defines cards, loading states, API calls
- **Widgets:** `NewStatsCard.vue` (metric cards with trend arrows, growth %), `AiInsightsCard.vue`, `MainGraph.vue`
- **Tooltips:** i18n-driven from `locales/en/analytics.json` — keys: `analytics.<platform>.composable.cards.<metric>.tooltip`
- **Sidebar:** `useAnalyticsRoutes.js` → `socialRouteConfigurations` object defines navigation entries
- **Overview:** `OverviewV2` aggregates across platforms — `PLATFORM_TITLE_AND_COLOR` constant maps platform → color/icon
- **Campaign/Label:** `useLabelAndCampaign.js` composable — cross-platform with label/campaign dropdowns
- **Export:** `ExportButton.vue` → PDF/Email/Schedule options → `ExportReportJob` + `SendReportEmailJob`

### What's Missing for GMB
- ❌ No Clickhouse tables for GMB analytics
- ❌ No GmbBuilder in `app/Builders/Analytics/Analyze/`
- ❌ No GMB analytics controller
- ❌ No GMB analytics routes
- ❌ No GMB in overview aggregation (OverviewV2Builder)
- ❌ No GMB in campaign/label analytics
- ❌ No frontend GMB analytics view/composable/route
- ❌ No GMB in sidebar navigation
- ❌ No GMB AI insights
- ❌ No GMB in export reports

---

## Files That Will Be Touched

### Backend (New)
- `app/Builders/Analytics/Analyze/GmbBuilder.php` — Clickhouse query builder for GMB metrics
- `app/Http/Controllers/Analytics/Gmb/GmbAnalyticsController.php` — API endpoints
- `app/Http/Controllers/Analytics/Gmb/GmbInsightsController.php` — AI insights
- Clickhouse migration/table creation for `gmb_*` tables
- `routes/web/analytics.php` — Add GMB routes

### Backend (Modified)
- `app/Builders/Analytics/Analyze/OverviewV2Builder.php` — Add GMB to cross-platform aggregation
- Campaign/Label analytics repos — Include GMB platform_id
- Report generation controllers — Add GMB report type
- `app/Libraries/Analytics/AnalyticsHelper.php` — GMB analytics job triggering

### Frontend (New)
- `src/modules/analytics/views/gmb/MainComponent.vue` — GMB analytics page
- `src/modules/analytics/views/gmb/composables/useGmbAnalytics.js` — State/API management
- `src/modules/analytics/views/gmb/components/` — GMB-specific widgets
- i18n entries in `locales/en/analytics.json` — GMB metric labels + tooltips

### Frontend (Modified)
- `src/modules/analytics_v3/config/routes.js` — Add GMB route
- `src/modules/analytics/components/common/composables/useAnalyticsRoutes.js` — Add GMB to sidebar
- Overview composable — Add GMB to platform list
- Campaign/Label composable — Include GMB
- Export components — Add GMB report option
- `PlatformTooltip.vue` — Add GMB tooltip lines
