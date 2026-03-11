# Stories: Content Library Upload Modal Redesign

---

## Story 1: [FE] Redesign Content Library upload modal — sidebar, uploads view, and external source tabs

### Description:
As a ContentStudio user, I want the Media Library upload modal redesigned into a "Content Library" modal with an improved collapsible sidebar, an uploads view that matches the new Content Library page layout, and updated external source tabs so that the experience is consistent and modern across the entire app.

**What we're doing:**
- Rename the modal from "Media Library" / "Add Media" → **Content Library** everywhere it appears (modal title, CTA buttons, tooltips, i18n keys)
- **Sidebar redesign** (`SideTabs.vue`): Replace the current `Tabs` vertical sidebar with proper list items from the `@contentstudio/ui` design system. The sidebar should be **collapsible** — user can collapse it to icon-only mode to maximize content area. Sidebar items: Uploads, Direct Link, Pinterest, Facebook, Flickr, Pixabay, Giphy, Dropbox, Google Drive
- **Uploads view** (`MediaLibraryTab.vue`): Update the layout to match the new Content Library page (sc-113550) — same grid layout, same filters bar, same team member filter dropdown. This ensures a consistent experience whether the user is browsing media from the full Content Library page or from within the modal
- **External source tabs** (Pinterest, Facebook, Flickr, Pixabay, Giphy, Dropbox, Google Drive, Direct Link): Update the UI of tabs that currently have a basic input field + fetch CTA — redesign per the new prototype with improved layout, better spacing, and consistent styling with the rest of the modal

**Key files to modify:**
- `src/modules/publish/components/media-library/components/SideTabs.vue` — sidebar redesign + collapsibility
- `src/modules/publish/components/media-library/components/UploadMediaModal.vue` — modal shell, title update
- `src/modules/publish/components/media-library/components/MediaTabs/MediaLibraryTab.vue` — uploads view layout
- `src/modules/publish/components/media-library/components/FetchMediaUrlTab.vue` — Pinterest, Facebook, Flickr tab UI
- `src/modules/publish/components/media-library/components/SearchMediaTab.vue` — Pixabay, Giphy tab UI
- `src/modules/publish/components/media-library/components/DropBoxMediaTab.vue` — Dropbox tab UI
- `src/modules/publish/components/media-library/components/GoogleDriveAuth.vue` — Google Drive tab UI
- `src/modules/publish/components/media-library/components/DirectUploadFileTab.vue` — Direct Link tab UI
- `src/locales/*/publisher.json` (all 7 locales) — i18n key updates
- `src/assets/sass/app/components/modals/_modal_layout.scss` — `.media-library-modal-header` class rename
- All 22+ trigger-point files where CTA labels reference "Media Library"

---

### Workflow:
1. User is in the Composer (or any other surface — AI Library, Automation, Discovery, Planner, etc.) and clicks the "Content Library" CTA button (formerly "Media Library")
2. The Content Library modal opens with the **Uploads** view as the default tab
3. User sees a collapsible sidebar on the left with list items: Uploads, Direct Link, Pinterest, Facebook, Flickr, Pixabay, Giphy, Dropbox, Google Drive
4. The **Uploads** view shows media in the same grid layout as the Content Library page — with the same filters bar and team member filter dropdown
5. User can collapse the sidebar by clicking the collapse toggle — sidebar shrinks to icon-only mode, giving more space to the content area
6. User can expand the sidebar again by clicking the expand toggle
7. User clicks "Pinterest" in the sidebar → sees the updated Pinterest tab with improved input + fetch UI (better layout, consistent styling)
8. User clicks "Pixabay" in the sidebar → sees the updated Pixabay search tab with improved search input and results grid
9. User selects media from any tab and inserts it into their content as before — all existing selection/insertion behavior is preserved
10. User closes the modal — returns to their previous context

---

