# Stories: Remember Last Login Method

---

## Story 1: [BE] Store last login method on successful authentication

### Description:

Currently, when a user logs in to ContentStudio, the backend updates `last_login` (timestamp) on the User model but does not record **which authentication method** was used. We need to add a `last_login_method` field to the User model and set it on every successful login — whether via email/password, Google, Facebook, Twitter/X, Apple, or magic link.

The field should also be included in the login API response so the frontend can persist it client-side for displaying on the sign-in page.

**Files to modify:**
- `contentstudio-backend/app/Models/User.php` — add `last_login_method` to fillable fields
- `contentstudio-backend/app/Http/Controllers/Authentication/AuthController.php` — set `last_login_method` in `authenticate()` (line ~192, where `last_login` is already updated), in `SSOAccountExist()` (line ~711), and in the magic link verification flow (`checkMagictoken()`, line ~879)

---

### Workflow:

1. User enters their email and password on the sign-in page and clicks "Sign In"
2. Backend authenticates the user successfully and sets `last_login_method` to `email` on the User document
3. The login API response now includes `last_login_method: "email"` alongside the existing `token`, `logged_user`, etc.
4. Alternatively, if the user signs in via Google SSO, backend sets `last_login_method` to `google`
5. Same for Facebook (`facebook`), Twitter/X (`twitter`), Apple (`apple`), and Magic Link (`magic_link`)

---

### Acceptance criteria:

- [ ] User model has a new `last_login_method` string field with allowed values: `email`, `google`, `facebook`, `twitter`, `apple`, `magic_link`
- [ ] `POST /login` (email/password) sets `last_login_method` to `email` on the authenticated user
- [ ] `POST /accountExist` (SSO login) sets `last_login_method` to the provider used (`google`, `facebook`, `twitter`, or `apple`) based on the `type` field in the request
- [ ] Magic link verification (`POST /checkMagictoken`) sets `last_login_method` to `magic_link`
- [ ] The login API response (for all methods) includes `last_login_method` in the JSON payload
- [ ] SSO callback response (the base64-encoded JSON returned to `/sso`) includes `last_login_method`
- [ ] Existing users without the field default gracefully (field returns `null`, no errors)
- [ ] The field is updated on every login, not just the first — it always reflects the most recent method

---

### Mock-ups:

N/A — backend only.

---

### Impact on existing data:

- **User collection (MongoDB):** New field `last_login_method` added. Existing user documents will not have this field; it will be `null` until their next login. No migration needed — MongoDB is schema-flexible.
- **API response:** The login response payload gains one new field. Frontend consumers that don't use it are unaffected.

---

### Impact on other products:

- **Mobile apps (iOS/Android):** Mobile apps call the same login API. They will receive `last_login_method` in the response but can ignore it. No breaking change.
- **Chrome extension:** Uses the same JWT auth flow. Unaffected.
- **White-label:** Same auth endpoints. The field is included regardless of white-label status.

---

### Dependencies:

None — this is a standalone backend change.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A, backend-only story
- [ ] Multilingual support — N/A, no user-facing strings
- [ ] UI theming support — N/A, ContentStudio has no dark mode
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---

### Metadata:
- **Group:** Backend
- **Skill Set:** Backend
- **Product Area:** Onboarding
- **Technical Area:** API
- **Priority:** Medium
- **Story Type:** feature
- **Estimate:** 2 points

---
---

## Story 2: [FE] Display "Last used" indicator on the sign-in page

### Description:

After a successful login, the frontend should store the login method used (received from the API response via **[BE] Store last login method on successful authentication**) in a persistent cookie + localStorage. On the sign-in page (`SignIn.vue`), if a returning user has this value stored, the corresponding login button should be visually highlighted with a "Last used" pill badge and a filled button style, while other SSO buttons remain in their default outline style.

This helps returning users instantly recognize which sign-in method they used last time, reducing friction and confusion — especially for users who don't remember whether they signed up with Google, Apple, or email.

