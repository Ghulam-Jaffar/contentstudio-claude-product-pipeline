# PRD: Social Listening

**Author:** Product Team
**Last Updated:** 2026-02-23
**Status:** Draft
**Target Release:** Q2 2026

---

## 1. Overview

Social Listening is a new top-level ContentStudio module that lets users monitor the entire web — social media platforms, news, blogs, forums, and review sites — for mentions of their brand, keywords, competitors, and industry topics. Every matching mention is captured, classified by sentiment (positive/neutral/negative), and surfaced in a filterable live feed alongside volume trend charts and spike alerts. Users can respond to discovered mentions directly through ContentStudio's Inbox without switching tools, closing the full loop from discovery to engagement within one platform.

This feature addresses the #1 most-requested capability missing from ContentStudio and positions the product against dedicated listening tools like Brand24 ($149–$499/mo) and Agorapulse's Mention add-on ($40/topic/mo) — while offering something neither can: a seamless workflow that connects listening to publishing, scheduling, and analytics in a single platform.

---

## 2. Problem Statement

**What problem are we solving?**

ContentStudio users — primarily social media managers at agencies and growing brands — have no way to know what's being said about their clients' brands outside of their own managed channels. They only see comments and messages on accounts they've connected. Everything said on X/Twitter, Reddit, news sites, blogs, or competitor brand pages is completely invisible to them inside ContentStudio today.

This means:
- Brand crises form and grow before users are aware of them
- Competitor intelligence requires manually checking competitor profiles or using a separate tool
- Content strategy is based on internal ideas rather than real trending conversations
- Campaign effectiveness (earned media, word-of-mouth reach) cannot be measured
- Agencies cannot demonstrate the full brand impact of their work to clients in a single report

**Who has this problem?**

- **Social media managers at agencies** (primary): managing 5–50 client brands; need to monitor each client independently; need to report on brand health without extra tools
- **In-house brand managers at SMBs** (secondary): monitoring their own brand; limited budgets; currently using Brand24 ($149+/mo) or a free tool alongside ContentStudio
- Estimated: 40–60% of ContentStudio's active workspaces would benefit immediately (any workspace publishing for a brand that has a public presence worth monitoring)

**What happens if we don't solve it?**

- Users paying for Brand24 or Mention see ContentStudio as incomplete — one of the most common reasons they keep a second tool subscription
- Competitors (Hootsuite, Agorapulse, Sprout Social) use this gap actively in their sales and comparison content
- Social Listening appears on ContentStudio's public feature request board as a top-upvoted request
- We cannot compete for enterprise/agency accounts where listening is a requirement, not a nice-to-have

---

## 3. Goals & Success Metrics

| Goal | Metric | Target | How We'll Measure |
|---|---|---|---|
| Drive feature adoption | % of active workspaces with at least one Listening topic created | 30% within 90 days of launch | Product analytics (Mixpanel/Amplitude) |
| Reduce churn from users citing "missing listening" | Churn rate among users with ≥1 active listening topic | < 2% quarterly churn | Billing data cross-referenced with feature usage |
| Competitive displacement | New sign-ups citing "switched from Brand24/Mention" | Track in onboarding survey | Onboarding questionnaire |
| Upsell to higher tiers | Conversion from Starter (3 topics) to Growth (10 topics) within 60 days of Starter topic limit being hit | 25% conversion rate | Billing data |
| Core engagement | Weekly Active Users of the Listening module as % of total WAU | 15% within 6 months | Product analytics |
| Alert value | % of spike alert emails resulting in a click-through to the Listening module | > 40% email CTR | Email analytics |

---

## 4. Target Users

**Primary Persona: Agency Social Media Manager ("Sara")**
Sara manages 8–20 client accounts. She publishes content, tracks analytics, and manages inbox engagement for each client in ContentStudio. She also needs to report to clients on "how the brand is doing" — which currently means pulling data from multiple tools. She pays for ContentStudio ($89–$199/mo) and also pays for Brand24 ($249/mo) separately. She would immediately cancel Brand24 if ContentStudio offered equivalent listening. Her skill level: intermediate; she understands keywords and sentiment but is not a data scientist and will not write Boolean queries.

