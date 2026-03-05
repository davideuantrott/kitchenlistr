# KitchenListr — Code Review

**Reviewed files:**
- `index.html` (~303KB, ~10,888 lines)
- `sw.js`
- `manifest.json`
- `offline.html`

> **Status key:**
> - ✅ **Fixed** — implemented and merged into `main`
> - 🔶 **Deferred** — identified risk or complexity; see rationale below each item

---

## Executive Summary

KitchenListr is a well-featured, thoughtfully designed meal planning PWA. For a solo-developer side project built in vanilla JS without a framework or build toolchain, the quality is genuinely impressive. The Firestore data model is clean, the household/personal context split is well-abstracted, and the UI feature set is extensive.

The dominant architectural reality is that everything lives in a single ~303KB HTML file — a deliberate and valid choice for this project type, but one that creates compounding costs in every quality dimension. The most critical individual issues are: several XSS vectors in the print output, a fragile ad-hoc item text extraction in keyboard autocomplete, and PWA configuration mismatches across files.

---

## Critical Issues (Security / Correctness)

### 1. ✅ XSS in print output — `item.text` and `item.context` not escaped

**Severity: Critical** | **Status: Fixed**

In `printShoppingList()`, `item.text` (ingredient name) and `item.context` (recipe name) were inserted into the print HTML string without `escapeHtml()`. The same risk existed for `aisle.name` in the aisle-grouped branch. The planner print functions (`generateWeekPrintHtml`, `generateMonthPrintHtml`) were already using `escapeHtml()` correctly.

**Fix applied:** `escapeHtml()` added to `item.text`, `item.context`, and `aisle.name` in all three render branches of `printShoppingList()`.

---

### 2. ✅ `confirmImport()` — imported share data stored without field validation

**Severity: Critical (data integrity)** | **Status: Fixed**

`confirmImport()` was reading a Firestore document from the `shared/` collection and writing its contents directly to the user's personal/household paths with no validation of field types, array lengths, string lengths, or unexpected fields.

**Fix applied:** New `isValidSharePayload(data)` validator checks:
- Field types and required presence (`name` as non-empty string, `ingredients`/`aisles`/`recipes` as arrays if present)
- Reasonable length caps: 200 ingredients per recipe, 500 recipes per bulk import, 200 aisles per shop
- Rejects unknown `type` values entirely

`confirmImport()` now calls the validator before any Firestore write.

---

### 3. ✅ Autocomplete text extraction via `textContent.split('(')[0]` — silently truncates ingredient names

**Severity: Critical (data correctness)** | **Status: Fixed**

In `handleAdhocKeydown()` and `handleAutocompleteKeydown()`, keyboard-selected autocomplete suggestions used `selectedItem.textContent.split('(')[0].trim()` to recover the ingredient name. This silently truncated names containing parentheses (e.g. `"Peppers (red)"` → `"Peppers "`).

**Fix applied:** Both autocomplete render functions (`showIngredientAutocomplete`, `handleAdhocInput`) now set `item.dataset.name = name` on each suggestion element. Both keydown handlers now read `selectedItem.dataset.name` directly.

---

### 4. ✅ `generateId()` / `generateShareId()` use `Math.random()`

**Severity: Medium-Low** | **Status: Fixed (for access-control tokens)**

`Math.random()` is not cryptographically secure. This is acceptable for recipe/shop IDs but not for tokens used for access control (share links, household invite codes).

**Fix applied:** New `generateSecureToken(bytes)` function using `crypto.getRandomValues()`. `generateShareId()` and `generateInviteLink()` now use this. `generateId()` retains `Math.random()` — acceptable for non-access-control IDs.

---

## High Priority Issues (Performance / Reliability)

### 5. 🔶 Redundant full re-renders on every Firestore snapshot

**Severity: High (large datasets)** | **Status: Deferred**

Every `onSnapshot` listener calls `renderShoppingList()` after updating its local state. With 6–7 simultaneous listeners, the initial load triggers `renderShoppingList()` 6+ times in rapid succession. There is no debouncing or batching.