**Files to modify:**
- `contentstudio-frontend/src/modules/account/views/SignIn.vue` — read cookie on mount, apply conditional styling + pill badge
- `contentstudio-frontend/src/store/jwt.js` — store `last_login_method` in cookie alongside JWT token
- `contentstudio-frontend/src/components/authentication/SSO.vue` — store login method on successful SSO callback (in `processSSO()`, line ~358 where `response.status` is true)

---

### Workflow:

1. User opens the ContentStudio sign-in page at `/login`
2. If this is a **returning user** (cookie `cs_last_login_method` exists):
   - User immediately sees that one of the sign-in buttons (e.g., "Google") is visually highlighted — it has a themed border, a subtle themed background tint, and a small "Last used" pill badge to the right of the button label
   - All other SSO buttons appear in their normal outline style (no change from current design)
   - If the last method was `email`, the email/password form section gets a subtle highlight instead — a small "Last used" text label appears next to the "or" divider text, reading: "or sign in with email · Last used"
   - If the last method was `magic_link`, the magic login link at the bottom gets the "Last used" pill next to it
3. If this is a **first-time visitor** (no cookie):
   - User sees the sign-in page exactly as it looks today — all buttons equal, no highlights
4. User signs in using any method (e.g., clicks "Google")
5. After successful authentication, the frontend stores `google` in the `cs_last_login_method` cookie (30-day expiry, path `/`, same domain) and localStorage
6. Next time the user visits `/login`, they will see the Google button highlighted

---

### UI Copy & Elements:

#### "Last used" pill badge (for SSO buttons):
- **Position:** Inside the SSO button, to the right of the provider label text
- **Text:** `Last used`
- **Style:** Small rounded pill — `text-xs font-medium px-2 py-0.5 rounded-full bg-primary-cs-50 text-primary-cs-500`
- **Visibility:** Only on the button matching the stored last login method. Hidden on small screens where SSO label text is already hidden (the `small-screen-range:hidden` breakpoint) — in that case, show only a subtle themed ring/border around the icon-only button

#### Highlighted SSO button (when it's the last-used method):
- **Border:** `border-primary-cs-200` (replaces `border-gray-200`)
- **Background:** `bg-primary-cs-50/50` (very subtle blue tint, replaces `bg-white`)
- **Text color:** `text-primary-cs-700` (replaces `text-gray-900`)
- All other SSO buttons remain exactly as they are today

#### For email/password (when last method was `email`):
- **Divider text change:** The "or" separator between SSO buttons and the email form changes to:
  - Text: `or sign in with email`
  - Subtext directly below (same line, smaller): `Last used` in `text-primary-cs-500 text-xs`
- The email form itself does not change styling — only the divider text indicates it was the last-used method

#### For magic link (when last method was `magic_link`):
- **Position:** Next to the existing "Sign in with a magic login link" text at the bottom
- **Element:** Append a "Last used" pill after the link text: `Sign in with a magic login link · Last used`
- **Style:** Same pill style as SSO buttons — `text-xs font-medium px-2 py-0.5 rounded-full bg-primary-cs-50 text-primary-cs-500`

#### Tooltip on the "Last used" pill (all locations):
- **Trigger:** Hover over the "Last used" pill
- **Text:** `You signed in with [method] last time. You can still use any option below to sign in.`
  - Examples:
    - `You signed in with Google last time. You can still use any option below to sign in.`
    - `You signed in with your email and password last time. You can still use any option below to sign in.`
    - `You signed in with a magic link last time. You can still use any option below to sign in.`
- **Purpose:** Reassures the user that they're not locked into one method — all options are still available

---

### Cookie specification:

| Property | Value |
|---|---|
| Name | `cs_last_login_method` |
| Values | `email`, `google`, `facebook`, `twitter`, `apple`, `magic_link` |
| Expiry | 30 days |
| Path | `/` |
| Domain | Current domain (works for both main app and white-label domains) |
| HttpOnly | `false` (must be readable by client-side JS) |
| Secure | `true` in production |
| SameSite | `Lax` |