**Secondary Persona: In-House Brand Manager ("Marcus")**
Marcus manages one brand internally. He's less price-sensitive than Sara but more demanding on data depth and integration. He wants ContentStudio's listening tied into his published content performance so he can see "what we published" vs. "what people said" in one report. He uses ContentStudio ($49/mo) and a free Mention tier. He would upgrade to a higher ContentStudio plan if listening were included.

**Non-Users (Out of Scope):**
- Enterprise PR teams requiring global media monitoring with millions of monthly mentions, historical archives, image recognition, and audio monitoring — this is Brandwatch/Talkwalker territory; ContentStudio v1 is not positioned for this
- Individual creators with no brand presence to monitor

---

## 5. User Stories / Jobs to Be Done

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| US-1 | Social media manager | track all mentions of my client's brand name across social and web | I know immediately when people talk about them, positive or negative | Must Have |
| US-2 | Social media manager | see sentiment classification on each mention | I can quickly identify which mentions need a response vs. which to celebrate | Must Have |
| US-3 | Social media manager | receive an alert when mention volume spikes | I can respond to a potential brand crisis before it escalates | Must Have |
| US-4 | Social media manager | reply to a negative mention directly from the listening feed | I don't have to switch tools or lose track of the original mention | Must Have |
| US-5 | Agency manager | track competitor brand names alongside my client's brand | I can show clients how their share of conversation compares to competitors | Must Have |
| US-6 | Agency manager | export listening data as a PDF or CSV | I can include brand mention data in my monthly client reports | Must Have |
| US-7 | Brand manager | filter mentions by platform, sentiment, and date range | I can focus on what matters right now without scrolling through noise | Must Have |
| US-8 | Brand manager | set up a listening topic in under 5 minutes without reading documentation | The tool guides me through keyword setup without needing Boolean syntax | Must Have |
| US-9 | Social media manager | see which platforms generate the most mentions | I can prioritize where to engage | Should Have |
| US-10 | Brand manager | see mention volume and sentiment trends over time in charts | I can spot patterns and report on brand health progress over weeks/months | Must Have |
| US-11 | Agency manager | have separate listening topics per client workspace | My clients' data never mixes and I can report independently per client | Must Have |
| US-12 | Social media manager | receive an AI-generated weekly summary of what people are saying | I can brief my client in 30 seconds without reading every mention | Nice to Have (V1.5) |
| US-13 | Brand manager | see which social media accounts are driving the most conversation about my brand | I can identify potential influencer partnerships or vocal advocates | Nice to Have (V1.5) |

---

## 6. Requirements

### 6.1 Must Have (P0)

**Topic Management**
- Users can create a Listening Topic by specifying: a topic name, a primary keyword, optional additional keywords, and optional excluded keywords
- Topics are scoped to the current workspace (not shared across workspaces)
- Each workspace can have topics up to its plan limit (Starter: 3, Growth/Pro: 10, Agency/Scale: unlimited)
- Users can edit topic keywords, sources, and alert settings after creation
- Users can pause or delete a topic

**Keyword Setup — Guided Wizard**
- Topic creation uses a 3-step wizard modal (Keywords → Sources → Alerts)
- An "AI Assist" option allows users to describe in plain English what they want to monitor; the system generates a primary keyword + additional keywords + recommended exclusions
- Keyword matching supports: exact phrase match, any-keyword match, and exclusion filtering
- No raw Boolean syntax exposed in the UI for V1

**Source Coverage (V1)**
- X/Twitter (keyword search API + streaming where available)
- Instagram (mentions via webhook + hashtag search)
- Facebook (mentions via webhook)
- LinkedIn (keyword mention monitoring)
- TikTok (hashtag and keyword search)
- YouTube (video title/description/comment keyword search)
- Reddit (subreddit and keyword search via Reddit API)
- Web / News / Blogs (via RSS aggregation and web crawling)

