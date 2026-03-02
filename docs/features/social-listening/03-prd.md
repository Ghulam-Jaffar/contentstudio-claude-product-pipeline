# PRD: Social Listening

**Author:** Product Team
**Last Updated:** 2026-03-02
**Status:** Updated — Prototype Complete
**Target Release:** Q2 2026

---

## 1. Overview

Social Listening is a new top-level ContentStudio module that lets users monitor the entire web — social media platforms, news, blogs, forums, and review sites — for mentions of their brand, keywords, competitors, and industry topics. Every matching mention is captured, classified by sentiment (positive/neutral/negative), and surfaced in a filterable live feed alongside volume trend charts and spike alerts. Users can respond to discovered mentions directly through ContentStudio's Inbox without switching tools, closing the full loop from discovery to engagement within one platform.

This feature addresses the #1 most-requested capability missing from ContentStudio and positions the product against dedicated listening tools like Brand24 ($149–$499/mo) and Agorapulse's Mention add-on ($40/topic/mo) — while offering something neither can: a seamless workflow that connects listening to publishing, scheduling, and analytics in a single platform.

The feature ships as an **add-on module** at **$99/month** (or $79/month billed annually). It is visible to all users in the sidebar, but gated behind a purchase. Non-subscribers see a polished landing page and can explore a live demo topic. A prototype has been built at `cs-prototypes/app/features/listening/`.

---

## 2. Problem Statement

ContentStudio users — primarily social media managers at agencies and growing brands — have no way to know what's being said about their clients' brands outside of their own managed channels. They only see comments and messages on accounts they've connected. Everything said on X/Twitter, Reddit, news sites, blogs, or competitor brand pages is completely invisible to them inside ContentStudio today.

This means:
- Brand crises form and grow before users are aware of them
- Competitor intelligence requires manually checking competitor profiles or using a separate tool
- Content strategy is based on internal ideas rather than real trending conversations
- Campaign effectiveness (earned media, word-of-mouth reach) cannot be measured
- Agencies cannot demonstrate the full brand impact of their work to clients in a single report

**Who has this problem?**
- **Social media managers at agencies** (primary): managing 5–50 client brands; need to monitor each client independently; need to report on brand health without extra tools
- **In-house brand managers at SMBs** (secondary): monitoring their own brand; limited budgets; currently using Brand24 ($249/mo) alongside ContentStudio

---

## 3. Goals & Success Metrics

| Goal | Metric | Target | How We'll Measure |
|---|---|---|---|
| Drive feature adoption | % of active workspaces with ≥1 Listening topic | 30% within 90 days of launch | Product analytics |
| Reduce churn from users citing missing listening | Churn rate among users with ≥1 active topic | < 2% quarterly | Billing data |
| Competitive displacement | New sign-ups from Brand24/Mention switchers | Track in onboarding survey | Onboarding questionnaire |
| Upsell | Conversion from Starter → Growth after hitting topic limit | 25% conversion rate | Billing data |
| Core engagement | Weekly Active Users of Listening as % of total WAU | 15% within 6 months | Product analytics |
| Alert value | % of spike alert emails resulting in click-through | > 40% email CTR | Email analytics |

---

## 4. Target Users

**Primary Persona: Agency Social Media Manager ("Sara")**
Sara manages 8–20 client accounts. She publishes content, tracks analytics, and manages inbox engagement for each client in ContentStudio. She also needs to report to clients on "how the brand is doing" — which currently means pulling data from multiple tools. She pays for ContentStudio ($89–$199/mo) and also pays for Brand24 ($249/mo) separately. She would immediately cancel Brand24 if ContentStudio offered equivalent listening.

**Secondary Persona: In-House Brand Manager ("Marcus")**
Marcus manages one brand internally. He wants ContentStudio's listening tied into his published content performance so he can see "what we published" vs. "what people said" in one report. He uses ContentStudio ($49/mo) and a free Mention tier.

---

## 5. Pricing

Social Listening is a **flat add-on** — a single tier, not a tiered plan.

- **Monthly:** $99/month
- **Annual:** $79/month (billed as $948/year, saves ~20%)

