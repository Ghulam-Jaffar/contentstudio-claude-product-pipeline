# GMB Analytics — Epic & Stories

---

## Epic

**Title:** GMB (Google Business Profile) Analytics

**Description:**
Add comprehensive analytics for Google Business Profile (GBP/GMB) to ContentStudio. ContentStudio already supports GMB posting and inbox but has zero analytics — users must leave the platform to check performance on Google's native dashboard.

This epic delivers a dedicated GMB analytics page with impression breakdowns (Maps/Search × Desktop/Mobile), customer actions (calls, website clicks, direction requests), monthly search keywords, publishing behavior, and AI-powered insights. It also integrates GMB data into Overview analytics, Campaign & Label analytics, and the export/report system. Multi-location support with "All locations" aggregation is included.

Key differentiators over competitors: device breakdown (desktop vs mobile — no competitor shows this), AI insights for GMB (no competitor offers this), and search keyword analytics (only Metricool has this among competitors).

---

## Stories

---

### Story 1: [BE] Create Clickhouse schema and Argo data pipeline for GMB analytics

### Description:

Set up the foundational data infrastructure for GMB analytics. This involves creating Clickhouse tables to store GMB metrics and configuring an Argo Python worker to fetch data from the Google Business Profile Performance API v1.

**Clickhouse tables to create:**

1. **`gmb_daily_metrics`** — Stores daily metric time series
   - Columns: `workspace_id`, `account_id` (GMB location ID), `date`, `metric_name` (enum: BUSINESS_IMPRESSIONS_DESKTOP_MAPS, BUSINESS_IMPRESSIONS_DESKTOP_SEARCH, BUSINESS_IMPRESSIONS_MOBILE_MAPS, BUSINESS_IMPRESSIONS_MOBILE_SEARCH, CALL_CLICKS, WEBSITE_CLICKS, BUSINESS_DIRECTION_REQUESTS, BUSINESS_CONVERSATIONS, BUSINESS_BOOKINGS, BUSINESS_FOOD_ORDERS, BUSINESS_FOOD_MENU_CLICKS), `value` (Int64), `created_at`, `updated_at`
   - Partition by month, order by (workspace_id, account_id, date, metric_name)
   - Follow the same schema conventions as `facebook_insights`, `instagram_*`, etc.

2. **`gmb_search_keywords`** — Stores monthly keyword impression data
   - Columns: `workspace_id`, `account_id`, `year_month` (YYYYMM), `keyword`, `impressions` (Int64), `created_at`, `updated_at`
   - Partition by month, order by (workspace_id, account_id, year_month, impressions DESC)

3. **`gmb_posts`** — Stores GMB post performance data
   - Columns: `workspace_id`, `account_id`, `post_id`, `plan_id` (links to ContentStudio plan), `post_type` (STANDARD/EVENT/OFFER/PRODUCT), `created_time`, `views_search` (Int64), `views_maps` (Int64), `cta_clicks` (Int64), `created_at`, `updated_at`

**Argo Python worker configuration:**
- Trigger via `AnalyticsHelper.php` (`app/Libraries/Analytics/AnalyticsHelper.php`) using the existing Argo event API pattern: `{ARGO_BASE_URL}/social-account-added-{ARGO_ENV}` with `{"channel": "gmb", "account-id": "..."}`
- Worker calls GBP Performance API v1 endpoints:
  - `locations/{id}:fetchMultiDailyMetricsTimeSeries` for daily metrics
  - `locations/{id}/searchkeywords/impressions/monthly` for keyword data
- Worker writes results to Clickhouse tables
- Schedule: daily sync for metrics, monthly sync for keywords
- Token refresh via existing `GmbHelper` token management (`app/Libraries/Inbox/HelperClasses/GmbHelper.php`)

**Backend wiring:**
- Update `AnalyticsHelper.php` to support triggering GMB analytics jobs
- Add `AnalyticsJobController` endpoint for manual GMB sync trigger (follow existing pattern at `routes/api/v1.php` line 129)

---

### Workflow:

1. User connects a Google Business Profile account in Settings → Social Accounts
2. System automatically triggers the Argo GMB analytics worker within 24 hours
3. Worker fetches daily metrics and monthly keywords from Google's API
4. Data is written to Clickhouse tables, available for the analytics frontend
5. Daily sync keeps data up to date; user sees fresh data within ~24 hours of Google making it available

---

### Acceptance criteria:

- [ ] Clickhouse table `gmb_daily_metrics` is created with correct schema, partitioning, and ordering
- [ ] Clickhouse table `gmb_search_keywords` is created with correct schema
- [ ] Clickhouse table `gmb_posts` is created with correct schema
- [ ] Argo Python worker fetches daily metrics from `fetchMultiDailyMetricsTimeSeries` for all 11 DailyMetric enums
- [ ] Argo Python worker fetches monthly search keywords from the keywords endpoint
- [ ] Worker handles pagination for keyword results
- [ ] Worker handles token refresh via existing GmbHelper
- [ ] Worker handles API errors gracefully (rate limits, expired tokens, invalid locations)
- [ ] `AnalyticsHelper.php` supports triggering GMB analytics jobs
- [ ] Manual sync trigger endpoint works via AnalyticsJobController
- [ ] Data for multiple locations under the same account is stored with correct `account_id` per location
- [ ] Worker runs on a daily schedule for metrics and monthly for keywords