**Deferred because:** Debouncing render calls changes the timing of all UI updates across multiple snapshot listeners. This requires careful testing to ensure fast-path interactions (e.g. typing in a form field while a snapshot fires) are not perceptibly delayed. The risk of subtle regression across all views outweighs the benefit for typical dataset sizes.

---

### 6. ✅ `getAllIngredientNames()` called on every keypress — no memoization

**Severity: Medium-High** | **Status: Fixed**

The function was iterating every ingredient of every recipe on every keystroke in both autocomplete inputs (up to 1,000 object traversals per key press for a large recipe collection).

**Fix applied:** Module-level `_ingredientNamesCache` variable. `getAllIngredientNames()` returns the cached result on subsequent calls. Cache is set to `null` in the recipes `onSnapshot` callback (the only path that changes the ingredient list), causing a lazy rebuild on the next keypress.

---

### 7. ✅ Sequential unbatched Firestore writes in bulk operations

**Severity: High** | **Status: Fixed (for `saveIngredientEdit` and `deleteIngredient`)**

`saveIngredientEdit()` and `deleteIngredient()` were looping through recipes calling `await setDoc()` individually — sequential round-trips, and non-atomic (tab close mid-loop left data partially updated).

**Fix applied:** Both functions now use `writeBatch()` — a single atomic network call. `writeBatch` added to the Firestore SDK import. Note: `shareDataToHousehold()` also has this pattern but involves writes to a *different user's* Firestore path; Firestore `writeBatch` cannot span security boundaries in this way, so that path remains sequential and is tracked as a separate concern.

---

### 8. 🔶 `shoppingHave` and `dayNotes` grow unboundedly in Firestore

**Severity: Medium-High (long-term users)** | **Status: Deferred**

Both accumulate past-date keys indefinitely in single Firestore documents with a 1MB hard limit.

**Deferred because:** Any cleanup pass silently deletes user data. The retention threshold (e.g. 90 days) is a product decision that should be made explicitly rather than encoded silently. Recommend deciding on the threshold and communicating it to users before implementing.

---

### 9. 🔶 Full DOM rebuild on every shopping item checkbox toggle

**Severity: Medium** | **Status: Deferred**

Checking a shopping item triggers a full `renderShoppingList()` rebuild via the Firestore snapshot cycle.

**Deferred because:** `renderShoppingList()` also manages the `recentlyCheckedKey` spring-check animation, the hide-checked filter, and stagger animation order. Patching the DOM directly after a toggle would need to replicate or bypass all of this logic, creating a second code path that could diverge from the canonical render. A simpler win would be restoring scroll position after each re-render — tracked as a future improvement.

---

### 10. 🔶 `ingredientAisles` single-document model has a 1MB Firestore ceiling

**Severity: Low-Medium** | **Status: Deferred**

All ingredient-to-aisle mappings for all shops are stored in a single settings document.

**Deferred because:** Migrating to a subcollection per shop is a schema migration requiring a data migration path for existing users. Low urgency for typical usage sizes; worth adding a console warning if the entry count exceeds a threshold (e.g. 2,000 entries).

---

## Medium Priority Issues (Maintainability / Code Quality)

### 11. 🔶 ~50+ `window.functionName` assignments pollute the global namespace

**Severity: Medium** | **Status: Deferred**

Every handler-callable function is assigned to `window` to work around `<script type="module">` scoping.

**Deferred because:** This is a ~50-function refactor touching every interactive element in the app. The correct fix is converting inline `onclick="..."` handlers to delegated event listeners, following the pattern already used for recipe category filter chips (Phase 12). Should be done as a dedicated pass, not piecemeal.

---

### 12. 🔶 Inconsistent event handling — mix of inline handlers and delegated listeners

**Severity: Medium** | **Status: Deferred**

Inline handlers with string arguments are also an XSS surface in dynamically generated HTML.

**Deferred because:** Inextricably linked to #11. Both should be addressed together in a dedicated refactor pass.

---

### 13. 🔶 Fixed `setTimeout` delays used as synchronization in auth flow

**Severity: Medium** | **Status: Deferred**

`checkForJoin()`, `checkForImport()`, and `checkShowWelcome()` are triggered via 300/500/800ms fixed delays after auth state change. On slow connections this can fail silently.