The AddOnPricingBlock on the landing page shows a billing cycle toggle (Monthly / Annual). When annual is selected, it displays "$79/mo" with "Billed as $948/year" below.

---

## 6. User States

Four states drive different UI experiences:

| State | Who | UI |
|---|---|---|
| **trial** | On ContentStudio trial; no Social Listening add-on | Landing page with orange alert banner requiring Agency Unlimited plan first |
| **locked** | Has active ContentStudio plan; no Social Listening add-on | Landing page (no alert banner) |
| **unlocked** | Has active Social Listening add-on | Full topic list and all dashboard features |
| **expired** | Previously had add-on, subscription lapsed | If topics exist: topic list in churned mode (dimmed cards, UpgradeModal auto-opens). If no topics: landing page |

Route logic in `/features/listening/page.tsx`:
- `unlocked` → Topic List
- `expired` + topics → Topic List (churned mode)
- all other states → Landing Page

---

## 7. User Stories / Jobs to Be Done

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| US-1 | Social media manager | track all mentions of my client's brand across social and web | I know immediately when people talk about them | Must Have |
| US-2 | Social media manager | see sentiment classification on each mention | I can quickly identify which need a response vs. which to celebrate | Must Have |
| US-3 | Social media manager | receive an alert when mention volume spikes | I can respond to a potential brand crisis before it escalates | Must Have |
| US-4 | Social media manager | reply to a negative mention from the listening feed | I don't have to switch tools | Must Have |
| US-5 | Agency manager | track competitor brand names alongside my client's brand | I can show clients share of conversation vs. competitors | Must Have |
| US-6 | Agency manager | export listening data as a PDF or CSV | I can include brand mention data in monthly client reports | Must Have |
| US-7 | Brand manager | filter mentions by platform, sentiment, and date range | I can focus on what matters without scrolling through noise | Must Have |
| US-8 | Brand manager | set up a listening topic in under 5 minutes | The tool guides me through keyword setup without Boolean syntax | Must Have |
| US-9 | Social media manager | see AI-generated insights on every analytics chart | I get actionable takeaways without reading raw numbers | Should Have |
| US-10 | Brand manager | see mention volume and sentiment trends over time | I can spot patterns and report on brand health progress | Must Have |
| US-11 | Agency manager | have separate listening topics per client workspace | My clients' data never mixes | Must Have |
| US-12 | Agency manager | preview a static report before exporting | I can verify the content before sending to clients | Should Have |

---

## 8. Feature Requirements

### 8.1 Topic Management (P0)

- Users create a Listening Topic via a 4-step wizard: Basic Info → Keywords → Sources → Alerts
- Topics are workspace-scoped
- Plan limits: Starter=3, Growth/Pro=10, Agency/Scale=unlimited
- Users can edit topic keywords, sources, alert settings after creation
- Users can pause or delete a topic

### 8.2 Keyword Setup — Query Builder (P0)

Three keyword groups with AND/OR/NOT logic:
- **Include Keywords** (required, OR logic) — typed tags with type selector: Text / Hashtag / Mention
- **Required Keywords** (optional, AND logic) — every mention must also contain these
- **Excluded Keywords** (optional, NOT logic) — excludes matching mentions

Real-time conflict detection: include+exclude conflict, duplicate detection, short keyword warnings, volume estimation.

**AI Assist**: Users describe in plain English what to monitor → system generates Include/Required/Exclusions suggestions.

### 8.3 Source Coverage — V1 (P0)

Social: X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Bluesky, Pinterest, Threads

Web & Media: News Sites, Blogs & Websites, Forums, Review Sites, Podcasts

Additional controls: Language filter (170+ languages), hide reshares toggle, verified-only toggle.

### 8.4 Mention Data & Feed (P0)

Every mention stores: platform, author, follower count, date/time, text, sentiment, URL, engagement counts, topic_id, workspace_id.

