# Stories: X (Twitter) @Mention Restriction Handling

---

## Story 1: [BE] Strip @mentions from X (Twitter) posts to prevent API rejection

### Description:

X (Twitter) API v2 has restricted the ability for self-serve tier apps (Free, Basic, Pro) to programmatically `@mention` users in posts. Posts containing `@username` text sent via the API are now rejected or the mention is silently dropped by X, which can cause posting failures.

Currently, `contentstudio-backend/app/Libraries/Helper.php::parsePostMentions()` (line 639–650) handles the Twitter case by converting the internal mention markup `{(@username)[@display_name]}` into `@username` — a literal @handle in the tweet text payload. Under the new X API rule, this causes the post to fail.

**The fix:** Change the Twitter case in `parsePostMentions()` to resolve the mention markup to the display name only (no `@` prefix), so the text publishes cleanly without triggering the API restriction. The post goes out, the tagged text becomes plain display text, and the post does not fail.

**File to modify:**
- `contentstudio-backend/app/Libraries/Helper.php` — `parsePostMentions()`, `case 'twitter':` block (line 639)

**Change:**
```php
case 'twitter':
    return preg_replace_callback(
        '/{\((.*?)\)\[(.*?)\]}/',
        function ($matches) {
            // X (Twitter) API no longer allows programmatic @mentions for self-serve tiers.
            // Always return the display name ($matches[2]) instead of the @handle ($matches[1])
            // so the post text is clean and the publish does not fail.
            return $matches[2];
        },
        $message
    );
```

This change also applies automatically to threaded tweets, since `parsePostMentions()` is called for each thread in `TwitterPlatform.php` (line 205).

---

### Workflow:

1. User creates a post in ContentStudio, selects an X (Twitter) account, and tags someone using `@` in the caption (e.g., `@ContentStudio`)
2. The post is scheduled or published immediately
3. The publishing job runs; `parsePostMentions()` processes the message for the Twitter platform
4. The `@handle` mention is resolved to the display name only (e.g., `ContentStudio` instead of `@ContentStudio`)
5. The cleaned message is sent to X API — the post publishes successfully
6. The published tweet shows the display name as plain text (not as a linked profile tag)

---

### Acceptance criteria:

- [ ] A post containing `@mention` markup (`{(@username)[@display]}`) scheduled to X publishes without error after this change
- [ ] The published tweet text shows the display name (e.g., `ContentStudio`) without an `@` prefix, not the @handle
- [ ] Posts with no `@mention` markup are unaffected
- [ ] Threaded tweets containing `@mention` markup are also resolved correctly and post without API rejection
- [ ] The fix applies to `textPost`, `imagePost`, and `videoPost` since all go through the same `parsePostMentions()` call
- [ ] Facebook, LinkedIn, Instagram, and other platform mention parsing is not affected (only the `twitter` case changes)

---

### Mock-ups:

N/A — backend-only change. No UI changes.

---

### Impact on existing data:

No schema changes. Existing scheduled posts containing Twitter @mention markup will now be resolved to plain display names at publish time rather than @handles. This is intentional and prevents post failures.

---

### Impact on other products:

- **Web app:** The composer-side mention UX (suggestions dropdown, mention highlighting) is unchanged — users can still type and select mentions. The change only affects how the text is formatted when sent to X's API.
- **Mobile apps:** Mobile apps that pass mention markup for Twitter posts will benefit from this fix automatically since the same backend function is invoked on publish.
- **Chrome extension:** Not impacted directly; uses the same backend publish pipeline.

---

### Dependencies:

None.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — N/A (backend-only story)
- [ ] Multilingual support — N/A (no user-facing strings changed)
- [ ] UI theming support — N/A (backend-only story)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

---

## Story 2: [FE] Show X (Twitter) @mention restriction warning in the composer footer

### Description:

X (Twitter) API v2 has removed the ability for self-serve apps to programmatically `@mention` users in posts. When a user composes a post in ContentStudio that includes one or more `@mentions` in the caption (or in threaded tweet content), and X (Twitter) is one of the selected accounts, they need to be informed that X will not link those mentions to profiles — but the post will still go out successfully.

This is a **warning** (not an error). The post publishes without issue (the backend strips the @handle; see **[BE] Strip @mentions from X (Twitter) posts to prevent API rejection**). The warning is purely informational so the user understands the behaviour before scheduling.

**Where to add:** `contentstudio-frontend/src/modules/composer_v2/views/SocialModal.vue::socialPostWarnings()` (line 2635). This computed property already pushes warning objects read by `MainComposerFooter.vue`. No changes to the footer component are needed.