**Mention Data**
- Every captured mention stores: platform, author handle/name, author follower count (where available), mention date/time, mention text (full or truncated to 280 chars for display), sentiment classification, URL to original post, engagement counts (likes/shares/comments/views where available), topic_id, workspace_id
- Mentions are deduplicated (same URL not ingested twice)
- Mentions are retained for: 30 days (Starter), 90 days (Growth/Pro), 365 days (Agency/Scale)

**Sentiment Classification**
- Every mention is classified as Positive, Neutral, or Negative
- Classification is performed by an external NLP API (abstracted behind a `SentimentClassifierService` for future provider swap)
- Classification must handle: sarcasm awareness, emoji sentiment, multi-language text
- Accuracy target: ≥85% agreement with human labeling on ContentStudio brand-adjacent text (to be validated in beta)

**Mention Feed**
- Chronological list of all mentions for the selected topic, within the selected date range
- Filters available: Platform (multi-select), Sentiment (multi-select: Positive / Neutral / Negative), Source Type (Social / Web), Date Range picker (presets: Last 7 days, 30 days, 90 days + custom)
- Sort options: Newest First, Most Engaged (by engagement count), Most Influential (by author follower count)
- Each mention card displays: platform icon, author name, relative timestamp ("2 hours ago"), mention text snippet with keywords highlighted, sentiment tag (colored pill), engagement count, "Reply" button, "Save" button
- Clicking "Reply" opens the ContentStudio Inbox compose view with the mention pre-loaded (platform, author, message text)
- Clicking "Save" adds the mention to a saved collection for the topic (accessible in Settings tab)

**Overview Dashboard**
- Mention Volume Chart: line chart showing daily mention count over the selected date range (7/30/90 day presets)
- Sentiment Trend Chart: stacked area chart showing daily positive/neutral/negative breakdown over time
- Sentiment Summary Donut: current period total positive/neutral/negative as % and count
- Platform Breakdown: horizontal bar chart showing mention count per platform for the selected period
- All charts respond to the same date range selector

**Competitor Tracking & Share of Voice**
- Users can create competitor topics (same mechanism as brand topics — just a different brand name as the keyword)
- On a brand topic dashboard, a "Compare" button lets users select up to 3 competitor topics for side-by-side comparison
- Comparison view shows: mention volume bar chart (side by side), sentiment comparison, and a Share of Voice donut chart (% of total conversation among all tracked brands)

**Spike Alerts**
- Users configure spike alert thresholds per topic: "Alert me when mentions spike [25% / 50% / 100% / custom %] above my 7-day rolling average"
- Alert triggers: when the 6-hour rolling mention count exceeds the configured threshold above the 7-day average rate
- Alert delivery: email to the workspace admin (and any users with notifications enabled) + in-app notification
- Alert email content: topic name, alert threshold triggered, mention volume comparison, 3 sample mentions (text + platform + sentiment), link to "View Mentions" (deep link to mentions feed filtered to last 24 hours)
- Users can configure per-topic: email alerts ON/OFF, in-app alerts ON/OFF, threshold level

**Export**
- CSV export: all mentions for the selected topic and date range, with columns: date, platform, author, mention text, sentiment, URL, engagement count
- PDF summary report: cover page (topic name, date range, workspace name), mention volume chart, sentiment chart, platform breakdown, top 5 mentions by engagement — formatted for client presentation
- Export triggered from the Overview tab "Export" button

**Workspace Isolation**
- All listening topics, mentions, and alert settings are workspace-scoped
- No data leaks between workspaces (critical for agency use case)

### 6.2 Should Have (P1)

- **Mention language filter:** Users can restrict monitoring to specific languages per topic (e.g., English only, or English + Arabic)
- **Reply tracking:** Mentions that have been replied to via Inbox show a "Replied ✓" indicator in the listening feed, so users know which mentions have been actioned
- **Topic card list view:** The Listening home screen shows all active topics as cards with: topic name, mention count (last 7 days), sentiment mini-donut, spike alert status, last mention time
- **Pagination / infinite scroll** on the mentions feed (load 50 mentions at a time)
- **Keyword highlighting** in mention text (bold the matched keyword wherever it appears in the mention body)
- **Country/region filter (V1 scope review):** Optionally available if NLP provider exposes geo-tagging; not a hard requirement for v1 launch

