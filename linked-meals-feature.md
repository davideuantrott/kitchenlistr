# Linked Meals Feature — Handoff Notes

## What we were trying to do

Allow a user to mark a meal plan entry as "same as" another day's meal (e.g. cooking a big batch on Monday, eating leftovers on Tuesday). The linked meal should:

- Appear in the planner with a distinct visual indicator (italic, dashed border)
- Show which meal it references: "↩ Lasagne · Mon"
- Contribute **no ingredients** to the shopping list (avoids double-counting)

## Data model: Option B — prefix in Firestore

Store the linked entry as a string in `mealPlan[dateStr][mealKey]`:

```
linked:YYYY-MM-DD_mealKey
e.g. linked:2026-03-09_dinner
```

Fully backward-compatible: the shopping list already does `recipes.find(r => r.id === recipeId)`, which returns `null` for a `linked:` ID, so those entries are silently skipped. No migration needed.

## What is already in index.html (committed to main)

### CSS (around line 1435)

```css
.meal-slot-linked {
    background: var(--bg-cream);
    border: 1.5px dashed var(--accent-green-light);
    opacity: 0.85;
}
.meal-slot-linked-name {
    font-weight: 400;
    font-style: italic;
    color: var(--text-secondary);
    font-size: 0.82rem;
}
.month-meal-pill.linked {
    opacity: 0.7;
    font-style: italic;
}
```

### Helper functions (around line 6661)

```js
parseLinkedMeal(recipeId)       // returns { targetDateStr, targetMealKey } or null
getLinkedRecipe(recipeId)       // resolves to actual recipe object (or null if removed/chain-linked)
getLinkedDisplayText(recipeId)  // returns e.g. "↩ Lasagne · Mon"
```

## What still needs to be done

### a. `renderPlanner()` — week view (around line 6717)

Currently the slot template does:

```js
const recipe = recipeId ? recipes.find(r => r.id === recipeId) : null;
// class: meal-slot${recipe ? '' : ' meal-empty'}
// name:  ${recipe ? `<div class="meal-slot-name">${recipe.name}</div>` : `<div class="meal-slot-empty">+ Add meal</div>`}
```

Replace with:

```js
const isLinked = recipeId ? recipeId.startsWith('linked:') : false;
const recipe = (!isLinked && recipeId) ? recipes.find(r => r.id === recipeId) : null;
const hasEntry = isLinked || !!recipe;
// class: meal-slot${hasEntry ? (isLinked ? ' meal-slot-linked' : '') : ' meal-empty'}
// name:  isLinked  → <div class="meal-slot-linked-name">${getLinkedDisplayText(recipeId)}</div>
//        recipe    → <div class="meal-slot-name">${recipe.name}</div>
//        empty     → <div class="meal-slot-empty">+ Add meal</div>
// buttons: use hasEntry instead of recipe for showing Remove/Change buttons
```

### b. `renderMonthView()` (around line 7800)

When building the meals array for each day, add a branch:

```js
if (recipeId && recipeId.startsWith('linked:')) {
    meals.push({ name: getLinkedDisplayText(recipeId), isLinked: true });
}
```

Then add `.linked` class to the pill when `m.isLinked` is true.

### c. Print functions (around lines 7813 and 7861)

In `generateWeekPrintHtml()` and `generateMonthPrintHtml()`:
if `recipeId.startsWith('linked:')`, show `getLinkedDisplayText(recipeId)` instead of blank/dash.

### d. `#select-recipe-modal` — Link form HTML

Add an inline collapsible section at the bottom of the modal:

```html
<div class="link-meal-toggle">
  <button onclick="toggleLinkForm()">↩ Link to another day's meal instead</button>
</div>
<div id="link-meal-form" style="display:none">
  <label>Date <input type="date" id="link-target-date"></label>
  <label>Meal
    <select id="link-target-meal"><!-- populated by toggleLinkForm() --></select>
  </label>
  <button onclick="confirmLinkMeal()">Link</button>
</div>
```

### e. JS functions

```js
function toggleLinkForm() {
    const f = document.getElementById('link-meal-form');
    const sel = document.getElementById('link-target-meal');
    if (!sel.options.length) {
        getMealTypes().forEach(m => {
            const key = m.toLowerCase().replace(/[^a-z0-9]/g, '_');
            sel.add(new Option(m, key));
        });
    }
    f.style.display = f.style.display === 'none' ? 'block' : 'none';
}
window.toggleLinkForm = toggleLinkForm;

async function confirmLinkMeal() {
    const date = document.getElementById('link-target-date').value;
    const meal = document.getElementById('link-target-meal').value;
    if (!date || !meal) return;
    const linkedId = 'linked:' + date + '_' + meal;
    await selectRecipeForMeal(linkedId); // reuses existing Firestore save path
}
window.confirmLinkMeal = confirmLinkMeal;
```

## Test plan

1. Add a recipe to Monday's Dinner slot
2. Open Tuesday's Dinner slot → choose "Link to another day" → pick Monday / Dinner → Link
3. Verify Tuesday shows the linked label in week view (italic, dashed border)
4. Verify Tuesday shows the linked pill in month view
5. Verify the shopping list has no duplicate ingredients for Tuesday
6. Verify the print output shows the linked label rather than blank

## Key locations in index.html

| Area | Approx line |
|---|---|
| Linked meal CSS | 1435 |
| `parseLinkedMeal` / `getLinkedDisplayText` helpers | 6661 |
| `getMealTypes()` (just after helpers) | 6686 |
| `renderPlanner()` week slot template | 6717 |
| Month view meal pill loop | ~7800 |
| `generateWeekPrintHtml()` | ~7813 |
| `generateMonthPrintHtml()` | ~7861 |
| `#select-recipe-modal` HTML | search `id="select-recipe-modal"` |
| `selectRecipeForMeal()` JS | search `async function selectRecipeForMeal` |