**Detection logic:**
- X (Twitter) account is selected (`this.account_selection.twitter?.length > 0`)
- The active message (common box or Twitter-specific box) contains at least one `@` character followed by a letter (regex: `/@[A-Za-z]/`)
- Also check threaded tweets messages if `twitterOptions.has_threaded_tweets === true`

**Contextual warning copy (two variants):**

**Twitter only (no other platforms with mention support selected):**
> "X (Twitter) no longer allows user tagging via the API — @mentions in your post won't link to profiles, but your post will still be published as-is."

**Twitter + other platforms selected (Facebook, LinkedIn, or Bluesky):**
> "Your @mentions will work on [platform list], but X (Twitter) no longer allows user tagging via the API — mentions won't link to profiles on X."

Where `[platform list]` is a comma-separated list of the other selected platforms that support mentions (Facebook, LinkedIn, Bluesky).

**i18n keys to add (all 7 locales — en, fr, de, es, it, el, zh):**
- `composer.warnings.twitter_mention_restriction_only` — for the Twitter-only variant
- `composer.warnings.twitter_mention_restriction_mixed` — for the mixed-platforms variant (accepts a `{platforms}` interpolation variable)

---

### Workflow:

1. User opens the Composer
2. User selects an X (Twitter) account (and optionally other social accounts)
3. User types `@` followed by letters in the caption or in a threaded tweet
4. The footer warning area shows a yellow triangle alert with the warning message:
   - **Twitter only:** "X (Twitter) no longer allows user tagging via the API — @mentions in your post won't link to profiles, but your post will still be published as-is."
   - **Twitter + Facebook/LinkedIn/Bluesky:** "Your @mentions will work on Facebook, LinkedIn, but X (Twitter) no longer allows user tagging via the API — mentions won't link to profiles on X."
5. The user can still click "Schedule" or "Publish Now" — the post goes out without issue
6. If the user removes the `@` mention from the caption, the warning disappears
7. If the user deselects the X (Twitter) account, the warning disappears

---

### Acceptance criteria:

- [ ] Warning appears in the composer footer when X is selected AND caption/threaded tweet content contains `@` followed by at least one letter
- [ ] Warning does NOT appear when X is selected but no `@mentions` are present in any content
- [ ] Warning does NOT appear when `@mentions` are present but X is not selected
- [ ] When only X is selected: warning reads "X (Twitter) no longer allows user tagging via the API — @mentions in your post won't link to profiles, but your post will still be published as-is."
- [ ] When X + other mention-supporting platforms (Facebook, LinkedIn, Bluesky) are selected: warning reads "Your @mentions will work on [comma-separated list], but X (Twitter) no longer allows user tagging via the API — mentions won't link to profiles on X."
- [ ] Warning is informational only — it does not block publishing or scheduling
- [ ] Warning disappears when the `@mention` text is removed from the caption
- [ ] Warning disappears when X is deselected
- [ ] Warning appears for mentions in the main caption box AND in threaded tweet content
- [ ] Warning renders using the existing `TriangleAlert` icon and warning styling in `MainComposerFooter.vue` (no new footer UI needed)
- [ ] Warning copy uses i18n keys — no hardcoded English strings
- [ ] All 7 locales have corresponding translation keys

---

### Mock-ups:

The warning uses the existing footer warning UI (yellow triangle icon + text). No new design needed.

**Footer warning — Twitter only:**
```
⚠ X (Twitter) no longer allows user tagging via the API — @mentions in your post won't link to profiles, but your post will still be published as-is.
```

**Footer warning — Twitter + other platforms:**
```
⚠ Your @mentions will work on Facebook, LinkedIn, but X (Twitter) no longer allows user tagging via the API — mentions won't link to profiles on X.
```

---

### Impact on existing data:

No data changes. This is a UI-only warning added to the existing composer warning pipeline.

---

### Impact on other products:

- **Mobile apps:** Separate iOS and Android stories will handle the in-app warning for mobile composers.
- **Chrome extension:** Not impacted.

---

### Dependencies:

- **[BE] Strip @mentions from X (Twitter) posts to prevent API rejection** — the backend fix ensures posts don't fail; this story adds the user-facing warning.

---

### Global quality & compliance (wherever applicable)

- [ ] Mobile responsiveness — the composer is web-only; the footer warning renders correctly at all composer widths
- [ ] Multilingual support — all 7 locales must have the new i18n keys before release
- [ ] UI theming support — warning uses existing `TriangleAlert` icon with `rgb(var(--cstu-warning-500))` color (already theme-aware)
- [ ] White-label domains impact review
- [ ] Cross-product impact assessed (web, mobile apps, Chrome extension)

