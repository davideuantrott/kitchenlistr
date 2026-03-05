# KitchenListr — Feature Brainstorm Session

> **How to use this file:**
> Open a new Claude session and paste this prompt:
> *"I'd like to run a feature brainstorm session for my app, KitchenListr. Please read the context in BRAINSTORM-SESSION.md and then facilitate a structured ideation session with me — working through the prompts, critiquing the current platform, and helping me evaluate and prioritise new ideas."*

---

## 1. App Overview

**KitchenListr** is a meal planning and shopping list PWA for personal and household use.

### What it does
- Store and manage recipes (name, ingredients, categories, emoji, image)
- Plan meals by day across a week or month view
- Auto-generate a shopping list from the meal plan for a selected date range
- Manage shops and aisle layouts; assign ingredients to aisles for sorted shopping
- Share recipes via link; household mode for real-time shared meal planning
- Install as a PWA on mobile and desktop; works offline

### Tech stack
- Vanilla HTML/CSS/JS — single `index.html` file, no framework or build step
- Firebase Auth (email/password + Google OAuth)
- Firestore for all persistent data (recipes, meal plan, shopping list, settings)
- GitHub Pages hosting (static, HTTPS, auto-deploy on push to `main`)
- PWA: service worker + manifest for installability and offline support

### Current feature set (as of Phase 12)
| Area | Features |
|---|---|
| Recipes | Add/edit/delete; ingredients; categories (multi-tag); custom emoji; image URL; avatar system; share by link; filter chips by category |
| Meal Planner | Week + month view; day strip navigation; day notes; FAB for quick add; print plan; meal categories configurable in settings |
| Shopping List | Auto-generated from meal plan; sort by A–Z / Aisle / Date / Recipe; date range navigator + presets; progress bar; hide checked toggle; add extra items; Clear Checked pill; Export (text + CSV); aisle emoji headers |
| Settings | Shops & aisles (emoji per aisle); ingredient dictionary (aisle assignments); meal categories |
| Account | Personal vs household mode toggle; household invite links; member roles (owner/admin/member); profile card |
| UX | "Warm Pantry" design system; tomato/blush/herb palette; Fraunces + DM Sans fonts; blob animations; card pop-in; spring check animation; mobile bottom nav; desktop header nav; frosted glass header |

---

## 2. Friction Audit

Areas where the current experience is heavier than it should be:

### High friction
- **Recipe entry** — adding ingredients one by one is slow. No import from URL, no batch import from a text block structured as a recipe. Users with many recipes face a long manual data-entry task upfront.
- **First-time setup** — new users must set up shops, aisles, and the ingredient dictionary before the shopping list becomes useful. There's no guided onboarding or sample data.
- **Aisle assignment** — the ingredient dictionary requires manually assigning every ingredient to an aisle. For a household with 200+ ingredients, this is a significant one-time burden.

### Medium friction
- **Meal plan recipe selector** — still a plain text list (card grid with avatars is a planned improvement). With many recipes, finding the right one is slow.
- **No quick-add from shopping list** — if you realise you're missing a recipe while shopping, there's no shortcut to the planner from the shopping list.
- **Recipe discovery within the app** — no way to browse recipes by what you feel like eating (e.g. filter by cooking time, cuisine, ingredient you have). Search is name-only.
- **No repeat / copy meal** — to repeat a meal from last week you have to manually re-add it. There's no "copy this week" or "use last week's plan".
- **Household suggestions UX** — suggestions panel (household mode) is powerful but not prominent enough; easy to miss.

### Low friction (but worth noting)
- No feedback when a recipe is saved (relies on the sync status toast which is small)
- No way to reorder ingredients within a recipe
- Shopping list doesn't show which shop you're shopping at as a visual header
- No dark mode

---

## 3. Known Gaps & Deferred Features

| Feature | Status | Notes |
|---|---|---|
| Recipe import from URL | Not started | Parse schema.org/Recipe JSON-LD via CORS proxy or Cloudflare Worker |
| Meal plan recipe selector redesign | Deferred (Phase 13) | Card grid with avatars; see CLAUDE.md Future Phase note |
| Recipe image upload | Deferred | Firebase Storage; currently URL-only |
| Theme system | Removed, to be reintroduced | Needs redesign for Warm Pantry token system |
| Native iOS/Android app | Not started | Capacitor wrapper recommended |
| Public shop layout library | Not started | `publicShops/` Firestore collection; browse + clone |
| Public ingredient library | Not started | Shop-specific ingredient→aisle mappings; crowdsourced |
| Logo & app icon | Not started | Needs brand brief from owner |
| Nutrition information | Not started | Per-ingredient or per-recipe; API integration needed |
| Cooking mode / step-by-step | Not started | Recipe instructions field not yet supported |
| Meal history / analytics | Not started | What have we eaten? Most-cooked recipes? |
| Push notifications | Not started | "You haven't planned this week yet"; Capacitor plugin needed |

---

## 4. Brainstorm Prompts

Use these to drive the session. Work through them one at a time, generating ideas before evaluating:

### 4.1 Effortlessness
- What would make adding a new recipe feel instant rather than laborious?
- If a user could plan an entire week in under 60 seconds, how would that work?
- What information could the app infer or suggest, rather than asking the user to enter?

### 4.2 Discovery & Inspiration
- How might the app help users answer "what should we have for dinner tonight?"
- What signals could the app use to make recipe suggestions? (season, what's already planned, what was cooked recently, what's in the fridge)
- Could the app learn from usage patterns to surface relevant recipes?

### 4.3 Household & Social
- What does a family of 4 need that a single person doesn't?
- How could the household suggestions feature be made more engaging or game-like?
- Could friends or family outside the household share or inspire recipes?

### 4.4 The Shopping Experience
- What would make the in-store shopping experience as frictionless as possible?
- What information is missing from the shopping list that would be useful in the moment?
- How could the app handle "I already have this at home" more intelligently?

### 4.5 Reducing Load
- What's the most time-consuming thing a user has to do right now that could be automated?
- What data could be imported from other apps or services (e.g. supermarket loyalty apps, recipe websites, calendar apps)?
- Where does the app ask users to make decisions that could have smart defaults?

### 4.6 Adjacent Problems
- What problems do users have around food and cooking that KitchenListr doesn't address at all?
- Could the app help with food waste, budget, or nutrition?
- What would a "pro" or power-user version of this app look like?

### 4.7 Critique
- Where does the current UI feel confusing or unintuitive for a new user?
- What features exist but are underused — and why?
- If you were designing this from scratch today, what would you do differently?

---

## 5. Idea Capture Template

For each idea generated, record it using this structure:

```
### [Idea Name]
**What it does:** [1-2 sentence description]
**User value:** [What problem does it solve? Who benefits most?]
**Effort estimate:** Low / Medium / High / Unknown
**Dependencies:** [Other features, external services, or platform changes needed]
**Priority signal:** Must-have / Nice-to-have / Speculative
**Notes:** [Any concerns, alternatives, or follow-up questions]
```

---

## 6. Ideas Already in the Backlog

(Capture new ideas above these; don't duplicate)

- Recipe import from URL (scrape schema.org/Recipe)
- Meal plan recipe selector card grid
- Recipe image upload (Firebase Storage)
- Theme system for Warm Pantry
- Native iOS/Android app (Capacitor)
- Public shop layout library
- Public ingredient library (crowdsourced, shop-specific)
- Logo and app icon design
- Cooking mode (step-by-step recipe instructions)
- Meal history and analytics
- Push notifications (Capacitor)
