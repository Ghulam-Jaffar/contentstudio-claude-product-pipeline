# Research: Publishing API v1.7 — Labels & Campaigns Support

## Current State

### Fetch Posts (GET) — Labels & Campaigns ALREADY added
Commit `437832592` (by Arslan) added `labels`, `campaign`, and `created_by` to `PostResource`. This is on `features`/`develop` already.

- `PostResource.php` now calls `PlannerViewService::getLabels()` and `PlannerViewService::getFolderData()` to resolve label IDs and folder IDs into `{id, name, color}` objects.
- The GET response includes: `labels: [{id, name, color}]` and `campaign: {id, name, color}`.

### Create Post (POST) — Labels & Campaigns NOT supported yet
The `POST /api/v1/workspaces/{workspace_id}/posts` endpoint in `PostController@store` (line 492) does NOT accept `labels` or `campaign_id` parameters.

- `transformApiPayloadToInternal()` (line 606) builds the internal `socialPostConfig` but never sets `labels` or `folderId` from the public payload.
- The internal posting system (`processSocialShare`) supports both — the internal payload uses `labels: [array of IDs]` and `folderId: string`.

### List Labels — No V1 endpoint exists
- Internal controller: `contentstudio-backend/app/Http/Controllers/Planner/LabelController.php`
- Internal route exists in `routes/web.php` for authenticated web users but NOT exposed in `routes/api/v1.php`.

### List Campaigns (Folders) — No V1 endpoint exists
- Internal controller: `contentstudio-backend/app/Http/Controllers/Composer/FolderController.php`
- Internal route exists for authenticated web users but NOT exposed in `routes/api/v1.php`.

### Integration Apps — None currently use labels/campaigns
No references to Zapier, Make.com, n8n, GPT, Claude extension, or MCP found in the BE codebase for label/campaign support. These are external apps that consume the Publishing API — once the API supports it, each app needs to be updated to expose the new fields.

## What Needs to Change

### BE Stories (Publishing API)
1. **New endpoint: `GET /api/v1/workspaces/{workspace_id}/labels`** — List all labels in a workspace (id, name, color)
2. **New endpoint: `GET /api/v1/workspaces/{workspace_id}/campaigns`** — List all campaigns/folders in a workspace (id, name, color)
3. **Update `POST /api/v1/workspaces/{workspace_id}/posts`** — Accept optional `labels` (array of label IDs) and `campaign_id` (string) in the create post payload, pass them through to the internal posting system

### Integration App Stories
4. **Zapier** — Update the Zapier app to expose labels/campaigns dropdowns in the "Create Post" action
5. **Make.com** — Update the Make.com app module to include labels/campaigns fields
6. **n8n** — Update the n8n node to include labels/campaigns fields
7. **GPT App** — Update the GPT action schema to include labels/campaigns parameters
8. **Claude Extension** — Update the Claude MCP extension to include labels/campaigns in post creation
9. **MCP** — Update the MCP server tools to support labels/campaigns
10. **Smart Scheduling via AI** — Update the AI scheduling feature to support assigning labels/campaigns

## Files Involved

### BE (Publishing API)
- `contentstudio-backend/routes/api/v1.php` — Add new routes
- `contentstudio-backend/app/Http/Controllers/Api/V1/PostController.php` — Update `transformApiPayloadToInternal()` to handle labels + campaign_id
- New: `contentstudio-backend/app/Http/Controllers/Api/V1/LabelController.php`
- New: `contentstudio-backend/app/Http/Controllers/Api/V1/CampaignController.php`
- New: `contentstudio-backend/app/Http/Resources/Api/V1/LabelResource.php`
- New: `contentstudio-backend/app/Http/Resources/Api/V1/CampaignResource.php`
- `contentstudio-backend/app/Http/Requests/Api/V1/PostStoreRequest.php` — Add validation for labels and campaign_id
- `contentstudio-backend/storage/api-docs/api-docs.json` — Update Swagger docs
