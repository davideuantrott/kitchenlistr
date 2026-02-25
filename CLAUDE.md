# KitchenListr – Project Guide for Claude Code

## Project Overview
KitchenListr is a meal planning and shopping list PWA built with vanilla HTML, CSS, and JavaScript. It is hosted on GitHub Pages and uses Firebase as its backend/database.

## Tech Stack
- **Frontend:** Vanilla HTML, CSS, JavaScript (no framework)
- **Backend/Database:** Firebase (Firestore)
- **Hosting:** GitHub Pages
- **PWA:** manifest.json + service worker (sw.js) for offline support and installability
- **Fonts:** Google Fonts (Fraunces + Source Sans 3)

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
