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

#### 🔲 Phase 4 — Shopping list upgrades
- Item cards: add meal name + recipe date as second line of text (DM Sans small / DM Mono for date)
- Sort/filter chip row at top of shopping list
- Date range navigator (prev/next arrows + label) for selecting which week's meals to include
- Progress bar (herb green fill, shows X/Y items checked)
- "Clear checked" floating pill button
- Tomato FAB for adding ad-hoc extra items

#### 🔲 Phase 5 — Planner reskin
- Week strip: pill-shaped day selectors, horizontal scroll, tomato active day
- Week/month toggle: keep existing logic, restyle toggle as pill pair
- Month grid: rounded cells, tomato for today/active date, small category-coloured dots for meals
- Empty meal slot: dashed border, muted style
- Tomato FAB for adding meals

#### 🔲 Phase 6 — Emoji icon system (aisles)
- Aisle create/edit form: add emoji picker tap target next to name field
- Shopping list aisle headers: display emoji, fallback to coloured circle with initial letter
- Simple emoji picker: a popover grid of food-relevant emoji (no third-party library)

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
