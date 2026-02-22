# Analytics APIs — PHP to Golang Migration: Scope & Inventory

## Overview

All analytics APIs currently run in the Laravel 10 PHP monolith (`contentstudio-backend`). The goal is to migrate these to Golang for better performance, memory efficiency, and independent scalability. This document catalogs the full analytics surface that needs to be migrated.

## Analytics PHP Surface — By the Numbers

- **154 POST route endpoints** in `contentstudio-backend/routes/web/analytics.php`
- **24 controller files** in `contentstudio-backend/app/Http/Controllers/Analytics/`
- **8 platform-specific controller groups** + cross-platform overview, AI insights, reports, dashboard, share links

## Platform Controllers & Their Routes

| Platform | Controller | Route Prefix | ~Endpoints |
|----------|-----------|-------------|------------|
| Facebook | `FacebookController` | `analytics/overview/facebook/` | 14 |
| Facebook Competitors | `FacebookCompetitorController` | `analytics/overview/facebook/competitor/` | 10+ |
| Instagram | `InstagramController` | `analytics/overview/instagram/` | 13 |
| Instagram Competitors | `InstagramCompetitorController` | `analytics/overview/instagram/competitor/` | ~10 |
| LinkedIn | `LinkedinController` | `analytics/overview/linkedin/` | 8 |
| YouTube | `YoutubeController` | `analytics/overview/youtube/` | 14 |
| TikTok | `TiktokController` | `analytics/overview/tiktok/` | 6 |
| Pinterest | `PinterestController` | `analytics/overview/pinterest/` | 10 |
| Twitter/X | `TwitterController` | `analytics/overview/twitter/` + settings | 12 |
| Overview (cross-platform) | `OverviewController` + `OverviewV2Controller` | `analytics/overview/` | 11 |
| AI Insights | 7 controllers (one per platform + overview) | `analytics/overview/{platform}/ai_insights` | 7 |
| Reports | `AnalyticsReports` + `ScheduleReports` | `analytics/reports/` | 7 |
| Dashboard | `DashboardAnalytics` | root | 3 |
| Campaign/Label Analytics | `CampaignLabelAnalyticsController` | `analytics/` | varies |
| Share Links | `AnalyticsShareLinkController` | `api/analytics/share-link/` | 5 |

## Key Files (PHP Side)

### Routes
- `contentstudio-backend/routes/web/analytics.php` — all 154 analytics endpoints

### Controllers
- `contentstudio-backend/app/Http/Controllers/Analytics/Analyze/` — platform-specific controllers (Facebook, Instagram, LinkedIn, YouTube, TikTok, Pinterest, Twitter/X, competitors, overview)
- `contentstudio-backend/app/Http/Controllers/Analytics/AI/` — AI insights controllers (7 files, one per platform + overview)
- `contentstudio-backend/app/Http/Controllers/Analytics/Analytics/` — reports (AnalyticsReports, ScheduleReports), dashboard (DashboardAnalytics)
- `contentstudio-backend/app/Http/Controllers/Analytics/AnalyticsShareLinkController.php` — share link public/authenticated routes
- `contentstudio-backend/app/Http/Controllers/Analytics/Analyze/AnalyticsJobController.php` — job triggering

### Data Layer
- Analytics data stored in **MongoDB**
- Data fetching jobs run via **Laravel Horizon** (Redis queues) — pull data from social platform APIs and store aggregated results
- Some analytics use **Elasticsearch** for time-series aggregations

### Auth / Middleware Patterns
- Authenticated routes use `auth` + `set.locale` middleware
- Share link routes use `shareable.optional` + `shareable.auth` — custom middleware for public analytics links with optional password protection
- Some routes behind `PermissionMiddleware` for workspace-level permissions

## Frontend Integration

- The Vue SPA (`contentstudio-frontend`) calls these endpoints directly
- Analytics module: `contentstudio-frontend/src/modules/analytics/` (not inventoried in detail — frontend changes will be needed to point to new Go endpoints when ready)

## What Needs to Migrate

Every endpoint listed above needs a Golang equivalent that:
1. Accepts the same request payload structure (or a documented new one with frontend changes)
2. Reads from the same MongoDB collections / Elasticsearch indices
3. Returns the same response shape (or a documented new one with frontend changes)
4. Respects the same auth/permission model (session auth, API key auth, share link auth)
5. Handles the same edge cases (empty data, date ranges, timezone conversions)
