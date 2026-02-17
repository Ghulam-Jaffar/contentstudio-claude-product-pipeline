# **PRD: GMB (Google Business Profile) Analytics**

**Author:** Product Owner
**Last Updated:** 2026-02-16
**Status:** In Review
**Target Release:** Q1 2026

---

## **1. Overview**

Add comprehensive analytics for Google Business Profile (GBP/GMB) to ContentStudio. This includes a dedicated GMB analytics page with impression breakdowns, customer actions, search keywords, and AI insights; integration into Overview analytics and Campaign & Label analytics; and full export/report support. ContentStudio currently supports GMB posting and inbox but has zero analytics — this fills a critical gap that competitors like Metricool and Hootsuite already address, and adds differentiators (AI insights, device breakdown) that no competitor offers.

---

## **2. Problem Statement**

**What problem are we solving?**

ContentStudio users who manage Google Business Profiles cannot see how their GMB posts and profiles perform. They must leave ContentStudio and use Google's native Business Profile dashboard to check impressions, calls, website clicks, and direction requests. This breaks the "single platform" value proposition and creates friction for agencies managing multiple locations across multiple platforms.

**Who has this problem?**

- Local business owners using ContentStudio to manage their GMB presence alongside social media
- Agencies managing multiple client GMB locations who need unified reporting
- Social media managers who want cross-platform analytics (including GMB) in one dashboard

**What happens if we don't solve it?**

- Users managing GMB accounts have an incomplete analytics experience — every other supported platform has analytics except GMB
- Agencies cannot include GMB data in client reports, reducing ContentStudio's value for multi-platform reporting
- Competitors like Metricool, Hootsuite, and SocialBee already offer GMB analytics — continued absence puts ContentStudio at a competitive disadvantage
- Users may adopt secondary tools (Metricool) specifically for GMB analytics, increasing churn risk

---

## **3. Goals & Success Metrics**

| Goal | Metric | Target | How We'll Measure |
|---|---|---|---|
| GMB analytics adoption | % of workspaces with GMB accounts that view GMB analytics | 50% within 60 days of launch | Product analytics (page views) |
| Cross-platform reporting completeness | % of export reports that include GMB data | 20% of reports from GMB-connected workspaces | Report generation logs |
| Reduce platform-switching | Drop in users leaving CS to check Google Business Profile dashboard | -25% based on session analysis | Session analytics |
| Feature satisfaction | User feedback / NPS for GMB analytics | Positive sentiment, <5 support tickets/month | Intercom + in-app feedback |

---

## **4. Target Users**

**Primary Persona:**
Local Business Owner / Social Media Manager — Manages 1-5 GMB locations alongside social media accounts. Wants to see how their Google profile is performing (are people finding them? calling? visiting the website?) without leaving ContentStudio. Non-technical, needs clear labels and plain-language tooltips.

**Secondary Persona:**
Agency Account Manager — Manages 10-50+ GMB locations across multiple client workspaces. Needs aggregated and per-location analytics, scheduled PDF reports for clients, and campaign/label tracking that includes GMB posts. Values efficiency and unified reporting.

**Non-Users (explicitly out of scope):**
- Users without GMB accounts connected — feature is gated behind having at least one GMB account
- SEO specialists looking for deep local SEO tools (keyword ranking, citation tracking) — this is analytics, not SEO tooling

---

## **5. User Stories / Jobs to Be Done**

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| US-1 | Social media manager | See how many people viewed my GMB profile on Maps and Search | I know if my profile visibility is growing | Must Have |
| US-2 | Local business owner | See how many calls, direction requests, and website clicks my profile generated | I can measure the real business impact of my GMB presence | Must Have |
| US-3 | Social media manager | See which search keywords people used to find my business | I can optimize my profile description and posts for those terms | Must Have |
| US-4 | Agency manager | See GMB analytics aggregated across all my client's locations | I get a single view of multi-location performance | Must Have |
| US-5 | Agency manager | Include GMB data in my scheduled PDF reports to clients | I don't need a separate tool for GMB reporting | Must Have |
| US-6 | Social media manager | See GMB data alongside other platforms in the Overview dashboard | I get a complete cross-platform picture | Must Have |
| US-7 | Social media manager | See how my GMB posts tagged with campaigns/labels performed | I can measure campaign effectiveness across all platforms including GMB | Must Have |
| US-8 | Social media manager | Get AI-powered insights about my GMB performance trends | I get actionable recommendations without manual analysis | Should Have |
| US-9 | Social media manager | Break down impressions by Maps vs Search and Desktop vs Mobile | I understand where and how people are finding my business | Should Have |
| US-10 | Social media manager | See my top-performing GMB posts | I know what content resonates on Google | Should Have |

