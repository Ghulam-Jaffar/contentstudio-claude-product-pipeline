# Research: Remember Last Login Method

## Current State

### Supported Login Methods
ContentStudio supports **6 authentication methods:**
1. **Email/Password** — standard form login
2. **Google SSO** — via GoogleSignin component
3. **Facebook SSO** — OAuth redirect
4. **Twitter/X SSO** — OAuth redirect
5. **Apple SSO** — OAuth redirect (special: if `is_password_set=false` + `apple_id` exists, user MUST use Apple)
6. **Magic Link** — passwordless email login

### Backend (Key Files)
| File | What it does |
|---|---|
| `contentstudio-backend/routes/web/auth.php` | Auth routes: `POST /login`, `POST /accountExist` (SSO), `POST /magicLink` |
| `contentstudio-backend/app/Http/Controllers/Authentication/AuthController.php` | Main login logic: `authenticate()` (email/pass), `SSOAccountExist()` (SSO), `magicLink()` |
| `contentstudio-backend/app/Models/User.php` | User model — has `facebook_id`, `twitter_id`, `google_id`, `apple_id`, `is_password_set`, `last_login` |

### Frontend (Key Files)
| File | What it does |
|---|---|
| `contentstudio-frontend/src/modules/account/views/SignIn.vue` | Login page — 398 lines, shows all 5 SSO buttons + email/password form |
| `contentstudio-frontend/src/components/authentication/SSO.vue` | SSO callback handler after OAuth redirect |
| `contentstudio-frontend/src/modules/account/components/GoogleSignin.vue` | Google sign-in component |
| `contentstudio-frontend/src/store/jwt.js` | JWT token + logged_user stored in cookies (1-year expiry) + localStorage fallback |
| `contentstudio-frontend/src/store/sso.js` | SSO redirect URLs for each provider |
| `contentstudio-frontend/src/modules/common/mixins/authMixin.js` | Auth mixin — `loadRequiredData()`, `fetchProfile()`, `fetchWorkspaces()` |

### Current Login Page Structure (SignIn.vue)
```
SignIn.vue
├── LoginSideComponent (left sidebar — rotating feature highlights)
├── LogoComponent (branding)
├── LanguageSwitcher (top-right)
├── SSO buttons: Facebook, Twitter, Google (GoogleSignin component), Apple
├── Divider ("or")
├── Email/Password form
│   ├── Email input
│   ├── Password input (with show/hide toggle)
│   ├── "Remember me" checkbox
│   └── "Forgot password" link
└── Magic login link
```

### Gap: No Login Method Tracking
- **User model** has `last_login` (timestamp) but **NO field tracking which method was used** (e.g., `last_login_method`)
- SSO ID fields (`google_id`, `apple_id`, etc.) indicate the user CAN use that provider, but not that they DID
- **Frontend** stores JWT token + user ID in cookies, but **nothing about the auth method used**
- No localStorage or cookie tracking the last-used login method

---

## What Needs to Change

### Backend
1. **Add `last_login_method` field** to the User model — values: `email`, `google`, `facebook`, `twitter`, `apple`, `magic_link`
2. **Update `authenticate()`** in AuthController — set `last_login_method = 'email'` on successful login
3. **Update `SSOAccountExist()`** — set `last_login_method` to the SSO provider used
4. **Update `magicLink` flow** — set `last_login_method = 'magic_link'`
5. **Expose `last_login_method`** in the login API response so the frontend can store it client-side

### Frontend
1. **Store `last_login_method` in a cookie** (not httpOnly, 30-day expiry) + localStorage fallback after successful login
2. **Read the cookie on SignIn.vue mount** — if present, visually highlight the last-used button
3. **Add a "Last used" pill/badge** to the corresponding login button
4. **Style differentiation** — last-used button gets filled/primary variant; others get outline variant

---

## UX Reference: How Other Products Handle This

### 1. Google — Personalized Button (Best-in-class)
- Returning users see a **personalized "Continue as John Smith"** button instead of generic "Sign in with Google"
- The button morphs to show the user's name/email inside it
- Cookie-based persistence

### 2. Better Auth Framework — "Last Used" Badge (Best reference pattern)
- Ships a dedicated **Last Login Method plugin** as a first-class feature
- Displays a small **"Last used" badge** inside the button, to the right of the label
- Last-used button uses **filled variant**; other buttons use **outline variant** — creating instant visual hierarchy
- Cookie-based: `better-auth.last_used_login_method`, 30-day expiry, client-readable
- **This is the most directly applicable pattern for ContentStudio**

### 3. Notion — Identifier-First (sidesteps the problem)
- Uses magic link by default — user enters email, gets a login code
- No explicit "last used" label; routes automatically based on email

### 4. Slack — Workspace-Aware Routing
- Asks for workspace URL first, then shows only applicable auth methods
- No "last used" badge — method is determined by workspace admin settings

### 5. Atlassian — Email-First + Remember Me
- Identifier-first: enter email, server determines auth method
- "Remember me" checkbox saves email only, not the auth method

### Recommendation for ContentStudio
Follow the **Better Auth "Last Used" badge pattern:**
- Cookie-based (30 days, client-readable) + localStorage fallback
- Small "Last used" pill inside the relevant button
- Filled button for last-used method, outline for others
- Never hide other options — just visually prioritize the last-used one
- Graceful fallback: no cookie = all buttons equal

---

## Files That Will Be Touched

### Backend
- `contentstudio-backend/app/Models/User.php` — add `last_login_method` field
- `contentstudio-backend/app/Http/Controllers/Authentication/AuthController.php` — update `authenticate()`, `SSOAccountExist()`, magic link flow

### Frontend
- `contentstudio-frontend/src/modules/account/views/SignIn.vue` — read cookie, highlight last-used button, add badge
- `contentstudio-frontend/src/store/jwt.js` — store `last_login_method` in cookie alongside JWT
- `contentstudio-frontend/src/components/authentication/SSO.vue` — store login method on SSO callback
