# Social Listening — Complete Feature Spec

## Context

ContentStudio currently has no way for users to monitor what's being said about their clients' brands outside managed channels. Social Listening is the #1 most-requested missing feature. This spec covers the complete feature design — from the marketing/upsell landing page through setup, daily use, analytics, comparison, exports, and settings — with full copy, tooltips, empty/loading/error states, and a prototype implementation reference in `cs-prototypes/`.

The feature ships as an **add-on module**: visible to all users in the sidebar, but gated behind a purchase. Non-subscribers see a polished landing page and can explore a live demo topic. Trial users see the landing page with an alert explaining they need to upgrade their base plan first before adding Social Listening.

---

## 1. Naming Decision: "Topics"

We call the monitored entities **"Topics"** (not Projects, Searches, or Monitors).

- Sprout Social (the premium benchmark) uses "Topics" — agency users recognize it
- Intuitive to non-technical marketers: "I'm tracking a topic" = natural language
- Scales semantically: brand topic, competitor topic, campaign topic, industry topic
- Avoids "Project" confusion (ContentStudio uses "Workspace" for accounts)
- Short, clean, works in all UI contexts: "Your topics", "Create topic", "3 topics active"

**Secondary terms:**
- The act of monitoring = "listening"
- The results = "mentions" (not posts, not results)
- The configuration = "topic setup" or "topic settings"
- The dashboard for a topic = "topic dashboard" or just the topic name

---

## 2. Module Structure & Routes

### Routes (cs-prototypes prototype)
```
/features/listening                             → Landing (trial/locked/expired-no-topics) OR Topic List (unlocked/expired-with-topics)
/features/listening/demo                        → Server redirect → /features/listening/topic-demo/analytics
/features/listening/new                         → Create Topic Wizard (step 1)
/features/listening/new?step=2                  → Wizard step 2
/features/listening/new?step=3                  → Wizard step 3
/features/listening/new?step=4                  → Wizard step 4
/features/listening/new/preview                 → Preview results (non-subscriber post-setup)
/features/listening/[topicId]                   → Redirect → /[topicId]/analytics
/features/listening/[topicId]/analytics         → Analytics tab (default)
/features/listening/[topicId]/mentions          → Mentions feed tab
/features/listening/[topicId]/compare           → Compare tab
/features/listening/[topicId]/reports           → Reports tab
/features/listening/[topicId]/settings          → Settings tab
```

Note: The Overview tab has been removed. The default tab is now Analytics (labelled "Performance" in the tab bar). The `/demo` route is a Next.js server-side redirect to `/topic-demo/analytics`.

### File Structure (cs-prototypes)
```
app/features/listening/
├── page.tsx                          # Route logic: landing vs topic list vs churned list
├── layout.tsx                        # Listening shell layout
├── demo/
│   └── page.tsx                      # Server redirect to /features/listening/topic-demo/analytics
├── new/
│   ├── page.tsx                      # Wizard (4 steps via query param)
│   └── preview/
│       └── page.tsx                  # Post-setup preview (non-subscribers)
├── [topicId]/
│   ├── layout.tsx                    # Topic dashboard shell (tabs, filter bar, export modal)
│   ├── analytics/page.tsx            # Performance + Insights sub-tabs
│   ├── mentions/page.tsx
│   ├── compare/page.tsx
│   ├── reports/page.tsx              # Downloaded Reports + Scheduled Reports
│   ├── settings/page.tsx
│   └── components/
│       ├── KPICardsGrid.tsx          # 7-card KPI grid
│       ├── KPISummaryBar.tsx         # Compact KPI bar (legacy/compact)
│       ├── TopicFilterBar.tsx        # Filter trigger button (Zustand-connected)
│       ├── FilterSidebar.tsx         # Right-panel filter drawer content
│       └── ExportModal.tsx           # 3-tab export modal (rendered in layout)
│       ├── NetworkBreakdownTable.tsx # Sortable platform breakdown table
├── components/
│   ├── landing/
│   │   └── LandingPage.tsx           # Full landing page
│   ├── topic-list/
│   │   └── TopicListPage.tsx         # Topic grid + churned state
│   ├── wizard/
│   │   └── (step components)
│   └── shared/
│       ├── ChartCard.tsx             # Reusable chart wrapper
│       ├── DebugToggle.tsx           # Prototype state switcher
│       ├── UpgradeModal.tsx          # Reusable upgrade/churn modal
│       ├── PlatformBadge.tsx
│       ├── SentimentBadge.tsx
│       ├── InfoTooltip.tsx
│       ├── UpgradeGate.tsx
│       └── EmptyState.tsx

lib/
├── store.ts                          # Zustand store
├── mock-data.ts                      # All mock topics, mentions, analytics
├── types.ts                          # TypeScript interfaces
└── constants.ts                      # Platform list, language list, team members, etc.
```

---

## 3. User States

Four user states drive different UI experiences. The prototype DebugToggle (bottom-right floating panel) lets reviewers switch between them.

| State | Who | UI |
|---|---|---|
| **trial** | On ContentStudio trial; no Social Listening add-on | Landing page with orange alert banner explaining Agency Unlimited is required |
| **locked** | Has active ContentStudio plan (any paid tier); no Social Listening add-on | Landing page (no alert banner) |
| **unlocked** | Has active Social Listening add-on | Full topic list and all dashboard features |
| **expired** | Previously had add-on but subscription lapsed | If topics exist: topic list in churned mode (dimmed cards, UpgradeModal auto-opens, orange banner). If no topics: landing page |

The Zustand store tracks `userState: 'trial' | 'locked' | 'unlocked' | 'expired'`. The computed `isSubscribed` flag is `userState === 'unlocked'`.

### Route logic in `/features/listening/page.tsx`
```
unlocked            → <TopicListPage />
expired + topics    → <TopicListPage churned />
all other states    → <LandingPage />
```

---

## 4. Pricing

Social Listening is a flat add-on — a single tier, not a tiered plan.

- **Monthly:** $99/month
- **Annual:** $79/month (billed as $948/year, saves ~20%)

The AddOnPricingBlock on the landing page shows a billing cycle toggle (Monthly / Annual) and displays the price accordingly with "Billed as $948/year" when annual is selected.

---

## 5. Landing Page

Shown for `trial` and `locked` states.

### 5.1 Navigation Entry Point
All users see "Listening" in the left sidebar. Clicking it goes to the landing page when unsubscribed.

### 5.2 Trial-Specific Alert
When `userState === 'trial'`, an orange full-width alert banner is shown at the top of the landing page:

```
[Alert — orange, full-width, radius=0]
  ⚠  Social Listening requires an active Agency Unlimited plan.
     Upgrade your ContentStudio plan to add this feature.
```

### 5.3 Landing Page Structure

#### Section 1 — Hero
```
[Small label]  Social Listening

[H1]  Know what the world is saying
      about your brand — before it matters.

[Subtext]  Monitor mentions of your brand, competitors, and industry keywords
           across social media, news, blogs, Reddit, and more.
           Get real-time alerts, AI-powered insights, and actionable data —
           all inside ContentStudio.

[CTA Row]
  [Primary button]  Add to My Plan
  [Secondary button]  Open Demo Topic  →

[Hero visual: animated-style card showing:
  - a topic card: "ContentStudio Brand" · Active badge
  - KPI row: Mentions 1,247 · Sentiment 72/100 · Reach 4.2M (with ↑ deltas)
  - sentiment bar: 52% positive / 34% neutral / 14% negative
  - spike alert notification: "Spike Alert — 180% above 7-day average"]
```

Note: "Open Demo Topic" button links to `/features/listening/demo` which server-redirects to `/features/listening/topic-demo/analytics`.

There is **no** "no sign-up required" text anywhere on the landing page.

#### Section 2 — Feature Pillars (6 cards in 3×2 grid)
Each card: icon + title + 2-line description

1. **Monitor Any Brand or Keyword** — Track your brand, competitor names, campaign hashtags, industry terms — across 10+ platforms and the entire web.
2. **Real-Time Mention Feed** — See every mention the moment it's published. Filter by platform, sentiment, content type, or date. Reply directly without leaving ContentStudio.
3. **AI-Powered Sentiment Analysis** — Every mention is automatically classified as Positive, Neutral, or Negative — with a composite Sentiment Score that tells you exactly where your brand stands.
4. **Spike Alerts Before Crises Escalate** — Get notified the moment mentions spike beyond your normal baseline — with sample mentions included so you know what's driving the surge.
5. **Competitor & Share of Voice Tracking** — Monitor competitor brands and see your share of the conversation. Know when competitors are gaining ground — or when your audience is choosing you.
6. **White-Label Reports in One Click** — Generate branded PDF reports with your client's logo, your accent color, and exactly the charts that matter. Schedule weekly or monthly delivery.

#### Section 3 — How It Works (3 horizontal steps)
```
[Step 1]  Set Up a Topic in 2 Minutes
          Name your topic, add your brand keywords, choose platforms,
          and configure alerts. AI suggests keywords based on your description.
          No Boolean syntax required.

[Step 2]  Collect Mentions Automatically
          ContentStudio monitors 10+ platforms and the web 24/7.
          Every mention is classified, scored, and stored — ready for you
          to filter, sort, and act on.

[Step 3]  Act, Report, and Improve
          Reply to mentions from the feed, export PDF reports for clients,
          compare your brand against competitors, and use AI insights
          to improve your content strategy.
```

#### Section 4 — Pricing (AddOnPricingBlock)
Single-card block (not 3-tier). Compact, centered, max-width ~460px.

```
[Billing toggle: Monthly | Annual (save ~20%)]

[Price: $99/mo]  (or $79/mo when Annual selected)
[If Annual: "Billed as $948/year"]

[Button — gradient violet→indigo, full width]  Add to my plan

[Footer links]  Questions? Chat with us · View docs →
```

#### Section 5 — Dark Footer (combined CTA + links)
Single dark section combining the final CTA and informational links:

```
[Dark section — deep navy background]

[H2 — light]  Ready to start listening?
[Subtext — muted]  Monitor your brand, track competitors, and uncover insights
                    across the entire web — starting today.

[Button — gradient violet→indigo]  Add to My Plan

[Footer links row]
  Read the Social Listening documentation →
  Watch a 3-minute walkthrough →
  See customer stories →
```

There is **no** separate "Demo Strip" section with "no sign-up required" text.

---

## 6. Demo Topic

Always accessible to all users via the route `/features/listening/demo` (server redirect to `/features/listening/topic-demo/analytics`). The demo topic is pre-loaded as **"ContentStudio Brand"** with realistic mock data.

All tabs work normally with sample data. Create/edit actions are not available in the demo topic.

### Demo Banner
A persistent banner at the top of the topic layout differentiates by `userState`:

**Trial state** — orange gradient background:
```
[Banner — linear-gradient(135deg, #FFF7ED, #FFEDD5), orange border-bottom]
  👀 You're viewing a demo topic. Upgrade your ContentStudio plan to Agency Unlimited to add Social Listening.
  [Button: orange light variant]  Upgrade Plan
```

**Locked or Expired state** — violet gradient background:
```
[Banner — linear-gradient(135deg, #EEF2FF, #F5F3FF), indigo border-bottom]
  👀 You're viewing a demo topic with sample data. Add Social Listening to your plan to start monitoring real mentions.
  [Button: gradient violet→indigo]  Add to My Plan
```

Clicking either button navigates to `/features/listening`.

---

## 7. Topic List (Unlocked & Churned States)

### 7.1 Standard Topic List (Unlocked)

#### Header
```
[H1]  Listening
[Subtext]  Monitor brands, keywords, and competitors across the web.
[Right side: Button]  + Create Topic
```

#### Topic Cards (grid)
Each card shows:
- Color dot + topic name (bold) + status badge (Active / Paused / Setting up...)
- Competitor badge (orange outline, "Competitor") if `isCompetitor=true`
- KPI chips: Mentions · Reach · Sentiment · (other key metrics)
- Mini sparkline or engagement indicators

#### Plan Limit Banner
When at max topics, an amber banner appears above the grid:
```
[Banner — amber]
  You've reached your topic limit (N of M). Upgrade your plan to track more topics simultaneously.

[Create Topic button: disabled with tooltip explaining the limit]
```

#### Info Bar (dismissible, bottom of page)
After the topic grid, an informational bar appears:
```
[Dismissible info bar — at BOTTOM of page, after topic grid]
  What is Social Listening?
  • Monitor brand mentions across all major platforms
  • Track competitor activity and share of voice
  • Get real-time alerts when conversations spike
  [link]  Explore Demo →
```

#### Empty State (0 topics)
```
[Icon: sound wave / ear]

[H2]  You're not listening to anything yet.

[Body]  Create your first topic to start tracking mentions of your brand,
        competitors, or any keyword across social media and the web.

[button]  + Create Your First Topic
[link]  Not sure where to start? Read the setup guide →
```

### 7.2 Churned Topic List (`expired` state with topics)

When `userState === 'expired'` and the user has existing topics, `<TopicListPage churned />` is rendered.