Feed features:
- Filters: Platform (multi-select), Sentiment (multi-select), Source Type, Date Range
- Sort: Newest First, Most Engaged, Most Influential
- Each mention card: platform icon, author, timestamp, text snippet, sentiment pill, engagement count, Reply button, Save button
- Clicking Reply → Inbox composer pre-loaded with mention
- Mentions retained: 30 days (Starter), 90 days (Growth/Pro), 365 days (Agency/Scale)

### 8.5 Analytics Dashboard (P0)

The Analytics tab has two sub-tabs: **Performance** and **Insights**.

#### KPI Grid (always visible)
7 metrics shown as cards: Total Mentions, Reach, Impressions, Engagements, Sentiment Score, Net Reach, Avg Sentiment Velocity. AI Insights button visible above the KPI grid.

#### Performance Sub-tab
- Mention Volume Chart (area chart, line + bar toggle)
- Reach Chart (area chart)
- Engagement Chart (bar/line)
- Impressions Chart (area chart)
- Sentiment Trend Chart (stacked area)
- Sentiment Distribution Donut (with sentiment score)
- Network Breakdown Table (sortable: Mentions, Followers, Reach, Engagement, Impressions)

#### Insights Sub-tab
- Sentiment by Platform (grouped bar)
- Geographic Distribution (choropleth map + bar chart)
- Influencer/Top Authors Panel (top voices by reach)
- Word Cloud (keyword frequency visualization)
- Context charts (hashtags, content types, etc.)

#### AI Insights on Every Widget
Every chart and table widget has a violet Sparkles (✨) ActionIcon button in its header. Clicking it opens a Mantine Popover showing 2–3 AI-generated insight bullets specific to that widget's data. The button is visible in both interactive and static (report preview) modes.

### 8.6 Spike Alerts (P0)

- Per-topic threshold: 25% / 50% / 100% / custom % above 7-day rolling average
- Minimum mention floor: N mentions in a 6-hour window
- Alert delivery: email to configured recipients + in-app notification
- Alert email: topic name, threshold triggered, volume comparison, 3 sample mentions, deep link to mentions feed
- Alerts activate only after 7 days of data collection (baseline requirement)

### 8.7 Export & Reports (P1)

#### Export Modal (3 tabs)
Available from the layout header across all tabs via "Export" button.

1. **Download** — PDF summary report or CSV raw data; choose report type:
   - Performance & Mentions (always available)
   - Compare Topics (enabled only when competitor topics exist)
   - Compare Periods (always available)

2. **Schedule** — configure recurring report delivery (daily/weekly/monthly, day/time, email recipients)

3. **Share** — generate a shareable link to a read-only web view of the report

#### Reports Tab
Separate tab with two sub-tabs:

**Downloaded Reports sub-tab:**
- Table of all generated reports with: name, type, date range, status (generating/ready/failed), actions
- Each ready report row has a Download button and a **View button (eye icon)**
- Clicking View opens the Report Preview Drawer

**Scheduled Reports sub-tab:**
- Table of configured recurring reports with: name, frequency, next delivery date, recipients, toggle

#### Report Preview Drawer
A right-side drawer (Mantine Drawer, position="right", size="xl") triggered by the View (eye) button.

Contents — static, non-interactive analytics snapshot:
1. Key Metrics — 7 KPI cards (values only)
2. Mention Volume — AreaChart (day granularity, no controls)
3. Sentiment Overview — 2-col: SentimentDonut | Platform BarChart
4. Geographic Distribution — BarChart by country
5. Top Hashtags — horizontal BarChart

All sections use `ChartCard isStatic={true}` — hides all dropdown controls while keeping AI Insights button visible.

### 8.8 Competitor Comparison (P1)

The Compare tab allows side-by-side comparison of up to 3 competitor topics:
- Mention volume bar chart (side by side)
- Sentiment comparison
- Share of Voice donut chart (% of total conversation among all tracked brands)

### 8.9 Settings Tab (P1)

Per-topic settings editable after creation:
- **Keywords** — same 3-group keyword editor (Include / Required / Exclude) with conflict detection and AI Assist
- **Platforms** — same platform toggle grid as wizard Step 3
- **Members** — team member MultiSelect for topic visibility/access

---