---

---

## UPDATE: [iOS] Add @mention suggestions to the post composer (Story 111917)

### Description APPEND (add to bottom of existing description):

---

**⚠ X (Twitter) Mention Restriction — Additional Scope**

X (Twitter) API v2 has removed the ability for self-serve tier apps to programmatically tag users via `@mentions` in posts. When the user has X (Twitter) selected as one of the publishing accounts and types `@` in the caption, the iOS composer must show an inline warning so the user understands that @mentions won't link to profiles on X, even though the post will still publish successfully.

**Behaviour:**
- Detect when X (Twitter) is among the selected accounts
- When the user types `@` followed by at least one letter in the caption, show an inline warning beneath the text area (or as a dismissible notice banner above the toolbar)
- If X is the only selected platform: show "X (Twitter) doesn't support user tagging via the API — @mentions won't link to profiles, but your post will still go out."
- If X is selected alongside other platforms (Facebook, LinkedIn, Bluesky): show "Your @mentions will work on [platform names], but X (Twitter) doesn't support user tagging via the API."
- The warning is informational — it does not block the post from being scheduled or published
- The warning disappears if the user removes the `@` text or deselects X
- **Note:** The mention suggestions overlay should NOT show X (Twitter) accounts in its results — only Facebook pages, LinkedIn company pages, and Bluesky profiles (as originally scoped). This avoids surfacing suggestions for a platform that can no longer be tagged.

### Additional Acceptance Criteria (append to existing AC):

- [ ] Mention suggestions overlay does NOT show X (Twitter) account suggestions (X is excluded from the suggestions API results by platform filter)
- [ ] When X (Twitter) is selected and the user types `@` followed by a letter, an inline warning appears in the composer
- [ ] Warning reads: "X (Twitter) doesn't support user tagging via the API — @mentions won't link to profiles, but your post will still go out." (Twitter-only selection)
- [ ] Warning reads: "Your @mentions will work on [platform list], but X (Twitter) doesn't support user tagging via the API." (when other mention-supporting platforms are also selected)
- [ ] Warning is non-blocking — the user can still publish/schedule without dismissing it
- [ ] Warning disappears when X is deselected or when no `@` is present in the caption

---

---

## UPDATE: [Android] Add @mention suggestions to the post composer (Story 111918)

### Description APPEND (add to bottom of existing description):

---

**⚠ X (Twitter) Mention Restriction — Additional Scope**

X (Twitter) API v2 has removed the ability for self-serve tier apps to programmatically tag users via `@mentions` in posts. When the user has X (Twitter) selected as one of the publishing accounts and types `@` in the caption, the Android composer must show an inline warning so the user understands that @mentions won't link to profiles on X, even though the post will still publish successfully.

**Behaviour:**
- Detect when `isTwitterSelected == true`
- When the user types `@` followed by at least one letter in the caption (`TextWatcher.afterTextChanged`), show an inline warning view below the text input (or a Snackbar-style notice)
- If X is the only selected platform: show "X (Twitter) doesn't support user tagging via the API — @mentions won't link to profiles, but your post will still go out."
- If X is selected alongside other platforms (Facebook, LinkedIn, Bluesky): show "Your @mentions will work on [platform names], but X (Twitter) doesn't support user tagging via the API."
- The warning is informational — it does not block the post from being scheduled or published
- The warning disappears if the user removes the `@` text or deselects X
- **Note:** The mention suggestions popup should NOT show X (Twitter) accounts in its results — only Facebook pages, LinkedIn company pages, and Bluesky profiles (as originally scoped). This avoids surfacing suggestions for a platform that can no longer be tagged.
- Add the warning strings to `contentstudio-android-v2/app/src/main/res/values/composer_strings.xml`

### Additional Acceptance Criteria (append to existing AC):

- [ ] Mention suggestions popup does NOT show X (Twitter) account suggestions (X is excluded from the suggestions API results by platform filter)
- [ ] When `isTwitterSelected == true` and the user types `@` followed by a letter, an inline warning view is shown in the composer
- [ ] Warning reads: "X (Twitter) doesn't support user tagging via the API — @mentions won't link to profiles, but your post will still go out." (Twitter-only selection)
- [ ] Warning reads: "Your @mentions will work on [platform list], but X (Twitter) doesn't support user tagging via the API." (when other mention-supporting platforms are also selected)
- [ ] Warning is non-blocking — user can still publish/schedule without dismissing it
- [ ] Warning disappears when X is deselected or when no `@` is present in the caption