### Acceptance criteria:
- [ ] Modal title / header reads "Content Library" instead of "Add Media" or "Media Library"
- [ ] All CTA buttons across the app that open this modal read "Content Library" (not "Media Library")
- [ ] Sidebar uses proper list items from `@contentstudio/ui` design system (not basic `Tabs` component)
- [ ] Sidebar is collapsible — user can toggle between full sidebar and icon-only mode
- [ ] Sidebar collapsed/expanded state persists within the session (does not reset on tab switch)
- [ ] Sidebar items: Uploads, Direct Link, Pinterest, Facebook, Flickr, Pixabay, Giphy, Dropbox, Google Drive — with appropriate icons
- [ ] Uploads view matches the Content Library page layout: same grid view, same filters bar, same team member filter dropdown
- [ ] Team member filter dropdown in the Uploads view works the same as on the Content Library page
- [ ] External source tabs (Pinterest, Facebook, Flickr) have updated UI per design — improved input field + fetch CTA layout
- [ ] Search source tabs (Pixabay, Giphy) have updated UI per design — improved search input and results display
- [ ] Dropbox and Google Drive tabs have updated UI per design — consistent with other external source tabs
- [ ] Direct Link tab has updated UI per design — improved URL input and import flow
- [ ] All existing media selection and insertion behavior is preserved — no regressions in Composer, AI Library, Automation, etc.
- [ ] Modal opens correctly from all 22+ trigger points across the app (Composer, AI Library, Automation, Discovery, etc.)
- [ ] i18n keys updated across all 7 locales (en, fr, de, es, it, el, zh) — no hardcoded "Media Library" strings remain
- [ ] All theme-aware colors use CSS variable-backed classes (`text-primary-cs-500`, `bg-primary-cs-50`, etc.) — no hardcoded hex colors

### UI copy:

**Modal title:** "Content Library"

**Sidebar items (with icons):**
| Item | Icon | Tooltip (collapsed mode) |
|---|---|---|
| Uploads | CloudUpload | "Browse your uploaded files" |
| Direct Link | Link | "Import media from a URL" |
| Pinterest | Pinterest social icon | "Import images from Pinterest" |
| Facebook | Facebook social icon | "Import photos from Facebook" |
| Flickr | Flickr social icon | "Import photos from Flickr" |
| Pixabay | Image icon | "Search free stock photos on Pixabay" |
| Giphy | StickyNote icon | "Search GIFs on Giphy" |
| Dropbox | Dropbox social icon | "Import files from Dropbox" |
| Google Drive | GoogleDrive social icon | "Import files from Google Drive" |

**Sidebar collapse toggle tooltip:**
- When expanded: "Collapse sidebar"
- When collapsed: "Expand sidebar"

**Empty state (Uploads view, when no media exists):**
- Headline: "No uploads yet"
- Subtext: "Upload images, videos, and files to use across your posts and campaigns."
- CTA: "Upload Files"

**Error state (external source fetch fails):**
- Message: "We couldn't load media from [source name]. Please check the URL and try again."

**Loading state:**
- Skeleton grid placeholders matching the grid layout while content loads

---

### Mock-ups:

Figma: https://www.figma.com/proto/zJEk0csU8yeDNKhUR7DIEo/ContentStudio-WebApp?page-id=28789%3A35429&node-id=51759-157036&viewport=-10577%2C2987%2C0.49&t=PvYj9xdWWWu22oKq-1&scaling=min-zoom&content-scaling=fixed&starting-point-node-id=51759%3A156281

---

### Impact on existing data:
- No data changes — this is a purely frontend/UI redesign
- All existing media assets, folders, and external source integrations remain unchanged
- Event bus event name `show-media-library-modal` can remain as-is internally (renaming is optional, cosmetic only)

---

### Impact on other products:
- **Chrome extension** — if the Chrome extension surfaces a "Media Library" label, it needs updating to "Content Library"
- **White-label** — all renamed labels go through i18n, so white-label instances get the update automatically. Theme colors must use CSS variable-backed classes.
- **Mobile apps** — not impacted (this modal is web-only)

---

### Dependencies:
- Depends on: **[FE] Unified Content Library — rename and restructure Media Library into Content Library** (sc-113550) — the Uploads view inside the modal should share the same grid layout, filters, and team member filter dropdown as the Content Library page

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only — modal should remain responsive at smaller viewport widths)
- [ ] Multilingual support (frontend — all 7 locales updated with new copy)
- [ ] UI theming support (default + white-label, design library components are being used)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)
