# Coffee Timer

A standalone, single-file coffee brewing timer web app. No build tools, no server — just open `index.html` in any browser on PC or mobile.

## Project Structure

```
index.html   — the entire app (HTML + CSS + JS in one file)
```

## Architecture

Everything lives in `index.html`, organized into three logical sections:

- **`<style>`** — CSS custom properties for theming, mobile-first layout (max-width 480px)
- **`<body>`** — Three screens shown/hidden via JS:
  - `#screen-list` — recipe browser with filter tabs
  - `#screen-recipe` — recipe detail + gram picker
  - `#screen-brew` — active brew player with countdown ring
- **`<script>`** — Recipe data (JSON objects) + all app logic

## Recipe Data Schema

Each recipe in the `RECIPES` array follows this shape:

```js
{
  id: 'unique-id',
  method: 'V60',              // used for filter tabs
  name: 'Recipe Name',
  author: 'Author Name',
  defaultCoffee: 15,          // grams
  defaultWater: 250,          // ml
  grindSize: 'Medium-fine',
  waterTemp: '93°C / 200°F',
  steps: [
    { time: null, label: 'Preparation', description: '...' },  // no timer, shows Continue button
    { time: 0,    label: 'Step Name',   description: '...', tokens: { bloom: w => Math.round(w * 0.2) } },
    { time: 45,   label: 'Next Step',  description: '...' },
    { time: 210,  label: 'Done',       description: '...', last: true },
  ],
}
```

### Description tokens
- `{coffee}` — scaled coffee grams
- `{water}` — scaled water ml
- `{temp}` — water temperature string
- Custom tokens defined per-step in `tokens: { key: (scaledWater) => value }`

### Scaling
When the user changes the gram amount, all values scale proportionally:
- `scaledWater = defaultWater × (userGrams / defaultCoffee)`
- `scaledStepTime = stepTime × scaleFactor`

## Current Recipes (v1)

| Method | Recipe | Author |
|--------|--------|--------|
| V60 | The Ultimate V60 | James Hoffmann |
| V60 | 4:6 Method | Tetsu Kasuya |
| AeroPress | Inverted AeroPress #1 | Wendelien Van Bunnik |
| AeroPress | AeroPress Classic | Tim Wendelboe |
| French Press | French Press (No-Plunge) | James Hoffmann |
| Chemex | Classic Chemex | Chemex |
| Espresso | Classic Espresso | SCA Standard |
| Moka Pot | Stovetop Moka | Traditional Italian |

## Key Functions

| Function | Purpose |
|----------|---------|
| `buildList()` | Renders recipe cards, respects `currentFilter` |
| `openRecipe(id)` | Loads recipe into detail screen |
| `renderDetail()` | Re-renders specs + step preview when grams change |
| `buildScaledSteps(recipe, grams)` | Returns steps with all tokens resolved and times scaled |
| `startBrew()` | Enters brew screen at step 0 |
| `renderBrewStep()` | Renders current step; starts timer or shows Continue |
| `timerTick()` | `requestAnimationFrame` loop; updates ring + display |
| `togglePause()` | Freezes/resumes elapsed time |
| `advanceStep()` | Moves to next step or shows complete overlay |
| `setRingProgress(fraction)` | Updates SVG `stroke-dashoffset` |

## localStorage Keys

| Key | Value |
|-----|-------|
| `cf_favorites` | JSON array of favorited recipe IDs |
| `cf_last_grams` | JSON object `{ recipeId: grams }` |

## Adding a New Recipe

1. Add an entry to the `RECIPES` array in `index.html`
2. Use an existing method string (`'V60'`, `'AeroPress'`, `'French Press'`, `'Chemex'`, `'Espresso'`, `'Moka Pot'`) or add a new filter tab in `#filter-tabs`
3. First step should always have `time: null` (preparation)
4. Last step should have `last: true`
5. Step durations are inferred from the difference between consecutive `time` values

## Branch

Active development branch: `coffee-timer`
