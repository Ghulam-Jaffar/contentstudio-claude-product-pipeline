# GMB Analytics — Workflow Design

---

## 1. Feature Placement

GMB Analytics integrates into **four existing areas** of ContentStudio:

### Entry Points
1. **Analytics Sidebar** — New "Google Business Profile" entry under the existing platform list (below Pinterest, above Competitor Analytics). Icon: Google "G" icon. Route: `/analytics/gmb/:accountId?`
2. **Overview Analytics** — GMB data included in cross-platform aggregation (impressions, engagement, publishing behavior)
3. **Campaign & Label Analytics** — GMB posts included when filtering by campaign or label
4. **Export Reports** — New "Google Business Profile" option in report generation (PDF export, email, scheduled reports)

### Navigation Flow
```
Analytics (left sidebar)
├── Overview          ← GMB added to cross-platform aggregation
├── Facebook
├── Instagram
├── LinkedIn
├── Twitter/X
├── TikTok
├── YouTube
├── Pinterest
├── Google Business Profile  ← NEW dedicated page
├── Campaign & Label  ← GMB included in platform data
├── Competitor Analytics
└── Reports           ← GMB report type added
```

---

## 2. User Flow — Dedicated GMB Analytics Page

### Happy Path

1. User navigates to **Analytics → Google Business Profile** from the left sidebar
2. User sees a **filter bar** at the top with:
   - Date range picker (default: last 30 days)
   - Location account selector (dropdown of connected GMB locations)
   - "All locations" option for multi-location aggregation
3. User sees the **Summary section** with stat cards:
   - Total Impressions (with Maps vs Search breakdown)
   - Total Actions (calls + directions + website clicks combined)
   - Call Clicks
   - Website Clicks
   - Direction Requests
   - Conversations
   - Each card shows: current value, trend arrow, % change vs previous period
4. User scrolls to **Impressions Breakdown** section:
   - Line chart showing daily impressions over selected date range
   - Toggleable series: Maps Desktop, Maps Mobile, Search Desktop, Search Mobile
   - Summary bar showing Maps vs Search split (percentage)
   - Summary bar showing Desktop vs Mobile split (percentage)
5. User scrolls to **Customer Actions** section:
   - Line chart showing daily calls, website clicks, direction requests
   - Individual stat cards with trends
6. User scrolls to **Search Keywords** section:
   - Table showing keywords + monthly impression count
   - Sorted by impressions descending
   - Paginated (API returns paginated results)
7. User scrolls to **Publishing Behavior** section:
   - Posts published over time (bar chart)
   - Top-performing posts by engagement
8. User scrolls to **AI Insights** section:
   - AI-generated analysis of trends, anomalies, and recommendations
   - Collapsible insight cards grouped by category (impressions, actions, keywords)
9. User clicks **Export** button in the header → can export PDF, email PDF, or schedule recurring report

### Tab Structure (within GMB analytics page)

| Tab | Contents |
|---|---|
| Overview | Summary cards + Impressions chart + Actions chart |
| Search Keywords | Keyword table with monthly impressions |
| Posts | Publishing behavior + top/least performing posts |
| AI Insights | AI-generated analysis and recommendations |

---

## 3. User Flow — Overview Analytics (Cross-Platform)

1. User navigates to **Analytics → Overview**
2. User selects date range and accounts (GMB locations now appear in account selector)
3. Overview summary cards now include GMB data in aggregated totals:
   - Impressions: GMB impressions added to total
   - Engagement: GMB actions (calls + directions + clicks) added
4. Platform breakdown shows GMB as a separate row/color in charts
5. Per-platform cards show GMB alongside Facebook, Instagram, etc.

---

## 4. User Flow — Campaign & Label Analytics

1. User navigates to **Analytics → Campaign & Label**
2. Selects a campaign or label
3. If the campaign/label includes posts published to GMB, those posts' analytics are included in the aggregation
4. Platform breakdown shows GMB contribution alongside other platforms

---

## 5. User Flow — Export Reports

1. User is on any analytics page (GMB dedicated, Overview, or Campaign & Label)
2. User clicks **Export** dropdown → selects "Export PDF" / "Email PDF" / "Schedule PDF"
3. For GMB dedicated reports: includes all GMB sections (summary, impressions, actions, keywords, posts)
4. For Overview reports: GMB data included in cross-platform totals
5. Scheduled reports include GMB data when GMB accounts are selected

---

## 6. Alternative Flows & Edge Cases