### 6.3 Nice to Have (P2 — V1.5)

- **AI Weekly Trend Summary:** An AI-generated narrative summary of the past 7 days for each topic — surfaced as an in-app card on the Overview tab and optionally via email ("Your Listening Digest")
- **Top Voices Panel:** Top 5 accounts (by follower count) mentioning the topic in the selected period — shown on the Overview tab as a sidebar widget
- **Saved Mention Collections:** Users can save specific mentions to named collections per topic (e.g., "Client Report — February", "Competitor Watch")
- **Create Post from Trend:** Button on trending hashtag/keyword in insights view that opens the Composer with the hashtag pre-filled

### 6.4 Explicitly Out of Scope (V1)

- Image/logo recognition in photos or video (visual listening) — requires Talkwalker/Brandwatch-level ML infrastructure
- Audio/podcast monitoring — technically complex; not in the top-tier user requests
- Emotion clustering beyond Positive/Neutral/Negative — save for V2 AI enhancement
- Dark mode — ContentStudio does not support dark mode
- RTL language UI — ContentStudio does not support RTL
- Mobile app — AI-powered features are web-only; no iOS/Android Listening module
- Real-time push streaming to mobile (V1 web only)
- Influencer list management / CRM features — separate future feature
- API access to listening data for external tools — enterprise roadmap

---

## 7. User Flow (High Level)

**Setup Flow:**
1. User navigates to the new "Listening" sidebar item
2. First-time empty state → user clicks "Create Topic"
3. 3-step wizard: (1) Keywords: name + primary keyword + additional/excluded terms + AI-assist button → (2) Sources: platform toggles + language filter → (3) Alerts: spike threshold + delivery preferences
4. Topic is created → user lands on the Overview dashboard for the new topic
5. System begins mention ingestion; initial data appears within minutes for social platforms, up to 1 hour for web sources

**Daily Use Flow:**
1. User opens ContentStudio → sees Listening in sidebar
2. User clicks into a topic → Overview shows current mention volume and sentiment at a glance
3. User switches to Mentions Feed → filters to "Negative" sentiment
4. User finds a concerning mention → clicks "Reply" → Inbox composer opens → user responds
5. Mention is marked "Replied" in the feed

**Alert Response Flow:**
1. User receives spike alert email with 3 sample mentions
2. User clicks "View Mentions" → deep link opens the mentions feed filtered to last 24 hours
3. User reviews the spike, responds to key mentions, monitors the Overview chart to confirm volume is returning to normal

---

## 8. Business Rules & Constraints

| Rule ID | Rule | Rationale |
|---|---|---|
| BR-1 | A topic must have at least one primary keyword to be created | Prevents empty topics that generate meaningless noise |
| BR-2 | Topic names must be unique within a workspace | Prevents duplicate topic confusion in the topic list |
| BR-3 | Mention history is retained for: 30 days (Starter), 90 days (Growth/Pro), 365 days (Agency/Scale) | Storage cost management; creates a tangible plan upgrade incentive |
| BR-4 | Maximum topics per workspace: Starter=3, Growth/Pro=10, Agency/Scale=unlimited | Usage limit for plan tiering; topics are the natural unit of measurement |
| BR-5 | Spike alert fires when 6-hour rolling mention count exceeds (7-day average rate × threshold multiplier) | 6-hour window avoids false alerts from momentary bursts; 7-day rolling average establishes the baseline |
| BR-6 | A minimum of 7 days of mention history must exist for a topic before spike alerts can fire | Prevents false positives when a topic is brand new and has no baseline |
| BR-7 | Sentiment classification must run on every mention before it is surfaced in the feed | Unclassified mentions are not shown to users; they are queued for retry |
| BR-8 | Mentions from the same URL ingested within a 24-hour window are deduplicated | Prevents the same mention appearing multiple times from different ingestion runs |
| BR-9 | Topics are workspace-scoped and not visible or accessible across workspaces | Agency data isolation requirement — client data must never be visible in another client's workspace |
| BR-10 | Competitor comparison is limited to topics within the same workspace | Prevents cross-workspace data exposure |
| BR-11 | The "Reply" action from a listening mention requires the platform account to be connected in the workspace | A user cannot reply to an Instagram mention if no Instagram account is connected; the Reply button shows an error with a link to Integrations settings |
| BR-12 | CSV and PDF export are available on all plans but limited to the plan's history window | Exporting 1-year data requires the Agency/Scale plan that stores 1-year history |

