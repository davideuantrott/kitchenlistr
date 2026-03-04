# KitchenListr – Project Guide for Claude Code

## Project Overview
KitchenListr is a meal planning and shopping list PWA built with vanilla HTML, CSS, and JavaScript. It is hosted on GitHub Pages and uses Firebase as its backend/database.

## Tech Stack
- **Frontend:** Vanilla HTML, CSS, JavaScript (no framework)
- **Backend/Database:** Firebase (Firestore)
- **Hosting:** GitHub Pages
- **PWA:** manifest.json + service worker (sw.js) for offline support and installability
- **Fonts:** Google Fonts (Fraunces + DM Sans + DM Mono + Caveat)

## Key Files
- `index.html` – main application
- `manifest.json` – PWA manifest (app name, icons, theme colour)
- `sw.js` – service worker (do not edit manually; update cache version when deploying changes)
- `offline.html` – shown when user is offline and page isn't cached

## Firebase Notes
- Firebase is connected directly from the browser (no server-side code)
- If the GitHub Pages URL changes, update the authorised domains in the Firebase console
- Firestore is the database in use (not Realtime Database)

## Hosting Notes
- GitHub Pages serves static files only — no server-side logic
- All paths must be relative (no absolute server paths)
- HTTPS is handled automatically by GitHub Pages

## Core Features
- Meal planning (add, edit, remove meals)
- Ingredient management with autocomplete/dictionary
- Shopping list generation from meal plans
- Shops and aisles management
- Meal categories
- PWA installable on mobile and desktop
- Offline support via service worker

## Service Worker
- Cache name is versioned (e.g. `kitchenlistr-v1`) — increment version when deploying significant changes to force cache refresh
- Precaches core assets on install
- Cleans up old caches on activate

## Development Conventions
- Keep everything in vanilla JS — no frameworks or build tools
- All Firebase interactions happen client-side
- Shopping list and meal data stored in Firestore
- Prefer progressive enhancement — app should degrade gracefully offline

## Common Tasks
- **Deploy:** commit and push to `main` branch; GitHub Pages updates automatically
- **Force cache refresh:** increment the `CACHE_NAME` version in `sw.js`
- **Add Firebase domain:** Firebase Console → Project Settings → Authorised Domains

## What to Avoid
- Do not add a Node/npm build process unless explicitly agreed
- Do not edit `sw.js` cache logic without also updating the cache version string
- Do not use localStorage for persistent data — Firestore is the source of truth

---

## Redesign: "Warm Pantry" Design System

> **Branch:** `redesign` — do not merge to `main` until all phases are reviewed and approved.
> **Design spec:** `kitchenlistr-design-spec.jsonc` in the repo root.
> **Mockup reference:** `kitchenlistr-mockup.html` in the repo root (open in browser).

### Design Decisions Agreed

| Topic | Decision |
|---|---|
| Aisle icons | Emoji picker + initial-letter fallback; image upload deferred to later |
| Recipe images | Hash-based pastel avatar + emoji, optional URL field; Firebase Storage deferred |
| Categories | Shared between Recipes and Meal Planner — single source of truth in Settings |
| Add shopping item | FAB button |
| Tablet/desktop nav breakpoint | 768px |

### Design Token Summary

All tokens live in `:root {}` in `index.html`. **Existing variable names are kept** so no JS breaks — only values are updated.

| Role | Variable | Value |
|---|---|---|
| Primary accent | `--accent-green` | `#F2654A` (tomato) |
| Accent dark | `--accent-green-dark` | `#D94E35` |
| Light accent bg | `--accent-green-light` | `#FAD4C0` (blush) |
| Success/herb | `--accent-mint` | `#5BBF7A` |
| Page background | `--bg-cream` | `#F5F5F0` (warm off-white) |
| Card surface | `--bg-white` | `#FFFFFF` |
| Primary text | `--text-primary` | `#1A1A1A` (ink) |
| Secondary text | `--text-secondary` | `#3D3D3D` (charcoal) |
| Border | `--border-light` | `#E8E4DC` |
| Radius sm | `--radius-sm` | `8px` |
| Radius md | `--radius-md` | `16px` |
| Radius lg | `--radius-lg` | `24px` |
| Pill radius | `--radius-pill` | `999px` |
| Font heading | `--font-heading` | Fraunces (serif) |
| Font body | `--font-body` | DM Sans |
| Font mono | `--font-mono` | DM Mono (quantities) |
| Font accent | `--font-accent` | Caveat (handwritten annotations) |

New brand colour variables also added (not yet widely used): `--color-tomato`, `--color-saffron`, `--color-herb`, `--color-blush`, `--color-buttercup`, `--color-sage-pale`, `--color-lavender`, `--color-peach`, `--color-sky`.

### Phases: Status

#### ✅ Phase 1 — Design tokens & typography
- CSS custom properties replaced with Warm Pantry palette
- Font import streamlined: Fraunces + DM Sans + DM Mono + Caveat
- `meta theme-color` updated to tomato (`#F2654A`)

