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
- Main content gets `padding-bottom: calc(72px + 1rem)` on mobile to clear the fixed nav

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

#### 🔲 Phase 7 — Recipe screen
- Hash-based pastel avatar: deterministic colour from recipe name, large emoji centred (or image if URL provided)
- Recipe form: add optional image URL field
- Category filter chips at top of recipe list (auto-populated from shared category list)
- Two-column card grid — test at 480px before committing; revert to single-column if titles clip

#### 🔲 Phase 8 — Blob decorations & animations (additive, lowest priority)
- Decorative CSS blob shapes on screen headers (blush, buttercup, sage pastels)
- Blob float + morph keyframe animations
- Card pop-in, list stagger, spring check animation
- All animations behind `@media (prefers-reduced-motion: reduce)`