---

### Mock-ups:

N/A — backend only

---

### Impact on existing data:

- New Clickhouse tables added — no modification to existing tables
- `AnalyticsHelper.php` extended with GMB channel support — no changes to existing platform triggers

---

### Impact on other products:

- **Mobile apps:** No impact — mobile apps don't have per-platform analytics
- **Chrome extension:** No impact
- **White-label:** No impact — backend data infrastructure only

---

### Dependencies:

- Existing GMB account integration must be working (`GoogleMyBusinessAccounts` model, `GmbHelper` token refresh)
- Argo infrastructure must be available for new worker deployment

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 2: [BE] Build GMB dedicated analytics API endpoints

### Description:

Create the backend API endpoints and query builder for the dedicated GMB analytics page. Follow the existing per-platform pattern used by Facebook, Instagram, etc.

**Create `GmbBuilder.php`** (`app/Builders/Analytics/Analyze/GmbBuilder.php`):
Follow the pattern from `FacebookBuilder.php` and other platform builders. Query Clickhouse tables created in **[BE] Create Clickhouse schema and Argo data pipeline for GMB analytics**.

Methods to implement:
- `getSummary($params)` — Returns aggregated totals + previous period comparison for: total impressions (sum of 4 impression metrics), total actions (calls + directions + website clicks), call clicks, website clicks, direction requests, conversations. Each metric returns: current value, previous value, % change, trend direction.
- `getImpressionsTimeSeries($params)` — Returns daily time series for each of the 4 impression metrics (Maps Desktop, Maps Mobile, Search Desktop, Search Mobile). Also returns Maps vs Search split % and Desktop vs Mobile split %.
- `getActionsTimeSeries($params)` — Returns daily time series for call clicks, website clicks, direction requests.
- `getSearchKeywords($params)` — Returns paginated keyword list with impression counts for the selected month range. Sorted by impressions descending.
- `getTopPosts($params)` — Returns top-performing GMB posts by engagement (views + CTA clicks), joined with plan data from MongoDB.
- `getPublishingBehavior($params)` — Returns post count by day/week over the date range.

All methods accept: `workspace_id`, `account_ids` (array — supports multi-location "All locations"), `start_date`, `end_date`, `timezone`.

**Create `GmbAnalyticsController.php`** (`app/Http/Controllers/Analytics/Gmb/GmbAnalyticsController.php`):
- `summary(Request $request)` — Calls `GmbBuilder::getSummary()`
- `impressions(Request $request)` — Calls `GmbBuilder::getImpressionsTimeSeries()`
- `actions(Request $request)` — Calls `GmbBuilder::getActionsTimeSeries()`
- `searchKeywords(Request $request)` — Calls `GmbBuilder::getSearchKeywords()`
- `topPosts(Request $request)` — Calls `GmbBuilder::getTopPosts()`
- `publishingBehavior(Request $request)` — Calls `GmbBuilder::getPublishingBehavior()`

**Add routes** to `routes/web/analytics.php`:
```
/analytics/overview/gmb/summary
/analytics/overview/gmb/impressions
/analytics/overview/gmb/actions
/analytics/overview/gmb/search-keywords
/analytics/overview/gmb/top-posts
/analytics/overview/gmb/publishing-behavior
```

Follow the same URL pattern as other platforms (e.g., `/analytics/overview/facebook/summary`).

---

### Workflow:

1. User opens the GMB analytics page and selects a date range and location
2. Frontend calls the summary endpoint — backend queries Clickhouse for aggregated metrics and returns current + previous period values with % change
3. Frontend calls the impressions endpoint — backend returns daily time series for 4 impression metrics
4. Frontend calls the actions endpoint — backend returns daily time series for calls, clicks, directions
5. User switches to Search Keywords tab — frontend calls keywords endpoint with pagination
6. User switches to Posts tab — frontend calls top posts and publishing behavior endpoints

---

### Acceptance criteria:

- [ ] `GmbBuilder.php` created following the existing builder pattern
- [ ] `getSummary()` returns correct aggregated totals with previous period comparison and % change
- [ ] `getImpressionsTimeSeries()` returns daily data for all 4 impression metrics
- [ ] `getImpressionsTimeSeries()` includes Maps vs Search split % and Desktop vs Mobile split %
- [ ] `getActionsTimeSeries()` returns daily data for calls, website clicks, direction requests
- [ ] `getSearchKeywords()` returns paginated keyword list sorted by impressions descending
- [ ] `getTopPosts()` returns top GMB posts joined with plan metadata
- [ ] `getPublishingBehavior()` returns post count distribution over the date range
- [ ] All endpoints accept `workspace_id`, `account_ids` (array), `start_date`, `end_date`, `timezone`
- [ ] "All locations" mode: when multiple `account_ids` are passed, metrics are summed across locations
- [ ] Routes added to `routes/web/analytics.php` following existing platform URL pattern
- [ ] Controller validates required parameters and returns appropriate error responses
- [ ] Queries perform within 5 seconds for 90-day date ranges