#### ✅ Phase 2 — Component reskins
- **Buttons:** Pill shape (`--radius-pill`), hover shadow uses tomato tint
- **Cards:** Border removed; shadow-only depth; `--radius-lg` (24px)
- **Recipe cards:** Border removed, radius bumped to `--radius-lg`, shadow on hover
- **Inputs:** Mist (`--bg-cream`) background, transparent border until focus, tomato focus ring
- **Checkboxes:** 24×24px, tomato `accent-color`
- **Tags/badges/chips:** Pill radius throughout (suggestion tabs, vote buttons, shop default badge, recipe badge)

#### ✅ Phase 3 — Navigation
- **Mobile (<768px):** Nav is now a `position: fixed` bottom bar (72px), icon + label stacked, tomato active indicator line at top of tab
- **Desktop (≥768px):** Nav stays in header; active tab uses pill highlight, mint underline removed
- **Icons:** Inline SVG (Lucide-style, stroke 1.75) on all 5 buttons — hidden on desktop, visible on mobile
- **Header:** Frosted glass (`backdrop-filter: blur(12px)`, 92% opacity white)
- **Sign-out button:** Pill shape, ghost style
- Main content gets `padding-bottom: calc(72px + 1rem + env(safe-area-inset-bottom, 0px))` on mobile to clear the fixed nav and iPhone home indicator
- Nav height includes `env(safe-area-inset-bottom)` for iPhones with Dynamic Island / home bar
- Viewport meta has `viewport-fit=cover` (required for safe-area insets to work)
- `@media (max-height: 500px) and (orientation: landscape)` compacts header/nav in landscape on phones

---

#### ✅ Phase 4 — Shopping list upgrades
- Item cards: meal name (DM Sans small) + date (DM Mono) as separate spans in second line
- Sort chips: pill-shaped chip row replaces `<select>` (A–Z / Aisle / Date / Recipe); hidden `<select>` kept for print compat
- Date range navigator: `‹ [7 Apr – 13 Apr] ›` replaces raw date inputs; arrows shift by current span; preset chips (Today / 3d / 7d / This week / All) update the label
- Progress bar: herb-green fill inside summary card, "X of Y checked" label, updates live
- "Clear checked" floating pill: `position: fixed`, appears only when ≥1 item checked, shows count; hides on non-shopping views
- Tomato FAB: fixed bottom-right (shopping view only), scrolls to and focuses the add-item input

#### ✅ Phase 5 — Planner reskin
- Week/month toggle: pill pair (`--radius-pill`), tomato fill on active button
- Week nav buttons: pill shape, tomato hover
- Week strip: horizontal scrollable row of day pills (Mon/1 format) between nav and grid; today = tomato; dot indicator when meals exist; clicking scrolls to that day card
- Day cards: border removed, `--radius-lg`, shadow-only; today = tomato gradient header + tomato ring
- Empty meal slots: `.meal-empty` class → dashed border, transparent background
- Month grid: individual rounded cells with border (no gap-as-divider); today's date number = tomato circle badge
- Month meal pills: compact dot + recipe name, pill radius; dot colour matches meal type (breakfast/lunch/dinner)
- Planner FAB: tomato FAB (planner view only) opens recipe selector for today's first empty meal slot

#### ✅ Phase 6 — Emoji icon system (aisles)
- Emoji picker: `position: fixed` popover, 8-column grid of ~80 food-relevant emoji; no third-party library; closes on outside click; grid built once and cached via `data-built`
- Aisle settings: each aisle row has an emoji tap target (`+🍽️` placeholder → selected emoji); tapping opens picker and saves to Firestore via `setAisleEmoji(shopId, index, emoji)`
- Add-aisle row: emoji picker button prefixes name input; selected emoji persisted with new aisle in `addAisle()`
- Aisle data model: `{ name, emoji?, order }` — `emoji` field added, backward-compatible (undefined = no emoji)
- Shopping list headers: emoji shown if set; fallback = deterministic colour circle with initial letter (`aisleInitialColor()` — 10-colour brand palette, hash by char code)

#### ✅ Phase 7 — Recipe screen
- **Avatar system:** `recipeAvatarColor(name)` (10-colour brand palette, hash) + `recipeAvatarEmoji(name)` (food emoji set, second hash); deterministic per recipe name
- **Recipe cards restructured:** `padding: 0`, `overflow: hidden`; top 100px avatar strip (`.recipe-card-avatar`) + `.recipe-card-body` with padding; image shown instead of emoji when `imageUrl` set
- **Optional image URL field:** added to recipe form; saved to Firestore as `imageUrl`; backward-compatible (empty string = use avatar)
- **Category field:** originally a single `<select>`; now replaced with multi-select togglable chips (`.recipe-cat-chip`); recipes support multiple categories saved as `categories: string[]` in Firestore; `category: string` retained as first element for backward compat
- **Category filter chips:** pill chip row (`#recipe-category-filter`) above the grid; "All" + one chip per mealCategory; `currentRecipeCategory` state; filtering checks `recipe.categories` array (falls back to legacy `recipe.category` string); chips re-render on `showView('recipes')` and when `mealCategories` updates in settings listener
- **Custom recipe emoji:** emoji picker button (🍽️) in edit form; stored as `recipe.emoji`; overrides deterministic `recipeAvatarEmoji(name)`; "Auto (from name)" button clears it; `renderRecipeCategorySelector(selectedCats)` / `toggleRecipeCatChip()` / `clearRecipeEmoji()` are the new form helpers
- **Two-column grid:** `minmax` changed 280px → 200px (two columns from ~420px viewport)
- **View modal:** shows large 140px avatar at top; all categories shown as subtitle below title