---

## **6. Requirements**

### **6.1 Must Have (P0)**

* **Dedicated GMB analytics page** accessible from Analytics sidebar with date range and location selectors
* **Summary stat cards** — Total Impressions, Total Actions, Call Clicks, Website Clicks, Direction Requests, Conversations — each with current value, trend arrow, and % change vs previous period
* **Impressions chart** — Daily line chart over selected date range with toggleable series (Maps Desktop, Maps Mobile, Search Desktop, Search Mobile)
* **Customer actions chart** — Daily line chart for calls, website clicks, direction requests
* **Search keywords table** — Monthly keyword impressions, sorted descending, paginated
* **Publishing behavior section** — Posts published over time + top/least performing posts
* **Overview analytics integration** — GMB impressions included in cross-platform totals; GMB shown as a platform row in breakdown charts
* **Campaign & Label analytics integration** — GMB posts included when filtering by campaign or label
* **Export reports** — GMB available as a report type (PDF export, email, scheduled)
* **Multi-location support** — Location selector with "All locations" aggregation option
* **Backend data pipeline** — Argo Python worker to fetch GBP Performance API v1 data into Clickhouse
* **Clickhouse tables** for GMB daily metrics and search keywords

### **6.2 Should Have (P1)**

* **AI insights** — AI-generated analysis of GMB trends, anomalies, and recommendations using AiAgentService (Vellum)
* **Impressions split visualization** — Maps vs Search percentage bar + Desktop vs Mobile percentage bar
* **Tab structure** — Overview, Search Keywords, Posts, AI Insights tabs on dedicated page
* **GMB in overview "publishing behavior"** — GMB posts included in cross-platform publishing charts
* **Empty states and error handling** — Clear messaging for no data, syncing, token expired, API errors

### **6.3 Nice to Have (P2)**

* **Bookings metric** (Reserve with Google) — only relevant for eligible business categories
* **Food orders / menu clicks** — only relevant for restaurants
* **Conversations metric** — messaging may have limited adoption

### **6.4 Explicitly Out of Scope**

* **Review analytics** (star ratings, review trends, response rates) — requires separate Google My Business v4 reviews API; deferred to v2
* **Multi-location comparison view** (side-by-side) — v2 feature
* **Competitor benchmarking** — not available from Google's API
* **Geographic distribution of direction requests** — not available in Performance API v1
* **Photo/video view analytics** — legacy API only, being deprecated
* **Direct/Discovery/Chain search split** — legacy API only, being deprecated
* **Mobile app (iOS/Android) GMB analytics** — mobile apps don't currently have per-platform analytics pages

---

## **7. User Flow (High Level)**

### Dedicated GMB Analytics Page

1. User navigates to **Analytics → Google Business Profile** from the left sidebar
2. User sees a filter bar with date range picker (default: last 30 days) and location selector
3. User sees the **Overview tab** with summary stat cards (Impressions, Actions, Calls, Website Clicks, Directions, Conversations) each showing value + trend + % change
4. Below cards, user sees **Impressions Breakdown** — daily line chart with toggleable Maps/Search × Desktop/Mobile series, plus split percentage bars
5. Below that, **Customer Actions** — daily line chart for calls, website clicks, directions
6. User clicks **Search Keywords** tab — sees a table of keywords with monthly impression counts, sorted by impressions descending, paginated
7. User clicks **Posts** tab — sees publishing frequency chart and top/least performing posts
8. User clicks **AI Insights** tab — sees AI-generated analysis grouped by category (impressions, actions, keywords) with trend descriptions and recommendations
9. User clicks **Export** button → selects PDF/Email/Schedule to generate a report

### Overview Integration

1. User navigates to **Analytics → Overview**
2. GMB accounts appear in the account selector alongside other platforms
3. Summary cards include GMB impressions in the total impressions count
4. Platform breakdown charts show GMB as a separate color/row
5. Publishing behavior includes GMB posts

### Campaign & Label Integration

1. User navigates to **Analytics → Campaign & Label**
2. Selects a campaign or label containing GMB posts
3. Analytics aggregation includes GMB post performance data
4. Platform breakdown shows GMB contribution

---

## **8. Business Rules & Constraints**