| Scenario | Behavior |
|---|---|
| No GMB account connected | Show empty state: "Connect a Google Business Profile account to see analytics." with CTA to Settings → Social Accounts |
| GMB account connected but no data yet | Show "Syncing data..." state with progress indicator. Data appears within 24-48 hours after first sync |
| Multi-location account | Location selector shows all locations. "All locations" aggregates across all. Individual location shows that location only |
| Date range with no data | Show zero values with "No data available for this period" note |
| Search keywords — no data | Show "Keyword data is updated monthly by Google. Check back after the 1st of next month." |
| API rate limit hit | Show "Data is temporarily unavailable. Please try again in a few minutes." with retry button |
| Account token expired | Show warning banner: "Your Google Business Profile connection has expired. Reconnect to continue receiving analytics." with reconnect CTA |
| Export with no data | PDF shows "No data available" sections instead of empty charts |

---

## 7. Key Design Decisions

### Decision 1: Daily vs Monthly Search Keywords

**Options:**
- A) Show only monthly keywords (what the API provides)
- B) Show monthly keywords but display them in a separate tab to set expectations

**Recommendation: Option B** — Keywords are valuable but monthly-only. Putting them in their own tab with a note ("Updated monthly by Google") prevents confusion when users change date ranges and keyword data doesn't change.

### Decision 2: Multi-Location Handling

**Options:**
- A) Single location selector only — user picks one location at a time
- B) "All locations" aggregation + individual location selector
- C) Multi-location comparison view (side-by-side)

**Recommendation: Option B for v1, defer C to v2** — Aggregation across locations is useful for agencies and multi-location businesses. Side-by-side comparison is a v2 differentiator. The account selector already supports "All accounts" in other platform analytics.

### Decision 3: Where to Place GMB in Overview Analytics

**Options:**
- A) Full integration — GMB metrics counted in all overview totals (impressions, engagement, etc.)
- B) Partial integration — GMB appears as a platform row but isn't mixed into "Engagement" since GMB "actions" aren't the same as social media "engagement"
- C) Separate section — GMB gets its own card in overview, not mixed with social metrics

**Recommendation: Option B** — GMB impressions naturally aggregate with other platform impressions. But GMB "actions" (calls, directions) are fundamentally different from social "engagement" (likes, comments, shares). Show GMB as a platform in the breakdown charts, include impressions in totals, but keep GMB actions as their own line item rather than mixing into the engagement total.

---

## 8. Integration with Existing Features

| Feature | Integration |
|---|---|
| **Composer** | Already supports GMB posting. Post IDs will link to analytics. |
| **Planner** | GMB posts in planner can link to their analytics data. |
| **Overview Analytics** | GMB added as a platform in cross-platform aggregation. |
| **Campaign & Label** | GMB post analytics included when posts are tagged with campaigns/labels. |
| **Export Reports** | New GMB report type in export options. GMB data in overview reports. |
| **AI Insights** | New GmbInsightsController using AiAgentService (Vellum) — same pattern as other platforms. |
| **Settings → Social Accounts** | Already supports GMB account connection. No changes needed. |
| **Argo Data Pipeline** | New Python worker to fetch GBP Performance API data → Clickhouse. Triggered via AnalyticsHelper. |

---

## 9. Scope: v1 vs v2

### v1 (This Release)

- ✅ Dedicated GMB analytics page with 4 tabs (Overview, Keywords, Posts, AI Insights)
- ✅ Summary stat cards (impressions, actions, conversations)
- ✅ Impressions breakdown chart (Maps/Search × Desktop/Mobile)
- ✅ Customer actions chart (calls, website clicks, directions)
- ✅ Search keywords table (monthly)
- ✅ Publishing behavior + top posts
- ✅ AI insights for GMB
- ✅ GMB in Overview analytics (impressions + platform breakdown)
- ✅ GMB in Campaign & Label analytics
- ✅ GMB in export reports (PDF/email/scheduled)
- ✅ Multi-location support with "All locations" aggregation

### v2 (Deferred)

- ⏳ Multi-location comparison view (side-by-side)
- ⏳ Review analytics (ratings, review trends, response rates) — requires separate API integration
- ⏳ Competitor benchmarking (not available from API, would need manual/third-party data)
- ⏳ Geographic distribution of direction requests
- ⏳ Photo/video view analytics (legacy API only, may be deprecated)
- ⏳ Bookings & food orders analytics (niche — only relevant for restaurants/services with Reserve with Google)