#### ✅ Phase 8 — Blob decorations & animations
- **Blob decorations:** `.page-header::before` / `::after` pseudo-elements; organic blob shapes via `border-radius: 60% 40% 30% 70% / ...`; opacity 0.38; per-view pastel colours (blush/buttercup on recipes, sage-pale/lavender on planner, blush/peach on shopping, sage-pale/sky on settings, lavender/buttercup on account)
- **Blob animations:** `blobMorph` (border-radius oscillation, 9s/11s) + `blobFloat` (translate + rotate, 7s/9s); `::before` and `::after` run at different speeds and in reverse for organic feel
- **Card pop-in:** `popIn` keyframe (scale 0.92 + fade → scale 1); applied to `.recipe-card` (0.28s, 50ms stagger) and `.shopping-item` (0.22s, 35ms stagger); stagger driven by `--anim-order` CSS custom property set inline during render; `_shopAnimOrder` counter reset at start of each `renderShoppingList()` call
- **Spring check:** `springCheck` keyframe applied as `.spring-check` class on the newly checked shopping item; `recentlyCheckedKey` module variable set in `toggleShoppingItem()` before re-render, cleared after; class applied in `renderShoppingItem()` when `ing.key === recentlyCheckedKey`
- **Reduced motion:** all new animations inside `@media (prefers-reduced-motion: no-preference)`; existing `reduce` block kills them as second safety net

#### ✅ Phase 9 — Bug fixes & mobile polish

**Mobile nav & layout**
- `viewport-fit=cover` + `env(safe-area-inset-bottom)` ensures nav clears iPhone home indicator
- `-webkit-text-size-adjust: 100%` prevents iOS from zooming text on orientation change
- Landscape compact mode: `@media (max-height: 500px) and (orientation: landscape)` shrinks header/nav

**Shopping list**
- "Clear checked" pill no longer appears on non-shopping views; `renderShoppingList()` now checks `shopping-view` is `.active` before showing it

**Emoji picker (shared, used by aisles + recipe form)**
- Positioned to flip above trigger if near viewport bottom (`rect.bottom + pickerH > window.innerHeight`)
- `max-height: 260px; overflow-y: auto` prevents picker overflowing off-screen
- Outside-close uses `pointerdown` + `{ capture: true }` instead of `click`; stores `_emojiTrigger` reference to correctly exclude the trigger element on mobile

**Planner**
- `::before` blob on planner page-header resized (110→90px) and pushed right (`-15px → -45px`) to avoid clashing with Print Plan button
- Future (non-today) day cards get a slightly stronger border `#C8C2B8`
- Day notes section has horizontal padding so 📝 Add note text doesn't clip the card corner
- Month view more compact on mobile; dots-only mode on ≤400px screens

**Shopping summary**
- Replaced orange-to-green gradient with solid `var(--bg-white)`

**Settings / Dictionary**
- `.dictionary-stats` uses `flex-wrap: wrap` to prevent text overflowing narrow screens
- Aisle `<select>` in dictionary table has `max-width: 140px`

**Account page**
- Profile card: email truncates with ellipsis; Sign Out button has `flex-shrink: 0`
- Household member rows: `flex-wrap: wrap`; email truncates; role badge and remove button don't shrink

### Theme picker
The legacy theme picker (Garden / Terracotta / Nordic / Berry / Amber / Ink) has been **removed** from the redesign branch. The app ships with Warm Pantry only. Themes will be reintroduced in a future iteration designed specifically for the Warm Pantry token system.

What was removed: `[data-theme]` CSS blocks, Theme Selector UI CSS, Appearance section in account view, localStorage theme loader, `themes` object, `applyTheme` / `renderThemeSelector` / `selectTheme` functions, `theme` field from `saveUserPrefs`. Any `theme` value previously stored in Firestore `settings/userPrefs` is harmlessly ignored.

When reintroducing themes, each theme block should override: `--bg-cream`, `--bg-white`, `--text-primary`, `--text-secondary`, `--accent-green`, `--accent-green-light`, `--accent-green-dark`, `--accent-mint`, `--border-light`, `--border-medium`, `--shadow-*`, `--font-heading`, `--font-body`. The `--color-*` pastel variables and `--radius-*` values should remain fixed.

### Deploy checklist (before merging redesign → main)
1. Increment `CACHE_NAME` in `sw.js` (currently `kitchenlistr-v1` → `kitchenlistr-v2`) to force cache refresh for all users
2. Merge `redesign` branch into `main` via PR
3. GitHub Pages deploys automatically on push to `main`