**Deferred because:** Replacing these with a data-ready Promise/callback requires architectural changes to `setupDataListeners()` and the `onAuthStateChanged` flow. The existing approach has worked reliably in practice. The risk of introducing a new race condition during a refactor exceeds the benefit, given that the fixed delays are conservatively long and the import/join modals handle their own error states.

---

### 14. 🔶 `confirm()` / `alert()` / `prompt()` for destructive actions

**Severity: Medium** | **Status: Deferred**

Native browser dialogs block the main thread, cannot be styled, and look unprofessional in a PWA context (show the page URL on mobile Chrome).

**Deferred because:** Each call site needs a custom confirmation modal. This is meaningful UI effort — new HTML + CSS per dialog, or a reusable confirmation modal component. The app already has a well-established modal pattern; this should be a dedicated pass using that pattern.

---

### 15. 🔶 Stale `shoppingHave` keys persist after recipe or ingredient rename

**Severity: Medium** | **Status: Deferred**

Shopping "have" state is keyed by ingredient name + recipe ID + date. Renames leave stale keys that cause previously-checked items to appear unchecked after a rename.

**Deferred because:** Migrating `shoppingHave` keys on rename requires a complete understanding of the composite key format across all code paths. Incorrectly migrating keys could corrupt have-state in the opposite direction. Should be tackled alongside the batch-write improvements (#7) as part of a broader rename-operation hardening pass.

---

### 16. 🔶 `deleteCurrentRecipe()` — uncoordinated local and snapshot cleanup

**Severity: Medium-Low** | **Status: Deferred**

After `deleteDoc()`, the function immediately loops through the local `mealPlan` array. If the `recipes` snapshot fires before this completes, an intermediate state with a dangling reference may be rendered.

**Deferred because:** The race window is very small (local array iteration is synchronous after the `await deleteDoc()`). A full Firestore transaction to atomically delete the recipe and update meal plan entries adds complexity and introduces a new failure mode (transaction contention). The current risk is cosmetic (a brief dangling reference in the planner) and self-heals on the next snapshot.

---

### 17. ✅ `document.execCommand('copy')` is deprecated

**Severity: Low-Medium** | **Status: Fixed**

`copyShareLink()` was using the deprecated `document.execCommand('copy')`.

**Fix applied:** `copyShareLink()` is now `async` and uses `navigator.clipboard.writeText()`. A `try/catch` fallback to `execCommand` is retained for browsers that deny clipboard permission.

---

### 18. 🔶 Many Firestore write paths have no `try/catch` — silent failures

**Severity: Medium** | **Status: Deferred**

Functions such as `addAisle()`, `deleteAisle()`, `moveAisle()`, `setAisleEmoji()`, and `saveUserPrefs()` fire `await setDoc/updateDoc` without a surrounding `try/catch`. Firestore rejections are silent — the UI does not update and the user receives no feedback.

**Deferred because:** ~15 functions need updating. A piecemeal approach risks inconsistent error UX across the app. The correct fix is a shared `async function firestoreWrite(fn, errorMessage)` wrapper — doing that properly is a dedicated pass. Note: `saveIngredientEdit()` and `deleteIngredient()` already have `try/catch` and were updated with `writeBatch` in #7.

---

## Low Priority / Nice-to-Have Improvements

### 19. ✅ Service worker precaches wrong font URL

**Status: Fixed**

`sw.js` was precaching `Source+Sans+3` (unused). Replaced with the actual combined Google Fonts URL for all four fonts in use (Fraunces, DM Sans, DM Mono, Caveat). `CACHE_NAME` bumped to `kitchenlistr-v3`.

---

### 20. ✅ `manifest.json` theme_color doesn't match `<meta name="theme-color">`

**Status: Fixed**

`manifest.json` `theme_color` updated from `#4FB286` (old green) to `#F2654A` (Warm Pantry tomato). `background_color` updated from `#F4F9F7` to `#F5F5F0`.

---

### 21. ✅ PWA manifest shortcuts are not handled in app JS

**Status: Fixed**

`checkForShortcuts()` added; called synchronously after auth in `onAuthStateChanged`. Handles `?action=add-recipe` → `openRecipeModal()` and `?view=shopping` → `showView('shopping')`. URL params are cleared via `history.replaceState` after handling.

---

### 22. ✅ `offline.html` uses old green color scheme + missing `lang` attribute

**Status: Fixed**

All hardcoded old-green values replaced with Warm Pantry tokens (`#F5F5F0` background, `#F2654A` heading/button, `#D94E35` button hover, `#1A1A1A` body text, `#3D3D3D` paragraph). Note: `lang="en"` was already present on `<html>`.

---

### 23. 🔶 PWA manifest icons use `data:` URI SVGs — limited browser support

**Severity: Low** | **Status: Deferred**

Most browsers expect PWA icons as separate files with real URLs. `data:` URI icons are commonly rejected for home screen splash and app icon use, resulting in a generic icon on install.

**Deferred because:** Requires creating and committing `icon-192.png` and `icon-512.png` files. This is an asset/design task, not a code change.

---

### 24. ✅ `installPromptDismissed` localStorage entry never expires

**Status: Fixed**

Changed from storing `'true'` to storing a millisecond timestamp. `showInstallPrompt()` now re-offers after 30 days. `dismissInstallPrompt()` stores `Date.now().toString()`.

---

### 25. ✅ `welcomeShown_{uid}` localStorage keys accumulate and are never removed

**Status: Fixed**

`checkShowWelcome()` now removes all `welcomeShown_*` keys whose UID suffix does not match the currently signed-in user, before checking the current user's key. Prevents unbounded key accumulation when multiple accounts use the same device.

---

### 26. ✅ Service worker `cache.put()` has no error handling

**Status: Fixed**

`.catch(err => console.warn('Cache write failed:', err))` added to the `cache.put()` call in the fetch handler.

---

### 27. ✅ Sync status element has no `aria-live` region

**Status: Fixed**

`role="status"` and `aria-live="polite"` added to `#sync-status`.

---

### 28. ✅ Autocomplete dropdowns have no ARIA listbox/option roles

**Status: Fixed**

Both autocomplete dropdown containers now have `role="listbox"`. Each suggestion item now has `role="option"` and `aria-selected="false"`. (Dynamic `aria-selected="true"` on keyboard highlight is a further improvement, tracked under the full event-handler refactor — #11/12.)

---

### 29. 🔶 Modal focus is not trapped

**Severity: Low-Medium** | **Status: Deferred**

Modals have `role="dialog"` and `aria-modal="true"` but Tab will cycle through elements behind the modal overlay, violating ARIA authoring practices.

**Deferred because:** There are 14 modals. A focus trap implementation needs to handle: finding all focusable children, Tab/Shift-Tab interception, focus restoration to the trigger element on close, and edge cases (modals that open other modals). This is a self-contained accessibility pass that deserves dedicated implementation and testing.

---

### 30. 🔶 Firestore write rejections produce no user feedback in many handlers

**Status: Deferred** — see #18 above.

---

## Summary Table

| # | Issue | Category | Severity | Status |
|---|---|---|---|---|
| 1 | XSS in print output (`item.text`/`item.context` unescaped) | Security | **Critical** | ✅ Fixed |
| 2 | `confirmImport()` — no field validation on imported share data | Security | **Critical** | ✅ Fixed |
| 3 | Autocomplete text extraction via `textContent.split('(')` truncates ingredient names | Correctness | **Critical** | ✅ Fixed |
| 4 | `Math.random()` used for share/invite tokens | Security | Medium-Low | ✅ Fixed |
| 5 | 6+ redundant full re-renders on initial Firestore load | Performance | High | 🔶 Deferred |
| 6 | `getAllIngredientNames()` uncached — iterates all recipes on every keypress | Performance | Medium-High | ✅ Fixed |
| 7 | Sequential unbatched Firestore writes in bulk operations | Performance/Reliability | High | ✅ Fixed |
| 8 | `shoppingHave` and `dayNotes` grow unboundedly in Firestore | Data Model | Medium-High | 🔶 Deferred |
| 9 | Full DOM rebuild on every checkbox toggle | Performance/UX | Medium | 🔶 Deferred |
| 10 | `ingredientAisles` single-document model has 1MB Firestore ceiling | Data Model | Low-Medium | 🔶 Deferred |
| 11 | ~50 `window.functionName` assignments pollute global namespace | Maintainability | Medium | 🔶 Deferred |
| 12 | Inconsistent inline vs. delegated event handler patterns | Maintainability | Medium | 🔶 Deferred |
| 13 | Fixed `setTimeout` delays used as synchronization in auth flow | Reliability | Medium | 🔶 Deferred |
| 14 | `confirm()` / `alert()` / `prompt()` for destructive actions | UX/Accessibility | Medium | 🔶 Deferred |
| 15 | Stale `shoppingHave` keys after recipe/ingredient rename | Correctness | Medium | 🔶 Deferred |
| 16 | `deleteCurrentRecipe()` — uncoordinated local and snapshot cleanup | Correctness | Medium-Low | 🔶 Deferred |
| 17 | `document.execCommand('copy')` deprecated | Compatibility | Low-Medium | ✅ Fixed |
| 18 | Many Firestore write paths have no `try/catch` — silent failures | Reliability | Medium | 🔶 Deferred |
| 19 | `sw.js` precaches wrong font URL (`Source+Sans+3`, not used) | PWA | Low | ✅ Fixed |
| 20 | `manifest.json` `theme_color` (`#4FB286`) doesn't match `<meta>` (`#F2654A`) | PWA | Low | ✅ Fixed |
| 21 | PWA shortcuts (`?action=add-recipe`, `?view=shopping`) not handled in JS | PWA | Low | ✅ Fixed |
| 22 | `offline.html` uses old green; missing `lang` attribute | PWA/A11y | Low | ✅ Fixed |
| 23 | `manifest.json` icons use `data:` URI SVGs — limited browser support | PWA | Low | 🔶 Deferred |
| 24 | `installPromptDismissed` localStorage entry never expires | UX | Low | ✅ Fixed |
| 25 | `welcomeShown_{uid}` localStorage keys accumulate, never cleaned | UX | Very Low | ✅ Fixed |
| 26 | `cache.put()` in service worker has no error handling | PWA | Low | ✅ Fixed |
| 27 | Sync status element has no `aria-live` region | Accessibility | Low | ✅ Fixed |
| 28 | Autocomplete dropdowns have no ARIA `listbox`/`option` roles | Accessibility | Low-Medium | ✅ Fixed |
| 29 | Modal focus not trapped | Accessibility | Low-Medium | 🔶 Deferred |
| 30 | Firestore write rejections — silent failures in many handlers | Reliability | Medium | 🔶 Deferred |

---

## Positive Observations

1. **`escapeHtml()` used consistently in main render paths** — all `innerHTML` assignments for user-controlled strings go through escaping. The print path gaps are exceptions to an otherwise good pattern.

2. **`getDataPath()` is a clean abstraction** — personal vs. household data paths are handled through a single function, used consistently throughout.

3. **Backward-compatible data model evolution** — `categories: string[]` alongside legacy `category: string`, optional `emoji` fields, all introduced without breaking existing Firestore data. Disciplined migration practice.

4. **Emoji picker is well-built from scratch** — ~400 emoji, category headers, search, event delegation via `data-e`/`data-k` attributes, viewport-aware positioning. Avoids third-party library complexity.

5. **`prefers-reduced-motion` is respected** — all keyframe animations gated behind `no-preference` and killed by `reduce`. Correct and considerate accessibility practice.

6. **Household role guards are correct** — `canManageHousehold()` / `isHouseholdOwner()` checks before destructive operations; owner-only deletion with "transfer ownership first" prompt prevents accidental lockout.

7. **Service worker cache versioning strategy is clean** — versioned `CACHE_NAME`, activate event deletes all old caches, fetch handler correctly bypasses Firebase and external API requests.

8. **Personal/household visual differentiation via CSS `body[data-mode]`** — overriding CSS custom properties at the body level is zero-JS-overhead theming. Elegant approach.

9. **`setupAdhocAutocomplete()` correctly removes before re-adding listeners** — prevents listener duplication when called multiple times, a common source of bugs.

10. **Design system tokens are well-organised and consistently applied** — full token set (colour, radius, shadow, font, glow) in `:root`, used consistently. Glow variables replacing hardcoded `rgba()` values make household theming overrides clean.
