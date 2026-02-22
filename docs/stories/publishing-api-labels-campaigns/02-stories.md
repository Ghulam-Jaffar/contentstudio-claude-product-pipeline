# Stories: Publishing API v1.7 — Labels & Campaigns Support

## Epic

**Title:** Publishing API v1.7 — Labels & Campaigns Support

**Description:**
Extend the Publishing API v1 to support labels and campaigns (folders) across the full lifecycle: listing available labels and campaigns, and passing them when creating posts. Once the core API endpoints are ready, update all integration apps (Zapier, Make.com, n8n, GPT app, Claude extension, MCP) and the Smart Scheduling via AI feature to leverage the new fields.

The fetch posts endpoint (GET) already returns labels and campaigns in the response (shipped in the previous release). This epic covers the remaining pieces: list endpoints + create post support + integration rollout.

---

## Story 1: [BE] Add list labels endpoint to Publishing API v1

### Description:

Add a new `GET /api/v1/workspaces/{workspace_id}/labels` endpoint that returns all labels for a workspace. This allows API consumers (Zapier, Make.com, n8n, etc.) to fetch the available labels so users can pick which labels to assign to posts.

The internal label management already exists in `contentstudio-backend/app/Http/Controllers/Planner/LabelController.php`. Create a new V1-specific controller that reuses the existing label repository/model to fetch labels for a workspace.

**Key files:**
- `contentstudio-backend/routes/api/v1.php` — add new route
- New: `contentstudio-backend/app/Http/Controllers/Api/V1/LabelController.php`
- New: `contentstudio-backend/app/Http/Resources/Api/V1/LabelResource.php`
- `contentstudio-backend/storage/api-docs/api-docs.json` — update Swagger docs

---

### Workflow:

1. API consumer sends `GET /api/v1/workspaces/{workspace_id}/labels` with their API key
2. The API validates the API key and workspace access
3. The API returns a JSON array of all labels for that workspace
4. Each label includes: `id`, `name`, `color`

---

### Acceptance criteria:

- [ ] `GET /api/v1/workspaces/{workspace_id}/labels` returns a list of labels for the workspace
- [ ] Each label object contains `id` (string), `name` (string), and `color` (string)
- [ ] The endpoint requires valid API key authentication (`X-API-Key` header)
- [ ] Returns 401 if API key is missing or invalid
- [ ] Returns 403 if the API key does not have access to the specified workspace
- [ ] Returns empty array `[]` when the workspace has no labels
- [ ] Swagger/OpenAPI documentation is updated with the new endpoint
- [ ] The endpoint respects the existing `api.key` and `throttle:api-v1` middleware

---

### Mock-ups:

N/A — backend API only.

---

### Impact on existing data:

None. Read-only endpoint that queries existing label data.

---

### Impact on other products:

- Enables Zapier, Make.com, n8n, GPT app, Claude extension, and MCP to fetch labels for dropdown population.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend API only
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend API only
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 2: [BE] Add list campaigns endpoint to Publishing API v1

### Description:

Add a new `GET /api/v1/workspaces/{workspace_id}/campaigns` endpoint that returns all campaigns (folders) for a workspace. This allows API consumers to fetch available campaigns so users can assign posts to a campaign.

The internal campaign/folder management already exists in `contentstudio-backend/app/Http/Controllers/Composer/FolderController.php`. Create a new V1-specific controller that reuses the existing folder repository/model.

**Key files:**
- `contentstudio-backend/routes/api/v1.php` — add new route
- New: `contentstudio-backend/app/Http/Controllers/Api/V1/CampaignController.php`
- New: `contentstudio-backend/app/Http/Resources/Api/V1/CampaignResource.php`
- `contentstudio-backend/storage/api-docs/api-docs.json` — update Swagger docs

---

### Workflow:

1. API consumer sends `GET /api/v1/workspaces/{workspace_id}/campaigns` with their API key
2. The API validates the API key and workspace access
3. The API returns a JSON array of all campaigns for that workspace
4. Each campaign includes: `id`, `name`, `color`

---

### Acceptance criteria:

- [ ] `GET /api/v1/workspaces/{workspace_id}/campaigns` returns a list of campaigns for the workspace
- [ ] Each campaign object contains `id` (string), `name` (string), and `color` (string)
- [ ] The endpoint requires valid API key authentication (`X-API-Key` header)
- [ ] Returns 401 if API key is missing or invalid
- [ ] Returns 403 if the API key does not have access to the specified workspace
- [ ] Returns empty array `[]` when the workspace has no campaigns
- [ ] Swagger/OpenAPI documentation is updated with the new endpoint
- [ ] The endpoint respects the existing `api.key` and `throttle:api-v1` middleware

---

### Mock-ups:

N/A — backend API only.

---

### Impact on existing data:

None. Read-only endpoint that queries existing campaign/folder data.

---

### Impact on other products:

- Enables Zapier, Make.com, n8n, GPT app, Claude extension, and MCP to fetch campaigns for dropdown population.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend API only
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend API only
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 3: [BE] Accept labels and campaign_id in create post API

### Description:

Update the `POST /api/v1/workspaces/{workspace_id}/posts` endpoint to accept optional `labels` (array of label IDs) and `campaign_id` (string) parameters in the request payload. Pass these through to the internal posting system so posts created via the API can be assigned labels and a campaign.

**Key changes:**
- `contentstudio-backend/app/Http/Requests/Api/V1/PostStoreRequest.php` — add validation rules for `labels` (optional array of strings) and `campaign_id` (optional string)
- `contentstudio-backend/app/Http/Controllers/Api/V1/PostController.php` — update `transformApiPayloadToInternal()` (line 606) to set `$socialPostConfig['labels']` from `$publicPayload['labels']` and `$socialPostConfig['folderId']` from `$publicPayload['campaign_id']`
- `contentstudio-backend/storage/api-docs/api-docs.json` — update Swagger docs for the create post endpoint

The internal `processSocialShare` already supports `labels` (array of label IDs) and `folderId` (string) in the plan data — this story just pipes the public API fields through.

---

### Workflow:

1. API consumer sends `POST /api/v1/workspaces/{workspace_id}/posts` with the existing payload plus optional `labels` and `campaign_id` fields
2. The API validates that each label ID exists in the workspace and the campaign_id exists in the workspace
3. The post is created with the specified labels and campaign assigned
4. The created post appears in ContentStudio with the correct labels and campaign visible in the Planner

---

### Acceptance criteria:

- [ ] The create post payload accepts an optional `labels` field — an array of label ID strings (e.g., `["66a1f...", "66a2f..."]`)
- [ ] The create post payload accepts an optional `campaign_id` field — a single campaign/folder ID string
- [ ] When `labels` is provided, the created post has those labels assigned (visible in Planner)
- [ ] When `campaign_id` is provided, the created post is assigned to that campaign (visible in Planner)
- [ ] When `labels` is omitted or empty, the post is created with no labels (existing behavior)
- [ ] When `campaign_id` is omitted or null, the post is created with no campaign (existing behavior)
- [ ] Invalid label IDs return a 400 validation error with a clear message
- [ ] Invalid campaign_id returns a 400 validation error with a clear message
- [ ] Swagger/OpenAPI documentation is updated with the new optional fields
- [ ] Existing create post calls without labels/campaign_id continue to work unchanged

---

### Mock-ups:

N/A — backend API only.

---

### Impact on existing data:

None. Labels and campaigns are stored in the same fields as the web composer uses (`labels` array and `folderId` on the plan document).

---

### Impact on other products:

- Enables all integration apps to assign labels and campaigns when creating posts via the API.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1** and **[BE] Add list campaigns endpoint to Publishing API v1** — consumers need to fetch valid IDs before passing them.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend API only
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend API only
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 4: [BE] Update Zapier app to support labels and campaigns in create post action

### Description:

Update the ContentStudio Zapier app's "Create Post" action to include optional labels and campaign fields. Use the new list labels and list campaigns API endpoints to populate dynamic dropdowns so Zapier users can select labels and a campaign when creating posts.

---

### Workflow:

1. User sets up a "Create Post" action in Zapier with ContentStudio
2. User sees a "Labels" multi-select dropdown populated from the labels API endpoint
3. User sees a "Campaign" single-select dropdown populated from the campaigns API endpoint
4. User selects labels and/or a campaign (both optional)
5. When the Zap runs, the selected label IDs and campaign ID are passed in the create post API call
6. The post is created in ContentStudio with the selected labels and campaign assigned

---

### Acceptance criteria:

- [ ] The "Create Post" action in Zapier includes an optional "Labels" multi-select field
- [ ] The "Labels" field uses a dynamic dropdown powered by `GET /api/v1/workspaces/{workspace_id}/labels`
- [ ] The "Create Post" action includes an optional "Campaign" single-select field
- [ ] The "Campaign" field uses a dynamic dropdown powered by `GET /api/v1/workspaces/{workspace_id}/campaigns`
- [ ] Selected labels and campaign are passed as `labels` and `campaign_id` in the API call
- [ ] Both fields are optional — posts can still be created without labels or campaign
- [ ] Existing Zaps without labels/campaign continue to work unchanged

---

### Mock-ups:

N/A

---

### Impact on existing data:

None.

---

### Impact on other products:

None — Zapier-specific.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1**, **[BE] Add list campaigns endpoint to Publishing API v1**, and **[BE] Accept labels and campaign_id in create post API**.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, external integration
- [ ] Multilingual support — N/A, external integration
- [ ] UI theming support — N/A, external integration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 5: [BE] Update Make.com app to support labels and campaigns in create post module

### Description:

Update the ContentStudio Make.com (formerly Integromat) app's "Create Post" module to include optional labels and campaign fields. Use the new list endpoints to populate dynamic dropdowns.

---

### Workflow:

1. User configures a "Create Post" module in Make.com with ContentStudio
2. User sees a "Labels" multi-select dropdown populated from the labels API
3. User sees a "Campaign" single-select dropdown populated from the campaigns API
4. Selected values are passed in the create post API call
5. The post is created with labels and campaign assigned

---

### Acceptance criteria:

- [ ] The "Create Post" module in Make.com includes optional "Labels" multi-select and "Campaign" single-select fields
- [ ] Both fields use dynamic dropdowns powered by the new V1 list endpoints
- [ ] Selected values are passed as `labels` and `campaign_id` in the API call
- [ ] Both fields are optional — existing scenarios without labels/campaign work unchanged

---

### Mock-ups:

N/A

---

### Impact on existing data:

None.

---

### Impact on other products:

None — Make.com-specific.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1**, **[BE] Add list campaigns endpoint to Publishing API v1**, and **[BE] Accept labels and campaign_id in create post API**.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, external integration
- [ ] Multilingual support — N/A, external integration
- [ ] UI theming support — N/A, external integration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 6: [BE] Update n8n node to support labels and campaigns in create post action

### Description:

Update the ContentStudio n8n community node's "Create Post" action to include optional labels and campaign fields with dynamic dropdowns powered by the new V1 list endpoints.

---

### Workflow:

1. User configures a "Create Post" action in n8n with ContentStudio
2. User sees "Labels" and "Campaign" fields with loadable options from the API
3. Selected values are passed in the create post API call

---

### Acceptance criteria:

- [ ] The "Create Post" action in n8n includes optional "Labels" (multi-select) and "Campaign" (single-select) fields
- [ ] Fields use `loadOptionsMethod` to fetch from the new V1 list endpoints
- [ ] Selected values are passed as `labels` and `campaign_id` in the API call
- [ ] Both fields are optional — existing workflows work unchanged

---

### Mock-ups:

N/A

---

### Impact on existing data:

None.

---

### Impact on other products:

None — n8n-specific.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1**, **[BE] Add list campaigns endpoint to Publishing API v1**, and **[BE] Accept labels and campaign_id in create post API**.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, external integration
- [ ] Multilingual support — N/A, external integration
- [ ] UI theming support — N/A, external integration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 7: [BE] Update GPT app action schema to support labels and campaigns

### Description:

Update the ContentStudio GPT (OpenAI) app's action schema for "Create Post" to include optional `labels` and `campaign_id` parameters so ChatGPT users can assign labels and a campaign when creating posts through the GPT action.

---

### Workflow:

1. User asks the GPT to create a post in ContentStudio
2. The GPT can ask which labels and campaign to assign (based on the schema)
3. The GPT passes selected label IDs and campaign ID in the API call

---

### Acceptance criteria:

- [ ] The GPT action schema for "Create Post" includes optional `labels` (array of strings) and `campaign_id` (string) parameters
- [ ] The GPT can list available labels and campaigns by calling the new list endpoints
- [ ] Selected values are passed through to the create post API call
- [ ] Existing GPT action usage without labels/campaign works unchanged

---

### Mock-ups:

N/A

---

### Impact on existing data:

None.

---

### Impact on other products:

None — GPT app-specific.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1**, **[BE] Add list campaigns endpoint to Publishing API v1**, and **[BE] Accept labels and campaign_id in create post API**.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, external integration
- [ ] Multilingual support — N/A, external integration
- [ ] UI theming support — N/A, external integration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 8: [BE] Update Claude extension to support labels and campaigns in post creation

### Description:

Update the ContentStudio Claude extension (MCP-based Claude integration) to support labels and campaigns when creating posts. The extension should be able to list available labels/campaigns and pass them in the create post tool call.

---

### Workflow:

1. User asks Claude to create a post in ContentStudio
2. Claude can list available labels and campaigns via the API
3. Claude passes selected label IDs and campaign ID when calling the create post tool

---

### Acceptance criteria:

- [ ] The Claude extension's create post tool accepts optional `labels` and `campaign_id` parameters
- [ ] The extension can list labels and campaigns via the new V1 endpoints
- [ ] Selected values are passed through to the create post API call
- [ ] Existing usage without labels/campaign works unchanged

---

### Mock-ups:

N/A

---

### Impact on existing data:

None.

---

### Impact on other products:

None — Claude extension-specific.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1**, **[BE] Add list campaigns endpoint to Publishing API v1**, and **[BE] Accept labels and campaign_id in create post API**.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, external integration
- [ ] Multilingual support — N/A, external integration
- [ ] UI theming support — N/A, external integration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 9: [BE] Update MCP server tools to support labels and campaigns

### Description:

Update the ContentStudio MCP (Model Context Protocol) server's post creation tool to support labels and campaigns. The MCP tool definitions should include the new parameters and pass them through to the API.

---

### Workflow:

1. An MCP client (e.g., Claude Desktop, Cursor) calls the ContentStudio create post tool
2. The tool accepts optional `labels` and `campaign_id` parameters
3. The MCP server passes these to the Publishing API create post endpoint

---

### Acceptance criteria:

- [ ] The MCP server's create post tool schema includes optional `labels` (array of strings) and `campaign_id` (string) parameters
- [ ] The MCP server exposes tools to list labels and campaigns from the V1 endpoints
- [ ] Selected values are passed through to the create post API call
- [ ] Existing MCP tool usage without labels/campaign works unchanged

---

### Mock-ups:

N/A

---

### Impact on existing data:

None.

---

### Impact on other products:

None — MCP-specific.

---

### Dependencies:

Depends on **[BE] Add list labels endpoint to Publishing API v1**, **[BE] Add list campaigns endpoint to Publishing API v1**, and **[BE] Accept labels and campaign_id in create post API**.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, external integration
- [ ] Multilingual support — N/A, external integration
- [ ] UI theming support — N/A, external integration
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---
---

## Story 10: [BE] Add labels and campaign support to Smart Scheduling via AI

### Description:

Update the Smart Scheduling via AI feature to support assigning labels and a campaign when AI-generated posts are created. The AI scheduling flow should allow users to pre-select labels and a campaign that will be applied to all posts generated in a scheduling batch.

---

### Workflow:

1. User configures a Smart Scheduling via AI session
2. User sees options to select labels and a campaign for the generated posts
3. When AI generates and schedules posts, the selected labels and campaign are applied to each post
4. The scheduled posts appear in the Planner with the correct labels and campaign

---

### Acceptance criteria:

- [ ] The Smart Scheduling via AI flow accepts optional labels and campaign selection
- [ ] Selected labels are applied to all posts generated in the scheduling batch
- [ ] Selected campaign is applied to all posts generated in the scheduling batch
- [ ] Both fields are optional — AI scheduling works without labels/campaign (existing behavior)
- [ ] Labels and campaign are visible on the generated posts in the Planner

---

### Mock-ups:

N/A

---

### Impact on existing data:

None. Uses existing label and campaign/folder fields on the plan document.

---

### Impact on other products:

None — Smart Scheduling via AI specific.

---

### Dependencies:

None — this uses the internal posting system directly, not the V1 API. However, the labels and campaigns list endpoints from **[BE] Add list labels endpoint to Publishing API v1** and **[BE] Add list campaigns endpoint to Publishing API v1** may be reused if the AI scheduling calls the API internally.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend/AI feature
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, backend/AI feature
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