---

## 9. Open Questions

| Question | Options | Owner | Due Date | Decision |
|---|---|---|---|---|
| Which external NLP provider for sentiment classification? | Google Natural Language API, OpenAI (GPT-4o mini), AWS Comprehend, or Hugging Face hosted model | Engineering Lead | Sprint 1 | Pending — cost/accuracy evaluation needed |
| Twitter API tier — what data access level does ContentStudio's developer account have? | Basic ($100/mo — 500k tweets/month), Pro ($5,000/mo — 1M tweets/month) | Platform Partnerships | Pre-Sprint | Pending — determines monitoring volume for X/Twitter |
| Reddit API access — does ContentStudio have a current Reddit developer app registered? | Yes / No — if No, must apply | Engineering | Pre-Sprint | Pending |
| Should web/news monitoring be built in-house (web crawler) or via a third-party API (e.g., GDELT, NewsAPI, Meltwater)? | In-house crawler, NewsAPI ($449/mo), other vendor | Engineering / Product | Sprint 1 | Pending — build vs. buy decision |
| Is the spike alert threshold based on percentage above average, or absolute mention count threshold? | % above rolling average (recommended), absolute number, or user's choice | Product | Sprint 2 | Leaning toward % above rolling average as default with a way to also set an absolute floor (e.g., "only alert if ≥ 10 mentions in the window to avoid noise on low-volume brands") |
| How do we handle listen-only platforms where ContentStudio doesn't have publishing access? (e.g., Reddit) | Mention is shown in feed with "Reply" button disabled (no Reddit account connected) vs. external link to the original post | Product / UX | Sprint 2 | Pending — likely: show external link only for platforms where ContentStudio cannot publish (Reddit, news sites) |

---

## 10. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Twitter/X API cost escalation — Elon-era API pricing has become unpredictable; high-volume monitoring could be expensive | High | High | Launch with conservative rate limits per workspace; monitor spend; consider gating X/Twitter depth behind higher plan tiers; have a plan to substitute with web mentions if X API becomes cost-prohibitive |
| Sentiment accuracy complaints — users expect high accuracy on brand-adjacent text, including sarcasm ("ContentStudio, just great, crashed again 🙄") | Medium | High | Use a capable NLP model (not keyword-matching); run accuracy benchmarks on ContentStudio-related text before launch; add a "Mark as incorrect sentiment" feedback button so users can flag errors; use feedback to improve over time |
| High mention volume on broad keywords generating storage costs — a user tracking "coffee" generates millions of mentions/day | Medium | Medium | Validate keyword specificity in the wizard (show estimated mention volume before topic is saved); warn if keyword is too broad; enforce mention ingestion caps per workspace per day |
| Platform API deprecations — any of the 8 monitored platforms could deprecate or rate-limit their mention-access APIs | Medium | Medium | Abstract each platform behind a `PlatformListenerAdapter` interface; build one adapter per platform; platform outages affect only that source, not the whole feature |
| Data privacy compliance — GDPR/CCPA requirements around storing user-generated content from social platforms | Medium | High | Store only publicly accessible mentions; do not store user PII beyond what's returned by the platform API; implement data retention policies (30/90/365 day auto-deletion by plan); engage legal review before launch |
| Websocket/real-time load — a live mention stream per user per topic could stress Pusher infrastructure | Low | Medium | Paginate the mention feed by default; only stream new-mention notifications (count badge + toast), not full mention data in real time; full feed loads on demand via pagination |
| Mention deduplication edge cases — same content reposted across platforms or retweets generating duplicates | Low | Low | Use URL as the primary deduplication key; for platforms without canonical URLs (e.g., TikTok), use platform_id + post_id composite key |