| Rule ID | Rule | Rationale |
|---|---|---|
| BR-1 | Search keyword data is monthly granularity only — changing the date range does not affect keyword data granularity | Google's API limitation; keywords endpoint only returns monthly data |
| BR-2 | Analytics data has ~24 hour delay from Google | Google's API doesn't provide real-time data |
| BR-3 | "All locations" aggregation sums metrics across all connected GMB locations in the workspace | Consistent with how other platforms handle multi-account aggregation |
| BR-4 | GMB actions (calls, directions, website clicks) are NOT mixed into the Overview "engagement" total | These are fundamentally different from social engagement (likes, comments, shares); shown as separate line items |
| BR-5 | GMB impressions ARE included in the Overview "impressions" total | Impressions are conceptually similar across platforms |
| BR-6 | Only locations with active (non-expired) tokens fetch analytics data | Prevents failed API calls and misleading gaps in data |
| BR-7 | Bookings, food orders, and menu clicks are only shown if the location has non-zero data for these metrics | Avoids showing irrelevant zero-value cards for businesses that don't use these Google features |

---

## **9. Open Questions**

| Question | Options | Owner | Due Date | Decision |
|---|---|---|---|---|
| Should we show Bookings/Food Orders/Menu Clicks cards by default or hide them? | Always show / Show only if data exists / Hide entirely for v1 | Product | Before dev | Show only if data exists (P2) |
| How far back should initial data sync go? | 30 days / 90 days / 1 year | Engineering | Before dev | Pending — depends on API limits and Clickhouse storage cost |
| Should GMB AI insights be gated behind a paid plan tier? | Same tier as other AI insights / Higher tier / Available to all | Product | Before dev | Same tier as other platform AI insights |

---

## **10. Risks & Mitigations**

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Google deprecates more legacy API endpoints, limiting available metrics | Medium | Medium | Build exclusively on Performance API v1 (current); avoid legacy v4 endpoints. Monitor Google's deprecation notices. |
| Low adoption if users don't have GMB accounts connected | Medium | Medium | Add awareness prompts in Analytics sidebar: "Connect your Google Business Profile to unlock analytics." Feature gate keeps it invisible to non-GMB users. |
| Search keywords monthly delay confuses users who expect daily data | Medium | Low | Clear UI messaging: "Updated monthly by Google" note in Keywords tab. Separate tab prevents mixing with daily metrics. |
| Clickhouse storage costs increase with new GMB tables | Low | Low | Use same retention and aggregation policies as other platform analytics tables. GMB has fewer metrics than Facebook/Instagram. |
| Argo Python worker for GBP API hits rate limits | Low | Medium | Implement exponential backoff and per-location rate limiting in the worker. Google's GBP API has generous limits for authenticated requests. |
| Multi-location accounts with 50+ locations slow down "All locations" queries | Low | Medium | Aggregate at ingestion time (pre-computed totals in Clickhouse). Limit "All locations" to 50 locations with a warning for larger accounts. |

---

## **11. Dependencies**

* **Internal:**
  - Existing GMB account integration (`GoogleMyBusinessAccounts` model, `GmbHelper` library) — must have valid token refresh working
  - Argo data pipeline infrastructure — new Python worker needed to fetch GBP Performance API data
  - Clickhouse cluster — new tables for GMB metrics
  - AiAgentService (Vellum) — new prompt templates for GMB insight generation
  - Campaign & Label analytics repos — need to include GMB `platform_id`
  - Overview V2 Builder — needs GMB subqueries added

* **External:**
  - Google Business Profile Performance API v1 — primary data source
  - Google My Business API v4 — for account/location metadata (already integrated)
  - Google OAuth token refresh — already working via `GmbHelper`

* **Blockers:**
  - None — all prerequisites are in place (GMB accounts, Argo infra, Clickhouse, AI service)

---

## **12. Appendix**

* Research document: `docs/features/gmb-analytics/01-research.md`
* Workflow design: `docs/features/gmb-analytics/02-workflow.md`
* Google Business Profile Performance API v1 reference: https://developers.google.com/my-business/reference/performance/rest
* DailyMetric enum values: https://developers.google.com/my-business/reference/performance/rest/v1/DailyMetric
* Existing analytics architecture: `app/Builders/Analytics/Analyze/` (builder pattern), `routes/web/analytics.php` (route definitions)
* Frontend analytics pattern: `src/modules/analytics/views/facebook_v2/` (reference implementation to follow)

---

## **Changelog**

| Date | Author | Changes |
|---|---|---|
| 2026-02-16 | Product Owner | Initial draft |