#### Auto-open UpgradeModal
On mount, the UpgradeModal opens automatically in churned mode (see Section 21 for UpgradeModal spec). `closeOnClickOutside=false`.

#### Orange Top Banner
```
[Banner — orange, full-width, not dismissible]
  Your Social Listening add-on is no longer active.
  Your N topics and data are saved — re-subscribe to access them.
  [Button]  Re-subscribe
```

Clicking "Re-subscribe" opens the UpgradeModal.

#### Dimmed Topic Cards
All topic cards are displayed at `opacity: 0.6`. Clicking any card opens the UpgradeModal instead of navigating to the topic dashboard.

#### Hidden Create Button
The "+ Create Topic" button is not shown in churned mode.

---

## 8. Create Topic Wizard

### Wizard Shell
- Full-page dedicated route `/features/listening/new`
- 4 steps in a progress stepper: Basic Info → Keywords → Sources → Alerts
- Step managed via query param: `?step=2`, `?step=3`, `?step=4`
- "Back" / "Next" buttons in footer
- "Save as Draft" button in header (always visible)

---

### Step 1 — Basic Info

**Topic Name** (required)
- Placeholder: "e.g., My Brand, Campaign: Summer 2026, Competitor: Buffer"
- Helper text: "Give this topic a recognizable name. You can always change it later."
- Validation: Required, max 80 characters, must be unique within workspace

**Description** (optional)
- Placeholder: "What are you tracking and why? (For your team's reference only)"
- Helper text: "Optional. Visible to team members who have access to this topic."

**Topic Color** (optional)
- 8 preset color swatches for visual identification in the topic list

**Visibility**
- Options: `Only Me` / `Everyone` / `Specific Members`
- When "Specific Members" selected: MultiSelect of team members appears

---

### Step 2 — Query Builder (Keywords)

**Section title:** "What do you want to monitor?"

**Three keyword field groups:**

**Include Keywords** (required — at least one)
- Tooltip: "Find mentions that contain ANY of these keywords. A mention only needs to match one to be included."
- Each keyword has a type selector: Text / Hashtag / Mention
  - `Text` — keyword appears anywhere in the post text
  - `Hashtag` — tracks this specific hashtag
  - `Mention` — tracks posts that mention this account
- Sub-label: "Mention must contain at least one of these"
- Keyword chips: violet color

**Required Keywords** (optional, AND logic)
- Tooltip: "Every mention must ALSO contain ALL of these keywords. Use this to narrow broad keywords."
- Sub-label: "Mention must contain ALL of these (optional)"
- Keyword chips: blue color

**Excluded Keywords** (optional, NOT logic)
- Tooltip: "Mentions containing ANY of these keywords will be filtered out."
- Sub-label: "Skip mentions containing any of these (optional)"
- Keyword chips: red color

**Conflict detection** (real-time):
1. Same keyword in Include AND Exclude → Error; tag highlighted red in both; Next disabled
2. Same keyword in Required AND Exclude → same error
3. Duplicate within same field → warning, tag not added (or shake animation)
4. Keyword under 3 characters (non-hashtag) → warning badge "⚠ Too short"
5. Very broad keyword with high estimated volume → amber banner above fields
6. No keywords in Include on Next click → field-level error

**AI Assist Button:**
```
[✨ button]  Get AI keyword suggestions

[When clicked — inline panel:]
  [Textarea]  "Describe in plain English what you want to monitor."
  [button]  Generate Suggestions
  [Result cards: suggested Include / Required / Exclusions with [Add All] per group]
```

---

### Step 3 — Sources

**Section title:** "Where should we look?"

**Connected Profiles section:**
Shows `MOCK_CONNECTED_PROFILES` — the user's connected social profiles. Each can be toggled to include that profile's native mentions.

**Platform Toggles:**
Pill buttons using platform brand colors when active. Split into two groups:

Social: X/Twitter · Instagram · Facebook · LinkedIn · TikTok · YouTube · Reddit · Bluesky · Pinterest · Threads

Web & Media: News Sites · Blogs & Websites · Forums · Review Sites · Podcasts

**Language MultiSelect** (searchable)
- Default: All languages
- Options: English, Spanish, Arabic, French, German, Portuguese, and 170+ more

**Content Filters:**
- Switch: "Hide reshares, reposts, and retweets"
- Switch: "Only collect verified account mentions"

---

### Step 4 — Alerts & Notifications

**Section title:** "Set up your alerts"

**Spike Alert** (toggle — ON by default)
```
[When ON:]
  Alert me when mentions are: 25% higher · 50% higher · 100% higher · Custom
  Minimum mentions to trigger: [number input, default: 5] in a 6-hour window
[sub-note]  "Spike alerts activate after 7 days of data collection."
```

**Sentiment Drop Alert** (toggle — OFF by default)
```
[When ON:]
  Alert when Sentiment Score drops by [20] points within [24h / 48h / 72h]
```

**Notification Delivery:**
- Members MultiSelect (default: current user)
- Email toggle + recipients list
- In-app notification toggle

**Wizard Footer CTA:**
```
For non-subscribers:
  [button]  Preview My Results →
  [small text]  Subscribe to unlock real-time monitoring.

For subscribers:
  [button]  Start Listening →
```

---

## 9. Preview Mode (Non-Subscriber Post-Wizard)

After a non-subscriber completes the wizard and clicks "Preview My Results":

```
[Banner — gradient violet/blue]
  🎉  Here's a preview of what "{Topic Name}" will look like once you subscribe.
      Data shown is representative — subscribe to monitor real mentions.
  [button]  Add to My Plan  ·  $99/mo

[Below banner:]
  - KPI summary bar (sample values)
  - Mention volume chart (sample data)
  - Sentiment donut (sample)
  - 10 mention cards (mock, clearly labeled "Sample mention")
    - Reply button → disabled with tooltip "Subscribe to reply to mentions"

[Upgrade benefits grid + compact pricing block]
[button]  Add to My Plan
```

---

## 10. Topic Dashboard Shell

### Layout
- Full-width container: `maxWidth: 1920px`, `padding: 0 32px`
- No Mantine `Container size="xl"` — raw Box with inline maxWidth
- White header area with `border-bottom: 1px solid #F1F3F5`
- Content area: `background: #F8F9FA`, `padding: 32px 32px`

### Header Row 1 (breadcrumb + topic name + controls)
```
[←]  [color dot] [Topic Name]  [Paused badge if applicable]  [Competitor badge if applicable]
                                                    [Refresh icon]  [Date range Select]  [Filters button]  [Export button]
```