---

### Mock-ups:

N/A — backend only

---

### Impact on existing data:

- No modification to existing data — new builder and controller only
- New routes added to `routes/web/analytics.php` — no changes to existing routes

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** No impact — API endpoints only

---

### Dependencies:

- Depends on: **[BE] Create Clickhouse schema and Argo data pipeline for GMB analytics** — Clickhouse tables must exist with data before these endpoints return results

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 3: [BE] Integrate GMB into Overview and Campaign & Label analytics

### Description:

Add GMB data to the cross-platform Overview analytics aggregation and to Campaign & Label analytics so GMB posts contribute to campaign/label performance tracking.

**Overview integration — `OverviewV2Builder.php`** (`app/Builders/Analytics/Analyze/OverviewV2Builder.php`):
- Add GMB subqueries alongside existing platform subqueries (Facebook, Instagram, LinkedIn, etc.)
- GMB impressions (sum of 4 impression metrics) should be included in the total impressions aggregation
- GMB should appear as a separate platform row in platform breakdown charts
- GMB actions (calls + directions + website clicks) should appear as a separate "GMB Actions" line item — NOT mixed into the "Engagement" total (per BR-4)
- GMB publishing behavior (post counts) included in cross-platform publishing charts
- Add GMB to `PLATFORM_TITLE_AND_COLOR` constant or equivalent for the overview

**Campaign & Label integration:**
- Update `CampaignAnalyticsRepo.php` (`app/Repository/Analytics/CampaignAnalyticsRepo.php`) to include GMB `platform_id` when querying campaign analytics
- Update `LabelAnalyticsRepo.php` (`app/Repository/Analytics/LabelAnalyticsRepo.php`) to include GMB `platform_id`
- Update `CampaignLabelAnalyticsController.php` to handle GMB platform data in `getSummaryAnalytics()` and `getCampaignLabelBreakdownData()`
- Ensure GMB posts saved via `GmbPosting.php` create entries in `campaign_analytics` and `label_analytics` MongoDB collections (check if `GmbPosting.php` already does this — if not, add it)

---

### Workflow:

1. User navigates to Analytics → Overview and selects accounts including GMB locations
2. Overview summary cards show total impressions including GMB impressions
3. Platform breakdown chart shows GMB as a separate row with its own color
4. GMB Actions appears as its own section separate from social engagement
5. User navigates to Analytics → Campaign & Label and selects a campaign
6. If the campaign includes GMB posts, GMB data appears in the campaign breakdown

---

### Acceptance criteria:

- [ ] `OverviewV2Builder` includes GMB impressions in the total impressions aggregation
- [ ] GMB appears as a separate platform in platform breakdown charts with its own color/icon
- [ ] GMB actions (calls + directions + website clicks) are NOT mixed into the Engagement total
- [ ] GMB post counts are included in cross-platform publishing behavior charts
- [ ] `CampaignAnalyticsRepo` includes GMB `platform_id` in queries
- [ ] `LabelAnalyticsRepo` includes GMB `platform_id` in queries
- [ ] GMB posts tagged with campaigns/labels create entries in `campaign_analytics` and `label_analytics` collections
- [ ] Campaign breakdown shows GMB contribution when GMB posts are in the campaign
- [ ] Overview correctly handles workspaces with no GMB accounts (no errors, GMB simply doesn't appear)

---

### Mock-ups:

N/A — backend only

---

### Impact on existing data:

- `OverviewV2Builder` modified to add GMB subqueries — existing platform queries unchanged
- Campaign/Label repos extended — no changes to existing platform data
- If `GmbPosting.php` doesn't create campaign/label analytics entries, new code added to the posting flow

---

### Impact on other products:

- **Mobile apps:** No impact — mobile doesn't use Overview or Campaign analytics
- **Chrome extension:** No impact
- **White-label:** No impact

---

### Dependencies:

- Depends on: **[BE] Create Clickhouse schema and Argo data pipeline for GMB analytics** — Clickhouse tables must exist
- Depends on: **[BE] Build GMB dedicated analytics API endpoints** — GmbBuilder methods may be reused for overview subqueries

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 4: [BE] Add GMB analytics to export report generation

### Description:

Add GMB as a report type in the analytics export/report system so users can generate PDF reports, email them, and schedule recurring reports for GMB analytics.

**Update `AnalyticsReports.php`** (`app/Http/Controllers/Analytics/Analytics/AnalyticsReports.php`):
- Add `gmb` to the list of supported report types (alongside `facebook`, `instagram`, etc.)
- For `single-pdf-overview` and `single-pdf-detailed` report types, add GMB sections:
  - Summary metrics (impressions, actions, calls, clicks, directions)
  - Impressions breakdown (Maps vs Search, Desktop vs Mobile)
  - Customer actions chart
  - Search keywords table (top 20)
  - Top posts
- For `multiple-pdf-overview`, include GMB when GMB accounts are selected
- Use MPDF library (already used for other platform reports) to render GMB report sections

**Update `ExportReportJob`** to handle GMB report data:
- Fetch data from `GmbBuilder` methods
- Format for PDF rendering

**Update `ScheduleReports.php`** (`app/Http/Controllers/Analytics/Analytics/ScheduleReports.php`):
- Allow GMB to be selected for scheduled reports
- GMB data included when "all platforms" is selected

---

### Workflow:

1. User is on the GMB analytics page and clicks Export → Export PDF
2. System generates a PDF report with GMB summary, impressions chart, actions chart, keywords table, and top posts
3. User selects Email PDF — system sends the report to specified email addresses
4. User selects Schedule PDF — system creates a recurring schedule (weekly/monthly) for GMB reports
5. User can also include GMB in multi-platform overview reports

---

### Acceptance criteria:

- [ ] `gmb` is a valid report type in `AnalyticsReports` controller
- [ ] GMB PDF report includes: summary metrics, impressions breakdown, actions chart, top 20 keywords, top posts
- [ ] GMB report renders correctly via MPDF (no layout/rendering issues)
- [ ] `ExportReportJob` handles GMB report data from `GmbBuilder`
- [ ] GMB can be included in multi-platform overview reports
- [ ] Scheduled reports support GMB as a platform option
- [ ] Email delivery works for GMB reports via `SendReportEmailJob`
- [ ] Report storage controller handles GMB report uploads correctly

---

### Mock-ups:

N/A — backend only (PDF rendering follows existing platform report layout)

---

### Impact on existing data:

- Report type enum extended with `gmb` — no changes to existing report data
- Scheduled report configurations may include GMB — backward compatible (existing schedules unaffected)

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** Reports should use white-label branding if configured (follow existing pattern)

---

### Dependencies:

- Depends on: **[BE] Build GMB dedicated analytics API endpoints** — `GmbBuilder` methods needed for report data

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 5: [BE] Create GMB AI insights endpoint

### Description:

Create an AI insights endpoint for GMB analytics using the existing `AiAgentService` (Vellum AI) pattern. Follow the same architecture as `FacebookInsightsController`, `InstagramInsightsController`, etc.

**Create `GmbInsightsController.php`** (`app/Http/Controllers/Analytics/Gmb/GmbInsightsController.php`):
Follow the pattern from `FacebookInsightsController.php`:
- Accept insight type, date range, account IDs, and metric data
- Map GMB insight types to Vellum prompt templates:
  - `impressions_overview` → Analyze impression trends (Maps vs Search, Desktop vs Mobile)
  - `actions_overview` → Analyze customer action trends (calls, clicks, directions)
  - `search_keywords` → Analyze keyword patterns and optimization opportunities
  - `publishing_behavior` → Analyze posting frequency and engagement patterns
  - `insights_summary` → Overall GMB performance summary with recommendations
- Cache results using Redis (same caching pattern as other platform insights)
- Return structured insight objects with: category, title, description, recommendations, optional chart data

**Add route** to `routes/web/analytics.php`:
```
/analytics/overview/gmb/ai_insights
```

---

### Workflow:

1. User navigates to the AI Insights tab on the GMB analytics page
2. Frontend sends the current metrics data to the AI insights endpoint
3. Backend passes the data to AiAgentService (Vellum) with GMB-specific prompt templates
4. AI generates categorized insights about impression trends, action patterns, keyword opportunities, and publishing recommendations
5. Results are cached and returned to the frontend
6. User sees collapsible insight cards grouped by category

---

### Acceptance criteria:

- [ ] `GmbInsightsController.php` created following the `FacebookInsightsController` pattern
- [ ] 5 insight types implemented: impressions_overview, actions_overview, search_keywords, publishing_behavior, insights_summary
- [ ] Each insight type calls `AiAgentService` with appropriate GMB-specific prompt templates
- [ ] Results are cached in Redis with appropriate TTL (follow existing platform caching pattern)
- [ ] Route added at `/analytics/overview/gmb/ai_insights`
- [ ] Response format matches other platform insight controllers (category, title, description, recommendations, optional chart)
- [ ] Handles AiAgentService errors gracefully (timeout, rate limit, unavailable)
- [ ] AI insights are gated behind the same plan tier as other platform AI insights

---

### Mock-ups:

N/A — backend only

---

### Impact on existing data:

- No modification to existing data — new controller and route only
- New Redis cache keys for GMB insights — no impact on existing cache

---

### Impact on other products:

- **Mobile apps:** No impact — AI insights are web-only
- **Chrome extension:** No impact
- **White-label:** No impact

---

### Dependencies:

- Depends on: **[BE] Build GMB dedicated analytics API endpoints** — `GmbBuilder` provides the metric data sent to AI
- Requires Vellum AI prompt templates to be configured for GMB insight types

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend-only story
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 6: [FE] Build GMB analytics page — Overview tab with summary cards, impressions, and actions

### Description:

Create the dedicated GMB analytics page with the Overview tab containing the filter bar, summary stat cards, impressions breakdown chart, and customer actions chart. Follow the existing per-platform pattern (reference: `src/modules/analytics/views/facebook_v2/`).

**Create route** in `src/modules/analytics_v3/config/routes.js`:
```javascript
{
  path: 'gmb/:accountId?',
  component: GmbOverview,
  name: 'gmb_analytics_v3',
  meta: { title: 'Google Business Profile | Analytics' }
}
```

**Create main view** at `src/modules/analytics/views/gmb/MainComponent.vue`:
- Filter bar with:
  - Date range picker (default: last 30 days)
  - Location selector dropdown (connected GMB locations)
  - "All locations" option at top of dropdown
- Tab navigation: Overview | Search Keywords | Posts | AI Insights
- Content area renders the active tab's component

**Create composable** at `src/modules/analytics/views/gmb/composables/useGmbAnalytics.js`:
- Define stat cards configuration (title, API key, tooltip, icon)
- Loading state management per section
- API call methods for each endpoint
- Account selection and date range handling

**Overview tab components** (`src/modules/analytics/views/gmb/components/`):

1. **Summary cards** using `NewStatsCard.vue`:
   - Total Impressions — trend + % change
   - Total Actions — trend + % change
   - Call Clicks — trend + % change
   - Website Clicks — trend + % change
   - Direction Requests — trend + % change
   - Conversations — trend + % change

2. **Impressions breakdown chart** using `MainGraph.vue`:
   - Daily line chart with 4 toggleable series (Maps Desktop, Maps Mobile, Search Desktop, Search Mobile)
   - Below chart: two split bars:
     - Maps vs Search split (percentage bar, e.g., "Maps 62% | Search 38%")
     - Desktop vs Mobile split (percentage bar, e.g., "Desktop 28% | Mobile 72%")

3. **Customer actions chart** using `MainGraph.vue`:
   - Daily line chart with 3 series: Call Clicks, Website Clicks, Direction Requests

**Add sidebar entry** in `src/modules/analytics/components/common/composables/useAnalyticsRoutes.js`:
```javascript
gmb: {
  title: 'Google Business Profile',
  icon: getSidebarIcons('gmb'),
  routeName: 'gmb_analytics_v3',
  getParams: getGmbParams,
  show: true,
}
```
Place after Pinterest, before Competitor Analytics.

**Add i18n translations** in `src/locales/en/analytics.json` with key prefix `analytics.gmb`.

**Add GMB platform color** in the overview platform color constant (e.g., Google blue `#4285F4`).

**UI Copy:**

**Stat card labels and tooltips:**

| Card | Label | Tooltip |
|---|---|---|
| Total Impressions | "Total Impressions" | "The number of times your business profile was shown to people on Google Search and Google Maps. This counts unique views per person per day." |
| Total Actions | "Total Actions" | "The total number of actions people took after seeing your profile — including calls, website visits, and direction requests combined." |
| Call Clicks | "Call Clicks" | "The number of times people tapped the 'Call' button on your Google Business Profile. Each tap counts as one click, even if the call wasn't completed." |
| Website Clicks | "Website Clicks" | "The number of times people clicked the link to your website from your Google Business Profile." |
| Direction Requests | "Direction Requests" | "The number of times people requested driving directions to your business from Google Maps." |
| Conversations | "Conversations" | "The number of message conversations started through your Google Business Profile. This counts new conversation threads, not individual messages." |

**Impressions breakdown section:**
- Section title: "Impressions Breakdown"
- Toggle labels: "Maps Desktop", "Maps Mobile", "Search Desktop", "Search Mobile"
- Split bar labels: "Google Maps" / "Google Search", "Desktop" / "Mobile"
- Tooltip for section: "See where and how people are finding your business on Google. 'Maps' means they found you while browsing Google Maps. 'Search' means they found you in Google Search results."

**Customer actions section:**
- Section title: "Customer Actions"
- Tooltip: "Actions are things people did after seeing your business profile on Google — like calling you, visiting your website, or getting directions. These are your highest-value interactions."

**Filter bar:**
- Date range label: "Date Range"
- Location selector label: "Location"
- "All locations" option label: "All Locations"
- Location selector placeholder: "Select a location"

**Tab labels:** "Overview" | "Search Keywords" | "Posts" | "AI Insights"

**Platform tooltip** (for `PlatformTooltip.vue`):
- Line 1: "Google Business Profile analytics show how people find and interact with your business on Google."
- Line 2: "Impressions tell you how many times your profile appeared in Google Search and Google Maps."
- Line 3: "Actions tell you what people did after seeing your profile — calls, website visits, and direction requests."
- Line 4: "Search keywords show which terms people used to discover your business."
- Line 5: "Data is provided by Google and may be delayed by up to 24 hours."

**Empty state** (no GMB account connected):
- Headline: "No Google Business Profile connected"
- Subtext: "Connect your Google Business Profile to see how people find and interact with your business on Google Search and Maps."
- CTA: "Connect Account" (links to Settings → Social Accounts)

**Syncing state** (account connected, no data yet):
- Headline: "Syncing your data"
- Subtext: "We're pulling your Google Business Profile analytics. This usually takes a few hours for the first sync. Check back soon!"

**Error state:**
- Headline: "Something went wrong"
- Subtext: "We couldn't load your Google Business Profile analytics. Please try again in a few minutes."
- CTA: "Retry"

**Token expired state:**
- Banner text: "Your Google Business Profile connection has expired."
- CTA: "Reconnect" (links to Settings → Social Accounts)

---

### Workflow:

1. User clicks "Google Business Profile" in the Analytics sidebar
2. User sees the filter bar with date range (default: last 30 days) and location selector
3. User selects a location (or "All Locations") — the page loads with the Overview tab active
4. User sees 6 summary stat cards showing current values with trend arrows and % change vs previous period
5. User scrolls to the Impressions Breakdown section — sees a daily line chart with 4 toggleable series and split percentage bars
6. User toggles off "Maps Desktop" and "Search Desktop" to see only mobile impressions
7. User scrolls to Customer Actions — sees daily chart for calls, website clicks, and direction requests
8. User changes the date range to last 90 days — all charts and cards refresh with new data
9. User switches to "All Locations" — metrics aggregate across all connected GMB locations

---

### Acceptance criteria:

- [ ] GMB analytics route registered at `/analytics/gmb/:accountId?`
- [ ] "Google Business Profile" appears in the Analytics sidebar after Pinterest
- [ ] Filter bar shows date range picker and location selector with "All Locations" option
- [ ] 4 tabs visible: Overview, Search Keywords, Posts, AI Insights
- [ ] Overview tab shows 6 summary stat cards with correct labels, values, trends, and tooltips
- [ ] Impressions Breakdown shows daily line chart with 4 toggleable series
- [ ] Maps vs Search split bar shows correct percentages
- [ ] Desktop vs Mobile split bar shows correct percentages
- [ ] Customer Actions shows daily line chart with 3 series (calls, clicks, directions)
- [ ] Changing date range refreshes all cards and charts
- [ ] Selecting "All Locations" aggregates data across all connected GMB locations
- [ ] Empty state shows when no GMB account is connected with "Connect Account" CTA
- [ ] Syncing state shows when account is connected but data hasn't loaded yet
- [ ] Error state shows with "Retry" button when API calls fail
- [ ] Token expired banner shows with "Reconnect" CTA
- [ ] i18n translations added for all labels and tooltips
- [ ] Platform tooltip added for GMB in `PlatformTooltip.vue`
- [ ] No hardcoded color values — uses `primary-cs-*` theme classes for primary elements
- [ ] GMB platform color (Google blue) added to platform color constant
- [ ] Loading skeletons shown while data is being fetched

---

### Mock-ups:

N/A — follow the existing Facebook analytics page pattern (`src/modules/analytics/views/facebook_v2/`)

---

### Impact on existing data:

- No data changes — new frontend views only
- New route added to analytics routes config
- New sidebar entry added to analytics sidebar config

---

### Impact on other products:

- **Mobile apps:** No impact — mobile doesn't have per-platform analytics
- **Chrome extension:** No impact
- **White-label:** Must use `primary-cs-*` theme classes. Platform color (Google blue) is neutral, not primary-themed.

---

### Dependencies:

- Depends on: **[BE] Build GMB dedicated analytics API endpoints** — API must be available for frontend to fetch data

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 7: [FE] Build GMB Search Keywords, Posts, and AI Insights tabs

### Description:

Build the remaining 3 tabs for the dedicated GMB analytics page: Search Keywords, Posts, and AI Insights. These tabs are shown within the `MainComponent.vue` created in **[FE] Build GMB analytics page — Overview tab with summary cards, impressions, and actions**.

**Search Keywords tab** (`src/modules/analytics/views/gmb/components/SearchKeywordsSection.vue`):
- Table with columns: Keyword | Monthly Impressions
- Sorted by impressions descending
- Paginated (server-side — API returns paginated results)
- Info banner at top of tab explaining the monthly data constraint

**Posts tab** (`src/modules/analytics/views/gmb/components/PostsSection.vue`):
- Publishing behavior bar chart (posts published per day/week over date range)
- Top-performing posts list (ranked by views + CTA clicks)
- Least-performing posts list
- Each post shows: post preview text (truncated), post type badge (Standard/Event/Offer/Product), date published, views (Search + Maps), CTA clicks

**AI Insights tab** (`src/modules/analytics/views/gmb/components/AIInsightsSection.vue`):
- Use existing `AiInsightsCard.vue` component (`src/modules/analytics_v3/components/AiInsightsCard.vue`)
- Collapsible sections by category: Impressions, Actions, Keywords, Publishing
- Each insight has title + description + recommendations
- Loading spinner while AI processes
- "Insufficient data" message if not enough data available

**UI Copy:**

**Search Keywords tab:**
- Info banner: "Search keywords show what people searched for on Google to find your business. This data is updated monthly by Google, so it may not reflect the exact date range you've selected."
- Info banner icon: `ℹ` (info icon)
- Table column headers: "Keyword" | "Monthly Impressions"
- Empty state headline: "No keyword data available yet"
- Empty state subtext: "Google updates search keyword data monthly on the 1st. If you recently connected your account, check back after the start of next month."
- Pagination: "Showing {start}-{end} of {total} keywords"

**Posts tab:**
- Publishing behavior chart title: "Publishing Frequency"
- Publishing behavior tooltip: "How often you published posts to your Google Business Profile over the selected period."
- Top posts section title: "Top Performing Posts"
- Top posts tooltip: "Your best-performing Google Business Profile posts, ranked by total views across Search and Maps plus any call-to-action button clicks."
- Least posts section title: "Least Performing Posts"
- Post type badges: "Standard" | "Event" | "Offer" | "Product"
- Post metrics labels: "Views" | "CTA Clicks"
- Empty state headline: "No posts published yet"
- Empty state subtext: "Publish posts to your Google Business Profile to see how they perform. You can create GMB posts from the Composer."
- Empty state CTA: "Go to Composer" (links to Composer)

**AI Insights tab:**
- Section title: "AI Insights"
- Loading text: "Analyzing your Google Business Profile data..."
- Insight categories: "Impressions Insights" | "Actions Insights" | "Keyword Insights" | "Publishing Insights"
- Empty/insufficient data: "Not enough data for AI insights"
- Subtext: "AI insights require at least 14 days of analytics data. Keep your account connected and check back soon."

---

### Workflow:

1. User clicks the "Search Keywords" tab
2. User sees an info banner explaining that keyword data is monthly
3. User sees a table of keywords sorted by impressions descending
4. User pages through keyword results to see more terms
5. User clicks the "Posts" tab
6. User sees a publishing frequency bar chart showing how often they posted
7. User scrolls to see top-performing and least-performing posts with preview text and metrics
8. User clicks the "AI Insights" tab
9. User sees a loading spinner while insights are generated
10. User sees collapsible insight sections with analysis and recommendations for impressions, actions, keywords, and publishing

---

### Acceptance criteria:

- [ ] Search Keywords tab shows a paginated table of keywords with monthly impressions
- [ ] Info banner about monthly data constraint is visible at the top of the Keywords tab
- [ ] Keyword table supports server-side pagination
- [ ] Keywords sorted by impressions descending by default
- [ ] Posts tab shows publishing frequency bar chart
- [ ] Posts tab shows top-performing posts with: preview text, post type badge, date, views, CTA clicks
- [ ] Posts tab shows least-performing posts with same data
- [ ] Post type badges correctly display Standard/Event/Offer/Product
- [ ] AI Insights tab uses existing `AiInsightsCard.vue` component pattern
- [ ] AI insights show collapsible sections for 4 categories (Impressions, Actions, Keywords, Publishing)
- [ ] Loading spinner shows while AI processes
- [ ] "Insufficient data" state shows when less than 14 days of data available
- [ ] Empty states show with correct copy for keywords (no data yet) and posts (no posts published)
- [ ] All labels and tooltips use i18n translations
- [ ] No hardcoded color values — uses `primary-cs-*` theme classes for primary elements

---

### Mock-ups:

N/A — follow existing patterns: keyword table follows standard data table pattern; posts follow `TopPosts` component pattern from other platforms; AI insights follow `AiInsightsCard.vue` pattern

---

### Impact on existing data:

- No data changes — new frontend tab components only

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** Must use theme-aware classes

---

### Dependencies:

- Depends on: **[FE] Build GMB analytics page — Overview tab with summary cards, impressions, and actions** — Main page with tab navigation must exist
- Depends on: **[BE] Build GMB dedicated analytics API endpoints** — Keywords, posts, and publishing behavior endpoints needed
- Depends on: **[BE] Create GMB AI insights endpoint** — AI insights endpoint needed for AI Insights tab

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 8: [FE] Integrate GMB into Overview analytics and Campaign & Label analytics

### Description:

Update the Overview analytics page and Campaign & Label analytics page to include GMB data alongside existing platforms.

**Overview analytics** (`src/modules/analytics/views/overviewV2/`):
- Add GMB to the platform filter tabs (TabsComponent) so users can filter to see GMB-only data
- Add GMB to the account selector so GMB locations appear alongside other platform accounts
- Add GMB platform entry in `PLATFORM_TITLE_AND_COLOR` constant in `useOverviewAnalytics.js`:
  - Title: "Google Business Profile"
  - Color: `#4285F4` (Google blue)
  - Icon: Google "G" icon
- GMB impressions should appear in the total impressions summary card
- GMB should appear as a row in platform breakdown charts
- GMB actions displayed as a separate line item (not mixed with social engagement)
- GMB posts included in publishing behavior section
- Add Pusher channel support for GMB sync status updates (`useOverviewPusherAnalytics`)

**Campaign & Label analytics** (`src/modules/analytics/views/performance-report/label-and-campaign/`):
- Add GMB to the platform breakdown display
- GMB post data should appear when a campaign/label includes GMB posts
- Update `useLabelAndCampaign.js` composable to handle GMB platform data
- GMB media type mapping in the media fetching logic

**UI Copy:**

**Overview — GMB platform tab:**
- Tab label: "Google Business"
- Tab tooltip: "View analytics for your Google Business Profile accounts only."

**Overview — GMB in breakdown charts:**
- Legend label: "Google Business Profile"
- When hovering on GMB row: "Impressions from your Google Business Profile across Google Search and Maps."

**Campaign & Label — GMB breakdown:**
- Platform label: "Google Business Profile"
- When GMB data is included: standard platform row in breakdown table

---

### Workflow:

1. User navigates to Analytics → Overview
2. User sees GMB locations in the account selector alongside Facebook, Instagram, etc.
3. User selects GMB locations and sees total impressions increase to include GMB data
4. Platform breakdown charts show "Google Business Profile" as a separate colored row
5. User clicks the "Google Business" filter tab to see GMB-only data
6. User navigates to Analytics → Campaign & Label
7. User selects a campaign that includes GMB posts
8. Campaign breakdown shows GMB contribution alongside other platforms

---

### Acceptance criteria:

- [ ] GMB appears as a platform tab in Overview analytics filter
- [ ] GMB locations appear in the Overview account selector
- [ ] GMB platform color (`#4285F4`) and icon added to `PLATFORM_TITLE_AND_COLOR`
- [ ] Total impressions in Overview include GMB impressions when GMB accounts are selected
- [ ] Platform breakdown charts show GMB as a separate row
- [ ] GMB actions are NOT mixed into the Engagement summary — shown separately
- [ ] GMB posts appear in publishing behavior charts
- [ ] Pusher integration handles GMB sync status updates
- [ ] Campaign & Label analytics includes GMB data in platform breakdown
- [ ] `useLabelAndCampaign.js` handles GMB platform type correctly
- [ ] No errors when workspace has no GMB accounts (GMB simply doesn't appear)
- [ ] All new labels and tooltips use i18n translations
- [ ] No hardcoded color values — theme-aware classes used for primary elements

---

### Mock-ups:

N/A — follows existing pattern of how other platforms appear in Overview and Campaign & Label views

---

### Impact on existing data:

- No data changes — frontend display logic only
- New platform entry added to color/icon constants

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** Must use theme-aware classes for any primary-colored elements. Platform color (Google blue) is neutral.

---

### Dependencies:

- Depends on: **[BE] Integrate GMB into Overview and Campaign & Label analytics** — Backend must return GMB data in overview and campaign/label endpoints

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---
---

### Story 9: [FE] Add GMB to analytics export reports

### Description:

Add GMB as a report type option in the analytics export UI so users can generate, email, and schedule GMB analytics reports.

**Update `ExportButton.vue`** (`src/modules/analytics/views/common/ExportButton.vue`):
- When on the GMB analytics page, the export button should be available
- File naming via `getFileName()` helper should include "google-business-profile" for GMB reports

**Update report modals:**
- `ScheduleReportModal` — GMB should appear as a selectable platform for scheduled reports
- `SendReportByEmailModal` — GMB report type should be supported
- When "all platforms" is selected for scheduled reports, GMB should be included

**Update report listing:**
- `DownloadReports.vue` — GMB reports should appear in the generated reports list with correct platform label and icon
- `MyReport.vue` — Scheduled GMB reports should appear in the scheduled reports view

**UI Copy:**

**Export button (on GMB analytics page):**
- Same pattern as other platforms: "Export" dropdown → "Export PDF" | "Email PDF" | "Schedule PDF"

**Schedule report modal — GMB option:**
- Platform label: "Google Business Profile"
- Platform icon: Google "G" icon

**Download reports list:**
- Report name format: "Google Business Profile Report — {date range}"
- Status: "Generating..." → "Ready to download"

**Email subject line:**
- "Google Business Profile Analytics Report — {workspace name}"

---

### Workflow:

1. User is on the GMB analytics page and clicks the Export button
2. User selects "Export PDF" — a GMB analytics report begins generating
3. User sees "Generating..." status in the Reports section
4. Report completes — user downloads the PDF containing GMB summary, impressions, actions, keywords, and posts
5. Alternatively, user selects "Email PDF" — enters email addresses and the report is sent
6. Alternatively, user selects "Schedule PDF" — configures weekly/monthly schedule and recipients

---

### Acceptance criteria:

- [ ] Export button is visible and functional on the GMB analytics page
- [ ] "Export PDF" generates a GMB analytics report
- [ ] "Email PDF" opens the email modal and sends the report to specified addresses
- [ ] "Schedule PDF" opens the schedule modal with GMB as the selected platform
- [ ] GMB appears as a selectable platform in scheduled report configuration
- [ ] GMB is included when "all platforms" is selected for scheduled reports
- [ ] Generated GMB reports appear in the Download Reports view with correct label and icon
- [ ] Scheduled GMB reports appear in the My Reports view
- [ ] File naming includes "google-business-profile" identifier
- [ ] Feature gate check (`canAccess('exports_schedule_reports')`) applies to GMB report scheduling
- [ ] All labels use i18n translations
- [ ] No hardcoded color values

---

### Mock-ups:

N/A — follows the existing export button and modal patterns from other platform analytics pages

---

### Impact on existing data:

- No data changes — UI updates to export components only
- GMB added to platform options in report modals

---

### Impact on other products:

- **Mobile apps:** No impact
- **Chrome extension:** No impact
- **White-label:** Reports should use white-label branding if configured (existing pattern handles this)

---

### Dependencies:

- Depends on: **[BE] Add GMB analytics to export report generation** — Backend must support GMB report type
- Depends on: **[FE] Build GMB analytics page — Overview tab with summary cards, impressions, and actions** — GMB page must exist for export button placement

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)