## 9. User Flows

### Setup Flow
1. User navigates to Listening in sidebar
2. Non-subscriber → Landing Page → "Add to My Plan" or "Open Demo Topic"
3. Subscriber → Topic list → "+ Create Topic" → 4-step wizard
4. Wizard completes → topic dashboard (Analytics tab, default)
5. Initial data appears within minutes for social, up to 1 hour for web

### Non-Subscriber Preview Flow
1. Non-subscriber navigates to `/features/listening/new`
2. Completes wizard → lands on Preview page
3. Preview shows sample KPI bar, mention volume chart, sentiment donut, 10 mock mention cards (labeled "Sample mention")
4. Upgrade CTA prominently shown with pricing

### Daily Use Flow
1. User opens ContentStudio → Listening in sidebar → topic list
2. Clicks into a topic → Analytics tab shows current volume and sentiment at a glance
3. Switches to Mentions tab → filters to "Negative" sentiment
4. Clicks "Reply" → Inbox composer opens pre-loaded → user responds
5. Mention is marked "Replied" in the feed

### Alert Response Flow
1. User receives spike alert email with 3 sample mentions
2. Clicks "View Mentions" → deep link opens the mentions feed filtered to last 24 hours
3. User reviews spike, responds to key mentions, monitors the volume chart

### Report Preview Flow
1. User navigates to Reports tab
2. Clicks View (eye icon) on a ready report row
3. Report Preview Drawer opens from the right
4. Reads static analytics snapshot — no interactions available on charts
5. Closes drawer or downloads the report

---

## 10. Business Rules & Constraints

| Rule ID | Rule | Rationale |
|---|---|---|
| BR-1 | A topic must have at least one Include keyword | Prevents empty topics generating meaningless noise |
| BR-2 | Topic names must be unique within a workspace | Prevents duplicate topic confusion |
| BR-3 | Mention history: 30 days (Starter), 90 days (Growth/Pro), 365 days (Agency/Scale) | Storage cost management; plan upgrade incentive |
| BR-4 | Max topics: Starter=3, Growth/Pro=10, Agency/Scale=unlimited | Plan tiering |
| BR-5 | Spike alert fires when 6-hour rolling count exceeds (7-day avg × threshold) | 6-hour window avoids false alerts; 7-day rolling baseline |
| BR-6 | Minimum 7 days of history before spike alerts can fire | Prevents false positives on new topics |
| BR-7 | Sentiment classification runs before mention is shown in feed | Unclassified mentions are not surfaced |
| BR-8 | Same URL deduplicated within 24-hour ingestion window | Prevents same mention appearing multiple times |
| BR-9 | Topics are workspace-scoped — no cross-workspace visibility | Agency data isolation |
| BR-10 | Reply action requires the platform account to be connected in the workspace | Cannot reply to Instagram mention without a connected Instagram account |
| BR-11 | "Compare Topics" export is disabled when no competitor topics exist in the workspace | Prevents empty comparison reports |
| BR-12 | AI Insights button is visible in both interactive and static (report preview) modes | AI value should be available even in exported/shared contexts |

---

## 11. Open Questions

| Question | Options | Owner | Status |
|---|---|---|---|
| Which NLP provider for sentiment classification? | Google Natural Language API, OpenAI GPT-4o mini, AWS Comprehend | Engineering Lead | Pending — cost/accuracy evaluation |
| Twitter/X API tier? | Basic ($100/mo), Pro ($5,000/mo) | Platform Partnerships | Pending — determines X/Twitter monitoring volume |
| Reddit API developer app registered? | Yes / No | Engineering | Pending |
| Web/news monitoring: build vs. buy? | In-house crawler vs. NewsAPI ($449/mo) | Engineering/Product | Pending |
| Spike alert threshold basis? | % above rolling average (recommended) vs. absolute count | Product | Leaning % with absolute floor option |
| Platform-only listening (Reddit/news) — Reply button behavior? | Disabled + external link vs. hidden | Product/UX | Leaning: show external link for platforms where ContentStudio cannot publish |

---