- Date range Select: Last 7 days / Last 14 days / Last 30 days / Last 90 days / Custom
- Filters button: visible on Analytics, Mentions, AND Compare tabs
  - Shows active filter count badge when filters are applied
  - On desktop: toggles inline right-side filter panel (280px wide)
  - On mobile: opens right-side Drawer
- Export button: always visible; opens ExportModal

### Header Row 2 (Tab bar)
```
[Tabs.List — full width, flex]
  LEFT side:    Performance  |  Mentions  |  Compare
  [flex: 1 spacer]
  RIGHT side:   Reports  |  Settings
```

The tab bar uses a flex spacer (`<Box style={{ flex: 1 }} />`) between left and right tab groups.

Tab labels in code:
- `analytics` route → label "Performance"
- `mentions` route → label "Mentions"
- `compare` route → label "Compare"
- `reports` route → label "Reports"
- `settings` route → label "Settings"

### ExportModal
Rendered in the `[topicId]/layout.tsx`. Available from any tab by clicking the Export button. Not scoped to the Analytics page.

### Filter Panel
When filters are active on desktop (Analytics, Mentions, Compare tabs), the layout shifts to a 2-column grid: `1fr 280px`. The right column contains `<FilterSidebar />` in a sticky card.

### Demo Banner
When `topic.isDemo === true`, a state-dependent banner is shown above the header (see Section 6).

---

## 11. Filter Bar / Filter Sidebar

The filter system uses Zustand shared state (`topicFilters`) so filters persist and apply consistently across Analytics and Mentions tabs.

### Filter Trigger
A "Filters" button in the header row:
- `SlidersHorizontal` icon + "Filters" label
- Active count badge (violet, filled, circle) showing number of active filter groups
- Clicking toggles the filter panel open/closed
- Button uses `variant="light" color="violet"` when panel is open, `variant="default"` when closed

### Filter Panel Contents (FilterSidebar)
The sidebar has sections for:

| Section | Control | Options |
|---|---|---|
| Search | TextInput (always visible) | Search within mention text |
| Networks | Checkbox group | X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Bluesky, Pinterest, Threads, News, Blogs, Forums |
| Sentiment | Checkbox group | Positive, Neutral, Negative |
| Content Type | Checkbox group | Text, Image, Video, Link, GIF, Mixed |
| Message Type | Checkbox group | Post, Comment, Share/Repost |
| Search Options | Operator select | Any word / All words / Exact phrase |
| Language | MultiSelect (searchable) | All languages |
| Country | MultiSelect (searchable) | All countries |

### Filter State (Zustand `topicFilters`)
```typescript
interface TopicFilters {
  searchTokens: string[]
  networks: string[]
  sentiments: string[]
  contentTypes: string[]
  messageTypes: string[]
  languages: string[]
  countries: string[]
}
```

Active filter count = number of filter groups with at least one value selected (search tokens count as one group).

---

## 12. KPI Cards Grid

Shown at the top of the Analytics tab, above the Performance/Insights sub-tab switcher.

Above the grid: `[Key Metrics label]  [AI Insights overview button]`

7 cards in a responsive grid (2 → 3 → 4 columns based on viewport):

| # | Label | Icon | Tooltip |
|---|---|---|---|
| 1 | Mentions | MessageSquare | Total times your keywords were mentioned in the selected period |
| 2 | Reach | Eye | Estimated total audience size that could have seen mentions |
| 3 | Engagement | ThumbsUp | Total interactions across all collected mentions |
| 4 | Avg Engagement | TrendingUp | Average engagement per mention (total ÷ total mentions) |
| 5 | Impressions | Layers | Estimated total number of times mentions were displayed |
| 6 | Sentiment Score | Star | Composite 0–100 brand health score |
| 7 | Unique Authors | Users | Number of distinct accounts that mentioned your keywords |

**Card style:** white background, `border: 1px solid #F1F3F5`, consistent styling across all 7 cards (no elevated or highlighted card). Each card shows: label + info tooltip icon | large value | delta badge + "vs prev period" text.

There is no "Share of Voice" KPI card — SOV is shown in the Compare tab context only.

---

## 13. Analytics Tab (Performance sub-tab)

### Sub-Tab Switcher
Below the KPI grid, a SegmentedControl switches between two sub-views:
- **Performance** — time-series charts for all key metrics
- **Insights** — breakdown tables and structured analysis

The KPI grid is always visible above the sub-tab switcher regardless of which sub-tab is active.

---

### 13.1 Performance Sub-Tab

Widgets displayed top-to-bottom:

#### 1. Mention Volume
- Full width
- Chart type: AreaChart (Recharts)
- Granularity dropdown: Day / Week / Month
- Breakdown dropdown: None / Platform / Source / Content Type / Message Type / Sentiment
- Right panel: shows aggregate stats or breakdown bar list depending on active breakdown
- Tooltip: "Shows how many times your keywords were mentioned. Spikes often indicate news coverage, viral posts, or campaigns."

#### 2. Reach Over Time
- Full width
- Chart type: AreaChart
- Granularity: Day / Week
- Breakdown options (same as Mention Volume)
- Right panel
- Tooltip: "Estimated total audience that saw mentions. Calculated from author follower counts at time of mention."

#### 3. Engagement Over Time
- Full width
- Chart type: AreaChart
- Granularity: Day / Week
- Breakdown options
- Right panel
- Tooltip: "Total user interactions on mentions of your keywords."

#### 4. Potential Impressions Over Time
- Full width
- Chart type: AreaChart
- Granularity: Day / Week
- Breakdown options
- Right panel
- Tooltip: "Estimated total times mentions were displayed to users."

#### 5. Sentiment — 2-column grid
Left: **Sentiment Over Time**
- AreaChart, stacked (positive/neutral/negative layers)
- Granularity: Day / Week / Month
- Tooltip: "Tracks how the tone of conversation shifts over time."

Right: **Sentiment Breakdown**
- Donut chart (3 segments: positive/neutral/negative)
- Center: Sentiment Score (e.g., "72")
- Legend with percentages
- "Include neutral" Switch — toggles whether neutral mentions affect the score
- Tooltip: "Your overall Sentiment Score for this period."

#### 6. Sentiment by Platform
- Full width
- Chart type: Stacked BarChart (bars per platform, colored by sentiment)
- Tooltip: "How does sentiment differ across platforms? Reddit tends to be more critical; Instagram tends to be more promotional."

---

### 13.2 Insights Sub-Tab

Widgets displayed top-to-bottom:

#### 1. Network Breakdown
- Full width
- Sortable table component: `NetworkBreakdownTable`
- Columns: Platform | Mentions | Followers | Reach | Engagement | Impressions
- Sortable by any column (click header)
- Includes AI Insights button in card header

#### 2. Influencers
- Full width
- Sortable table
- Columns: Rank | Avatar | Handle | Platform badge | Followers | Mentions | Reach
- Includes AI Insights button in card header
- Tooltip: "Accounts with the largest potential audience among those who mentioned your keywords."

#### 3. 2-column grid
Left: **Geographic Breakdown**
- Horizontal BarChart (top countries by mention share)
- Includes AI Insights button

Right: **Context of Discussion / Word Cloud**
- Flex-wrapped words, font-size proportional to frequency
- Includes AI Insights button
- Tooltip: "The bigger and bolder the word, the more frequently it appears in mentions."

---

## 14. AI Insights Buttons

Every analytics widget has a Sparkles icon button (violet, Mantine ActionIcon) in its card header.

### ChartCard widgets
The `ChartCard` component accepts an `aiInsights` prop (string array of 2–3 bullet points). When provided, an ActionIcon with `<Sparkles size={14} />` renders before the dropdown controls in the card header.

The `ChartCard` also accepts `isStatic` prop — when true, all dropdown controls are hidden but the AI Insights button still shows. Used for static report previews.

### Non-ChartCard widgets
For `NetworkBreakdownTable`, Influencers panel, and Word Cloud: the AI Insights button is in the internal card header.