Also mirror the value in `localStorage` under the key `cs_last_login_method` as a fallback (some browsers restrict cookies in certain contexts).

---

### When to write the cookie:

1. **Email/password login** (`SignIn.vue`, `loginAccount()` method, after line ~345 where JWT is committed): store `email`
2. **SSO login** (`SSO.vue`, `processSSO()` method, after line ~375 where JWT is committed): store the `response.type` value (`google`, `facebook`, `twitter`, `apple`) — this is already available in the decoded SSO response as `response.type`
3. **Magic link login:** In the magic link verification handler, after successful auth: store `magic_link`

---

### Acceptance criteria:

- [ ] After email/password login, `cs_last_login_method` cookie is set to `email` with 30-day expiry
- [ ] After Google SSO login, cookie is set to `google`
- [ ] After Facebook SSO login, cookie is set to `facebook`
- [ ] After Twitter/X SSO login, cookie is set to `twitter`
- [ ] After Apple SSO login, cookie is set to `apple`
- [ ] After magic link login, cookie is set to `magic_link`
- [ ] Value is also stored in `localStorage` under the same key as fallback
- [ ] On the sign-in page, if cookie exists, the corresponding SSO button shows a "Last used" pill badge and highlighted border/background
- [ ] If last method was `email`, the divider text shows "or sign in with email" with a "Last used" label
- [ ] If last method was `magic_link`, the magic login link shows a "Last used" pill
- [ ] If no cookie exists (first-time visitor, cleared cookies, expired), the sign-in page looks exactly the same as it does today
- [ ] The "Last used" pill has a tooltip explaining: "You signed in with [method] last time. You can still use any option below to sign in."
- [ ] All other login methods remain fully visible and clickable — nothing is hidden or disabled
- [ ] On small screens (where SSO labels are hidden and only icons show), the last-used button has a subtle themed ring border instead of the pill text
- [ ] Cookie works correctly on white-label domains (scoped to current domain)
- [ ] The highlighted state updates after each login — if user switches from Google to email, next visit shows email as "Last used"

---

### Mock-ups:

N/A — follow the UI spec above. Reference: Better Auth's "Last used" badge pattern.

---

### Impact on existing data:

- **No existing data changes.** The cookie and localStorage value are new — created on first login after this feature ships.
- **No database changes on frontend side.** This is purely a client-side persistence mechanism.

---

### Impact on other products:

- **Mobile apps:** Not affected. Mobile apps have their own native login screens. The cookie is web-only.
- **Chrome extension:** The Chrome extension uses the same domain cookies. If a user logs in via the web app, the extension's next login page visit will show the "Last used" indicator. This is expected and helpful.
- **White-label domains:** Each white-label domain gets its own cookie (cookies are domain-scoped). A user logging into `app.acme.com` won't see a "Last used" indicator when visiting `app.contentstudio.io`, which is correct behavior.

---

### Dependencies:

- Depends on: **[BE] Store last login method on successful authentication** — the API must return `last_login_method` in the response. However, the frontend cookie-writing can work independently (FE can determine the method from context: if login was via `loginAccount()` → `email`, if via SSO.vue → `response.type`). The backend field is needed for server-side accuracy but the FE implementation can ship in parallel.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness (frontend only, N/A for backend-only stories)
- [ ] Multilingual support (frontend + backend, translations available or fallback handled)
- [ ] UI theming support — N/A, ContentStudio has no dark mode
- [ ] White-label domains impact review
- [ ] Cross-product impact assessment (web, mobile apps, Chrome extension)

---

### Metadata:
- **Group:** Frontend
- **Skill Set:** Frontend
- **Product Area:** Onboarding
- **Technical Area:** Web App
- **Priority:** Medium
- **Story Type:** feature
- **Estimate:** 3 points