## 12. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Twitter/X API cost escalation | High | High | Conservative rate limits; gate X depth behind higher plan tiers; fallback to web mentions if cost-prohibitive |
| Sentiment accuracy complaints | Medium | High | Use capable NLP model; accuracy benchmarks before launch; user feedback button for incorrect sentiment |
| High mention volume from broad keywords | Medium | Medium | Estimate volume in wizard before saving; warn if keyword too broad; ingestion caps per workspace |
| Platform API deprecations | Medium | Medium | Abstract each platform behind PlatformListenerAdapter interface |
| GDPR/CCPA compliance | Medium | High | Store only publicly accessible mentions; 30/90/365 day auto-deletion; legal review before launch |

---

## 13. Dependencies

**Internal:**
- **Inbox module** — Reply from Listening routes through the existing Inbox composer
- **Authentication / Workspace module** — topics and mentions respect workspace_id scoping
- **Notification system** — spike alerts use the existing email pipeline (Laravel Mailable) and in-app notifications (Pusher)
- **Billing / Plan management** — topic limits and history windows gated by plan tier
- **Frontend routing** — new top-level route `/listening` in main navigation
- **Redis / Kafka** — new Kafka topics (`contentstudio.mentions.ingest`, `contentstudio.mentions.sentiment`) and Redis cache keys

**External:**
- NLP sentiment API — provider TBD; required before Sentiment Classification story begins
- Twitter/X Developer API — existing app may need tier upgrade for search/streaming
- Reddit API — registered developer app required
- NewsAPI or equivalent for web/news/blog monitoring
- Instagram Webhooks — existing webhook already subscribes to `mentions` field; must be activated for Listening

---

## 14. Prototype Reference

A fully functional interactive prototype is available at `cs-prototypes/app/features/listening/`.

All routes compile and build clean. The prototype implements:
- All 4 user states (trial/locked/unlocked/expired) via the DebugToggle (bottom-right floating panel)
- Full landing page with AddOnPricingBlock ($99/mo, $79/mo annual)
- Demo topic at `/features/listening/demo` (redirects to `/features/listening/topic-demo/analytics`)
- 4-step topic creation wizard
- Topic dashboard with Analytics, Mentions, Compare, Reports, Settings tabs
- Full analytics with 2 sub-tabs (Performance + Insights) and KPI grid
- AI Insights Sparkles button on every chart widget and table
- Report Preview Drawer with static non-interactive analytics snapshot
- Export Modal with 3 tabs (Download / Schedule / Share)
- Settings tab with keyword editor, platform toggles, member access

Spec: `docs/features/social-listening/04-spec.md`

---

## 15. Appendix

- **Research Report:** `docs/features/social-listening/01-research.md`
- **Workflow Design:** `docs/features/social-listening/02-workflow.md`
- **Feature Spec:** `docs/features/social-listening/04-spec.md`
- **Shortcut Research Doc:** [Social Listening — Research](https://app.shortcut.com/contentstudio-team/write/IkRvYyI6I3V1aWQgIjY5OWMwYTljLTNlMzAtNDRkYy04MDU0LTA5NmQ5MTVhNTkyMiI=)
- **Shortcut PRD Doc:** [Social Listening — PRD](https://app.shortcut.com/contentstudio-team/write/IkRvYyI6I3V1aWQgIjY5OWMxODI0LWUyYTYtNDE3Yi1hZTU0LWUwYTFiOTQwOGI1YSI=)
- **Competitor tools studied:** Hootsuite (Talkwalker), Sprout Social, Agorapulse (Mention), Brand24, Brandwatch, Keyhole, Buffer, Metricool, Sendible, Later

---

## Changelog

| Date | Author | Changes |
|---|---|---|
| 2026-02-23 | Product Team | Initial draft via ContentStudio Claude pipeline |
| 2026-03-02 | Product Team | Updated: pricing ($99/mo, $79/mo annual), user states (trial/locked/unlocked/expired), AI Insights on every widget, Report Preview Drawer, Reports tab sub-tabs, export modal 3-tab structure, settings keyword editor, prototype reference added, sections restructured to 15 |