### Popover content
Clicking the button opens a Mantine `Popover` (no portal needed since it's within the card) showing:
```
[Popover — max-width 260px, violet-themed border]
  [Sparkles icon]  AI Insights
  • [insight 1 specific to this chart]
  • [insight 2]
  • [insight 3 — optional]
```

Insights are pre-written per widget in the prototype (mocked). In production, they will be generated by the AI agent pipeline.

---

## 15. Mentions Feed Tab

### Toolbar
```
[Search input: "Search mentions..."]
[Filters button with count badge]    (same filter sidebar as Analytics)
[Right side: Sort dropdown]  Newest first · Oldest first · Most engaged · Most influential
[Right side: Export button]  opens ExportModal
[Right side: result count]  N results
```

Filters are shared with the Analytics tab via `topicFilters` Zustand state.

### Mention Card

```
[Platform icon]  [Author name]  @handle  ·  [Follower count] 12.4k followers
[Relative time: "3 hours ago"]  ·  [Absolute date on hover: "Mar 1, 2026 at 14:32"]
[Sentiment badge: ● Positive]  ·  [Content type icon]

[Mention text — up to 3 lines]
  "Just switched our agency to ContentStudio..."
  [Show more] if truncated

[Keyword match highlight]  — matched keywords bold in text

[Engagement row]
  ♡ 142  💬 38  🔁 91  👁 2.4k

[Action row]
  [Reply]  [Save]  [Mark as reviewed ✓]  [··· more: Report as irrelevant / Copy link]
```

**Reply button behavior:**
- Subscribed + platform connected → opens Inbox composer
- Subscribed + platform not connected → "Your [Platform] account isn't connected. [Connect it in Settings →]"
- Platform doesn't support replies (Reddit, Web) → external link → "View on [Platform] →"
- Unsubscribed → upgrade modal

### Empty States

**No mentions match filters:**
```
[Icon: magnifying glass with X]
[H3]  No mentions found
[Body]  No mentions match your current filters in this date range.
        Try expanding the date range, adjusting filters, or checking your keywords.
[button]  Clear all filters
[link]  Edit topic keywords →
```

**Topic just created:**
```
[Icon: radar / sound wave animation]
[H3]  We're listening...
[Body]  We're collecting your first mentions now. Social media platforms
        typically appear within a few minutes. Web and news sources
        can take up to 1 hour.
[button]  Edit topic settings
```

---

## 16. Compare Tab

### Purpose
Compare up to 5 topics side-by-side (brand topic vs competitor topics, or campaign vs campaign).

### Layout

#### Topic Selector Row
```
[Your topic: ContentStudio]  [+ Add topic to compare ▾]  ...  (up to 4 more)
Each added topic: color-coded pill with topic name + [x] to remove
```

When fewer than 2 topics are selected, a placeholder CTA is shown:
```
[Illustration / icon]
[H3]  Compare topics side-by-side
[Body]  Add at least one more topic to compare share of voice,
        sentiment, and mention volume across your brand and competitors.
[button]  + Add topic to compare
```

#### SOV Chart (2+ topics selected)
```
[Title]  Share of Voice
[Tooltip (i)]  "Your brand's share of total mentions across all topics in this comparison."
[Donut chart — each slice = one topic, colored by topic color]
[Legend: Topic name · % share · Mention count]
```

#### Side-by-Side KPI Table
```
                        ContentStudio    Buffer     Hootsuite
Mentions                    1,247          3,891       2,104
Reach                       4.2M          14.1M        8.7M
Engagement                 89,420         312k         178k
Sentiment Score             72/100         61/100       68/100
Positive %                   52%            39%         47%
Negative %                   14%            22%         18%
Unique Authors                 892          2,841       1,543
```
Star (★) marks the highest value per row.

#### Side-by-Side Charts
- Volume over time (multi-line, one line per topic, colored by topic)
- Sentiment Score trend (multi-line)
- Platform distribution (grouped bar)

#### Compare Periods Toggle
```
[Toggle]  Compare current period to previous period
[When ON: shows Δ vs prev period column for each KPI]
```

---

## 17. Reports Tab

Two sub-tabs: **Downloaded Reports** | **Scheduled Reports**

The Reports tab is accessible from the RIGHT side of the tab bar (alongside Settings).

### 17.1 Downloaded Reports Sub-Tab

**Header row:**
```
[H1]  Reports
[Subtext]  Download snapshots or set up automatic scheduled deliveries.
[Right: Button]  Generate Report → opens ExportModal
```

**Report list:**
Each row shows:
- Format icon (FileText) in ThemeIcon — red for PDF, green for CSV
- Report name + format badge (PDF/CSV)
- Status badge: "Generating…" (blue) | "Failed" (red with AlertCircle icon)
- Date range + generated timestamp + file size

**Row actions:**
- Status `ready`: Eye ActionIcon (opens Report Preview Drawer) + Download button (violet light)
- Status `failed`: Retry button (gray)
- All rows: Trash ActionIcon

**Empty state:**
```
[ThemeIcon: FileDown, gray, large]
[H3]  No downloaded reports yet
[Body]  Use the Export button on any tab to generate a PDF or CSV report.
```

### 17.2 Report Preview Drawer

Triggered by the Eye icon on a downloaded report row.

- Mantine `Drawer`, `position="right"`, `size="xl"`, `padding=0`
- Header: FileText icon (violet) + report name + date range
- Scrollable content area (`ScrollArea h="calc(100vh - 70px)"`)

**Static widgets displayed (in order):**

1. **Key Metrics** — 7 KPI cards (SimpleGrid cols 2/sm:4), values only, no delta arrows
2. **Mention Volume** — AreaChart, `isStatic={true}` on ChartCard (no dropdowns, AI button shows)
3. **2-col grid:**
   - Sentiment Breakdown (donut with sentiment score in center + legend)
   - Platform Breakdown (horizontal BarChart, platform colors)
4. **2-col grid:**
   - Geographic Breakdown (horizontal BarChart, % values)
   - Top Hashtags (horizontal BarChart, mention counts)

All charts use `ChartCard` with `isStatic={true}`. AI Insights buttons remain visible in static mode.

### 17.3 Scheduled Reports Sub-Tab

**Header row:**
```
[H1]  Reports
[Subtext]  ...
[Right: Button]  New Schedule → opens NewScheduleModal
```

**Schedule list:**
Each row shows:
- Clock ThemeIcon — violet if active, gray if paused
- Schedule name + format badge
- "Paused" outline badge if `active=false`
- Frequency + day + time (e.g., "Weekly · Monday · 08:00")
- Recipients (first 2 emails shown + "+N more" if over 2)
- "Next: In 7 days (Mar 9)" (violet text, shown only if active)
- Right side: enable/disable Switch + More menu (Edit | Delete)

**Empty state:**
```
[ThemeIcon: Calendar, gray, large]
[H3]  No scheduled reports
[Body]  Automatically send PDF or CSV reports to your team on a recurring schedule.
[button]  Create Schedule
```

### 17.4 NewScheduleModal

Mantine Modal, `size="md"`, centered.

```
[Modal title]  New Scheduled Report

Fields:
  Report name: TextInput
  Format (Select): PDF | CSV
  Frequency (Select): Daily | Weekly | Monthly
  Send on (Select, conditional):
    - Weekly: Monday / Tuesday / Wednesday / Thursday / Friday
    - Monthly: 1st / 2nd / 15th / Last day
    - Daily: (field hidden)
  Time (Select): 06:00 / 07:00 / 08:00 / 09:00 / 10:00 / 12:00 / 17:00 / 18:00
  Recipients (TextInput): "alice@company.com, bob@company.com"
    description: "Separate multiple addresses with commas"

Footer:
  [Cancel]  [Schedule Report — disabled until name is filled]
```

---

## 18. Export Modal (3 Report Types)

The ExportModal is rendered in `[topicId]/layout.tsx` and is available from any tab via the Export button in the header. It is NOT scoped to the Analytics page.

### Modal Structure
Mantine `Modal`, `size="xl"`, with tabs inside.

**Three report types** (radio-style selector or segmented control at top):
1. **Performance & Mentions** — default selection
2. **Compare Topics** — disabled (with tooltip) if no topics have been added in the Compare tab
3. **Compare Periods** — compares current period vs previous period

Switching report type resets section selection to defaults for that type.

### Tabs inside the modal:
- **Download** — configure and generate
- **Schedule** — set recurring delivery
- **Share** — generate shareable link

### Download Tab

**Date range:** Last 7 days / Last 14 days / Last 30 days / Last 90 days / Custom

**Format toggle:** PDF | CSV

**Sections (checkboxes, defaults depend on report type):**

For **Performance & Mentions:**
- ☑ KPI Summary
- ☑ Mention Volume
- ☑ Reach Over Time
- ☑ Engagement Over Time
- ☑ Sentiment Trend
- ☑ Sentiment Breakdown
- ☑ Platform Breakdown
- ☑ Top Countries
- ☑ Top Hashtags
- ☑ Top Mentions (top 5 by engagement)
- ☐ Influencers
- ☐ Word Cloud

For **Compare Topics:**
- ☑ Share of Voice
- ☑ Side-by-Side KPI Table
- ☑ Volume Comparison
- ☑ Sentiment Comparison

For **Compare Periods:**
- ☑ KPI Summary with period delta
- ☑ Volume Comparison (current vs previous)
- ☑ Sentiment Comparison

**Footer:** [Cancel] [Preview Report] [Generate & Download]

### Schedule Tab
```
[Toggle]  Schedule recurring delivery
If ON:
  Frequency: Daily / Weekly / Monthly
  On: [day of week or day of month — conditional]
  At: [Time picker]
  Recipients: [Member picker + add external email]
  "Send even if no new mentions": [Toggle — OFF]
  [Note: "If off, the email is skipped if mention count is 0."]
```

### Share Tab
```
[Title]  Share this report
[Button]  Generate shareable link
[Once generated: copy link + expiry notice]
```

---

## 19. Settings Tab

### Layout
Single-page scrollable settings. No left nav or inner tab navigation. Sections are collapsible Accordions but pre-expanded.

### Section 1 — Basic Info
- **Topic Name** (TextInput, editable)
- **Description** (Textarea, editable)
- **Topic Color** (ColorPicker, 8 swatches)
- **Visibility** (SegmentedControl): Only Me | Everyone | Specific Members
  - When "Specific Members": MultiSelect of MOCK_TEAM_MEMBERS appears

### Section 2 — Keywords (editable, pre-expanded)
Full editable KeywordField interface:
- Three `KeywordField` sections: Include / Required / Exclude
- Each field: type selector (Text/Hashtag/Mention) + TextInput + Add button + Enter-key support
- Keyword chips: violet=include, blue=required, red=exclude; X button to remove
- Conflict detection: errors shown inline (same rules as wizard step 2)
- "Save Changes" button

### Section 3 — Sources (editable, pre-expanded)
- **PlatformToggle**: pill buttons using platform brand colors when active
  - Social group: X/Twitter, Instagram, Facebook, LinkedIn, TikTok, YouTube, Reddit, Bluesky, Pinterest, Threads
  - Web group: News Sites, Blogs & Websites, Forums, Review Sites, Podcasts
- **Language MultiSelect** (searchable)
- Switch: "Hide reshares, reposts, and retweets"
- Switch: "Verified only"

### Section 4 — Alerts
- **Spike alert:** toggle + threshold selector (25%/50%/100%/custom) + min mentions input
- **Sentiment drop alert:** toggle + drop threshold + window (24h/48h/72h)
- **Notification channels:**
  - Members MultiSelect (who gets notified)
  - Email toggle + recipients input
  - In-app notifications toggle

Note: Scheduled reports configuration has been removed from Settings. It is now managed exclusively in the Reports tab.

### Section 5 — Danger Zone
```
[Title — red]  Danger Zone

[Pause this topic]
  "Pausing stops collecting new mentions but keeps all existing data."
  [button]  Pause Topic
  (If paused: "Resume Topic")

[Delete this topic]
  "Permanently deletes this topic and all collected mentions. This cannot be undone."
  [button]  Delete Topic → opens confirmation modal:
    "Type '{topic name}' to confirm deletion."
    Lists what will be deleted (mentions count, alerts, saved mentions)
    [Cancel] [Delete Forever — red]
```

---

## 20. UpgradeModal

Reusable modal at `components/shared/UpgradeModal.tsx`. Accepts a `feature?` prop and a `churned?` boolean variant.

### Standard Mode (locked/trial states)
```
[Modal — centered, size="md", closeOnClickOutside=true]
  [ThemeIcon — violet, large]  [Sparkles or relevant icon]

  [H3]  Unlock Social Listening
  [Body]  Monitor your brand across all platforms with real-time alerts,
          sentiment analysis, and AI-powered insights.

  [Compact pricing block]  $99/mo or $79/mo (annual)

  [Button — gradient violet→indigo]  Add to My Plan
  [Link]  Maybe later
```

### Churned Mode (`churned=true`, expired users)
```
[Modal — centered, size="md", closeOnClickOutside=false]
  [ThemeIcon — orange]  [WifiOff icon]

  [H3]  Social Listening Add-on Expired

  [Alert — orange]
    Your subscription has lapsed, but your N topics and data are saved.
    Re-subscribe to restore full access.

  [Button — gradient orange→red OR violet→indigo]  Re-add to my plan
  [Link]  Contact support
```

---

## 21. Prototype Controls (DebugToggle)

Fixed floating panel at `position: fixed; bottom: 24px; right: 24px; z-index: 1000`.

A circular button with `<Settings2 />` icon opens the panel. Dark navy background (`#1a1a2e`).

### State Option Cards (4 total):
Each card is clickable and shows: colored dot + label + description

| State | Dot Color | Label | Description |
|---|---|---|---|
| `trial` | #F59E0B (orange) | Trial | "On trial, no add-on → landing page" |
| `locked` | #6B7280 (gray) | Locked | "Has plan, no add-on → landing page" |
| `unlocked` | #7C3AED (violet) | Unlocked | "Has add-on → topic list" |
| `expired` | #EA580C (orange-red) | Expired | "Had add-on, cancelled → churned list (if topics exist)" |

Active state card highlighted with `background: {color}22` and `border: 1px solid {color}`.

**Reset button:** "Reset to defaults" → sets state to `locked`.

There is no plan tier selector. Social Listening is a flat add-on with no tiers.

---

## 22. All Empty, Loading, and Error States

### Loading States
Every chart and data section has a skeleton loading state (gray animated placeholder bars) while data is being fetched. The skeleton matches the shape of the actual chart.

The prototype simulates ~900ms loading delay using a `useState` initialization pattern (not `useEffect`) to avoid SSR issues.

Topic list loading: 3 skeleton topic cards.

### Error States

**API error on chart load:**
```
[Error card replacing chart]
  ⚠  We couldn't load this chart.
  [Retry] button
  [small text]  If this keeps happening, check our status page →
```

**Platform disconnected:**
```
[Amber banner on Mentions feed]
  ⚠  Your Instagram connection expired and mentions may be incomplete.
     [Reconnect Instagram →]  ·  [Dismiss]
```

**API rate limit hit:**
```
[Info banner — blue]
  ℹ  X/Twitter data may be delayed right now due to API rate limits.
     We'll resume collection automatically. [Learn more →]
```

**Mention older than retention window:**
```
[Overlay on chart or feed]
  🔒  This data is older than your history window.
      Re-subscribe to access your saved data.
  [Re-subscribe]
```

**Topic paused:**
```
[Gray banner on topic dashboard]
  ⏸  This topic is paused — no new mentions are being collected.
     [Resume]
```

---

## 23. Tooltip Copy — Key Definitions

All (i) tooltips answer: What is this? / What does it tell me? / What should I do with it?

| Metric | Tooltip |
|---|---|
| Mentions | "Total number of times your keywords were mentioned in the selected period. Includes social posts, comments, news articles, and web content." |
| Reach | "Estimated total audience size that could have seen mentions of your keywords. Calculated from the follower/subscriber counts of accounts that mentioned you." |
| Engagement | "Total interactions (likes, comments, shares, views) across all collected mentions. Measures how much people are actively engaging with content about your brand." |
| Avg. Engagement | "Average engagement per mention (total engagement ÷ total mentions). Higher means each mention generates more interaction on average." |
| Impressions | "Estimated total number of times mentions were displayed to users (reach × average display frequency). Indicates potential exposure, not confirmed views." |
| Sentiment Score | "A composite score from 0–100 measuring overall sentiment. 100 = all positive. 50 = balanced. 0 = all negative. Calculated from the ratio of positive to negative mentions, weighted by engagement." |
| Unique Authors | "Number of distinct people or accounts that mentioned your keywords. High author count = broad organic conversation. Low count = concentrated mentions." |
| Share of Voice | "Your brand's percentage of total mentions across all tracked topics (your brand + competitors combined). Only meaningful when you have at least one competitor topic added." |

---

## 24. Conflict Detection — Complete Rules

| Trigger | Level | Message | UX Behavior |
|---|---|---|---|
| Same keyword in Include AND Exclude | Error | "'{keyword}' can't be both included and excluded. Remove it from one field to continue." | Tag highlighted red in both fields; Next/Save disabled |
| Same keyword in Required AND Exclude | Error | Same as above | Same |
| Duplicate within same field | Warning | "'{keyword}' is already in this list." | Tag shake animation, not added |
| Keyword under 3 characters (not a hashtag) | Warning | "'{keyword}' is very short and may generate too many unrelated results." | Tag shows ⚠ badge; does not block |
| Very broad keyword + high estimated volume | Warning | "'{keyword}' is very broad. Add Required Keywords to narrow your results." | Amber banner; does not block |
| No keywords in Include field on Next | Error | "Add at least one keyword, hashtag, or handle to monitor." | Field highlighted red; Next blocked |
| Keyword with spaces typed as single hashtag | Suggestion | "Did you mean two separate hashtags: #{word1} and #{word2}?" | Non-blocking suggestion below tag |

---

## 25. Prototype State (Zustand Store)

### `lib/types.ts`
```typescript
type UserState = 'trial' | 'locked' | 'unlocked' | 'expired';

interface TopicFilters {
  searchTokens: string[]
  networks: string[]
  sentiments: string[]
  contentTypes: string[]
  messageTypes: string[]
  languages: string[]
  countries: string[]
}

interface AlertConfig {
  spikeEnabled: boolean
  spikeThreshold: number
  spikeMinMentions: number
  spikeWindow: string
  sentimentDropEnabled: boolean
  sentimentDropThreshold: number
  sentimentDropWindow: string
  notifyMembers: string[]
  emailEnabled: boolean
  alertEmailRecipients: string[]  // required field
  inAppEnabled: boolean
}

interface Keyword {
  id: string
  value: string
  type: KeywordType  // 'text' | 'hashtag' | 'mention'
}
```

### `lib/store.ts` key fields
```typescript
interface ListeningStore {
  // Auth / user state
  userState: UserState
  isSubscribed: boolean         // computed: userState === 'unlocked'
  setUserState: (s: UserState) => void

  // Topics
  topics: Topic[]
  getTopicById: (id: string) => Topic | undefined
  selectedTopicId: string | null
  setSelectedTopic: (id: string) => void

  // Wizard
  wizardStep: 1 | 2 | 3 | 4
  wizardDraft: Partial<TopicDraft>
  setWizardStep: (step: number) => void
  updateWizardDraft: (patch: Partial<TopicDraft>) => void
  submitWizard: () => void

  // Shared filters (Analytics + Mentions)
  topicFilters: TopicFilters
  setTopicFilters: (f: Partial<TopicFilters>) => void
  resetTopicFilters: () => void
  filterSidebarOpen: boolean
  setFilterSidebarOpen: (v: boolean) => void

  // UI
  dateRange: DateRange
  setDateRange: (r: DateRange) => void
  activeTab: string
  setActiveTab: (t: string) => void
  compareTopicIds: string[]
  addCompareTopicId: (id: string) => void
  removeCompareTopicId: (id: string) => void

  // Export
  exportDialogOpen: boolean
  setExportDialogOpen: (v: boolean) => void

  // Analytics
  getAnalytics: (topicId: string) => TopicAnalytics | undefined
}
```

### `lib/mock-data.ts`
- Pre-built topics including `topic-demo` (always-accessible demo topic)
- `MOCK_ANALYTICS['topic-demo']` confirmed to exist
- ~200 mock mentions with realistic authors, platform distribution, sentiment, engagement
- Chart data arrays for all date ranges (7/30/90 days)
- Deterministic values (no `Math.random()`) to avoid SSR hydration mismatches

### `lib/constants.ts`
Key exports:
- `MOCK_TEAM_MEMBERS` — team member list for visibility/notification selectors
- `MOCK_CONNECTED_PROFILES` — connected social profiles for wizard step 3
- `MOCK_FB_PAGES` — Facebook page list
- `TOPIC_COLORS` — 8 preset topic colors with `id`, `label`, `hex`
- `PLATFORMS` — platform definitions
- `LANGUAGES` — full language list
- `DATE_RANGE_OPTIONS` — Select data for date range picker
- `SENTIMENT_COLORS` — `{ positive, neutral, negative }`

---

## 26. Implementation Notes (Prototype)

- **Charts:** Recharts 3 — `AreaChart`, `BarChart`, `PieChart`. Mantine `Tooltip` is imported as `Tooltip` and Recharts `Tooltip` is aliased as `RechartTooltip` in files that use both.
- **UI:** Mantine UI 8 — `Tabs`, `Modal`, `Drawer`, `Select`, `MultiSelect`, `TextInput`, `Switch`, `Badge`, `Progress`, `Table`, `Tooltip`, `ActionIcon`, `Popover`, `SegmentedControl`
- **Animations:** Framer Motion for wizard step transitions and mention card entrance animations
- **Loading simulation:** `useState` init pattern (900ms simulated delay) instead of `useEffect` to avoid SSR mismatch
- **Color theming:** Use ContentStudio color tokens (`text-primary-cs-500`, etc.) in production — prototype uses inline hex values for speed
- **ChartCard:** Reusable wrapper accepting `title`, `subtitle`, `tooltip`, `aiInsights`, `isStatic`, `dropdownOptions`, `children`
- **UpgradeModal:** Accepts `feature?` (string) and `churned?` (boolean); reusable across all gating scenarios

---

## 27. Verification Checklist (Prototype Review)

1. Run `npm run dev` from `cs-prototypes/`
2. Navigate to `/features/listening` — landing page renders (locked state by default)
3. Verify trial state: DebugToggle → Trial → orange alert banner visible
4. Click "Open Demo Topic" → redirects to `/features/listing/topic-demo/analytics`
5. Verify demo banner shows correct variant for trial vs locked state
6. Toggle to "Unlocked" → topic list visible with topic cards
7. Click a topic → dashboard loads at `/analytics` (Performance tab)
8. Verify 7 KPI cards above sub-tab switcher
9. Switch Performance ↔ Insights sub-tabs
10. Click AI Insights (Sparkles) on any chart → popover shows bullet points
11. Click Filters button → filter sidebar appears (desktop: inline panel; mobile: drawer)
12. Switch to Mentions tab → filter state shared (same active filters)
13. Switch to Compare tab → topic selector visible; add topic → SOV donut appears
14. Navigate to Reports tab → Downloaded Reports sub-tab shows report list
15. Click Eye icon on a ready report → Report Preview Drawer opens with static charts
16. Click Scheduled Reports sub-tab → schedule list with toggles
17. Click "New Schedule" → NewScheduleModal opens with conditional day selector
18. Click Export button (any tab) → ExportModal opens with 3 report types
19. Navigate to Settings → all 5 sections visible and editable
20. Toggle to "Expired" with topics → churned state: UpgradeModal auto-opens, orange banner, dimmed cards
21. Toggle to "Expired" with no topics → landing page shown
22. DebugToggle Reset → returns to "Locked" state

---

## 28. Next Steps

1. Update `docs/features/social-listening/03-prd.md` with finalized spec decisions (new user states, flat pricing, tab restructure, Reports tab, AI Insights buttons)
2. Run `/feature` Step 4 (Epic + Stories) using this finalized spec
3. Create Shortcut stories for: Landing Page updates, UserState logic, KPI Cards Grid, Analytics sub-tabs, Filter Bar/Drawer, Reports Tab, Export Modal redesign, Demo redirect, UpgradeModal churned mode, Settings tab restructure, ChartCard AI Insights