---

## 11. Dependencies

**Internal:**
- **Inbox module** — Reply from Listening routes through the existing Inbox composer; the Inbox team must expose a "pre-load mention" API or composable for this integration
- **Authentication / Workspace module** — topics and mentions must respect existing workspace_id scoping and user permission levels
- **Notification system** — spike alerts use the existing email notification pipeline (Laravel Mailable + Lumotive service) and in-app notification system (Pusher)
- **Billing / Plan management** — topic limits and history retention windows must be gated by plan tier; billing module must expose the current workspace plan tier to the Listening service
- **ContentStudio frontend routing** — a new top-level route `/listening` must be added to the main navigation
- **Redis / Kafka infrastructure** — new Kafka topics (`contentstudio.mentions.ingest`, `contentstudio.mentions.sentiment`) and Redis cache keys for active listening topics must be added; existing infrastructure in `contentstudio-backend/config/kafka.php` is the reference

**External:**
- **NLP sentiment API** — provider TBD (Google Natural Language, OpenAI, AWS Comprehend); required before Sentiment Classification can be built; must be selected and credentials provisioned in Sprint 1
- **Twitter/X API** — requires a Developer Portal app with appropriate access tier; ContentStudio's existing Twitter app (used for publishing) may need tier upgrade for search/streaming access
- **Reddit API** — requires a registered Reddit developer app; may require separate approval from Reddit for higher rate limits
- **NewsAPI or equivalent** — required for web/news/blog monitoring; build vs. buy decision must be resolved before Sprint 1
- **Instagram Webhooks** — ContentStudio's Facebook/Instagram webhook already subscribes to the `mentions` field (`contentstudio-backend/app/Libraries/Integrations/Platforms/Social/InstagramPlatform.php`); this must be activated/configured for the Listening module

**Blockers:**
- Twitter API access tier decision must happen before Sprint 1 begins (determines scope of X/Twitter monitoring)
- NLP provider selection must happen in Sprint 1 (blocks Sentiment Classification story)
- Legal review of data retention and GDPR/CCPA compliance must be completed before any mention data is stored in production
- Reddit API developer app registration (if not already done)

---

## 12. Appendix

- **Research Report:** `docs/features/social-listening/01-research.md`
- **Workflow Design:** `docs/features/social-listening/02-workflow.md`
- **Shortcut Research Doc:** [Social Listening — Research](https://app.shortcut.com/contentstudio-team/write/IkRvYyI6I3V1aWQgIjY5OWMwYTljLTNlMzAtNDRkYy04MDU0LTA5NmQ5MTVhNTkyMiI=)
- **Competitor tools studied:** Hootsuite (Talkwalker), Sprout Social, Agorapulse (Mention), Brand24, Brandwatch, Keyhole, Buffer, Metricool, Sendible, Later
- **Existing codebase reference points:**
  - `contentstudio-backend/app/Models/Discovery/Article/CustomTopicsModel.php` — keyword filtering schema to reuse
  - `contentstudio-backend/app/Libraries/Integrations/Platforms/Social/FacebookPlatform.php` — mentions webhook already subscribed
  - `contentstudio-backend/app/Libraries/Integrations/Platforms/Social/InstagramPlatform.php` — mentions webhook already subscribed
  - `contentstudio-backend/app/Models/Inbox/` — notification + cron patterns to reuse
  - `contentstudio-frontend/src/modules/discovery/` — routing and module structure reference
  - `contentstudio-frontend/src/modules/discovery/components/feeder/components/content-view/feed-item/FeederSentimentIcon.vue` — sentiment icon component to adapt

---

## Changelog

| Date | Author | Changes |
|---|---|---|
| 2026-02-23 | Product Team | Initial draft via ContentStudio Claude pipeline |
