# ⚙ DSP Production Planner

An interactive, client-side production chain calculator and visualizer for **Dyson Sphere Program**.

![DSP Production Planner Screenshot](screenshot.png)

---

## Features

| Feature | Details |
|---|---|
| 🎯 **Target item selector** | Searchable dropdown of all 170+ producible items |
| 📈 **Rate scaling** | Set any items/min target; all downstream rates auto-scale |
| 📊 **Visual graph** | Directed SVG graph — right = target, left = raw resources |
| 🖱 **Interactive graph** | Pan, zoom (scroll), and drag nodes freely |
| ⚙ **Alternate recipes** | Switch between recipes per item; graph updates instantly |
| 🏭 **Building tiers** | Choose assembler/smelter/etc. tier per step; rates recalculate |
| 🪨 **Raw resources** | Summary of all raw inputs per minute |
| ⚠ **Bottleneck highlight** | Items requiring >10 buildings flagged in amber |
| 💾 **Save/Load config** | Export your full plan to JSON; reload any time |
| ⚡ **Power estimate** | Total estimated energy consumption for the chain |

---

## Quick Start (Local)

No build step required — it's a single HTML file.

```bash
# 1. Put these two files in the same folder:
#    - index.html
#    - DSP_data.json

# 2. Serve with any static server, e.g.:
npx serve .
# or
python3 -m http.server 8080
# or
php -S localhost:8080

# 3. Open http://localhost:8080 in your browser
```

> ⚠️ **Important:** You must serve the files via HTTP (not `file://`). Browsers block
> `fetch()` calls from `file://` for security reasons. Any static server works.

---

## GitHub Pages Deployment

### Option A — Manual (simplest)

1. Create a new GitHub repository
2. Upload `index.html` and `DSP_data.json` to the repository root
3. Go to **Settings → Pages → Source → Deploy from branch → main / root**
4. Your planner will be live at `https://YOUR_USERNAME.github.io/REPO_NAME/`

### Option B — GitHub Actions (automated)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
```

Then push — GitHub Actions deploys automatically on every commit to `main`.

---

## File Structure

```
your-repo/
├── index.html          ← Single-file React app (self-contained)
├── DSP_data.json       ← Game data (recipes, buildings, icons)
├── icons/              ← Optional: item icon images
│   ├── 25px-Icon_Refined_Oil.png
│   └── ...
└── README.md
```

---

## Data Format (`DSP_data.json`)

The app uses the JSON as its single source of truth. Structure:

```jsonc
{
  "recipes": [
    {
      "ingredients": [{ "item": "iron ore", "quantity": 1 }],
      "output":      [{ "item": "iron ingot", "quantity": 1 }],
      "time": 1,
      "crafting_building_type": "smelter",
      "minable": false
    }
    // ... 185 total recipes
  ],
  "items icons": [
    { "iron ingot": "icons/25px-Icon_Iron_Ingot.png" }
    // ... one entry per item
  ],
  "factory buildings": {
    "assembler": [
      { "tier": 1, "name": "assembling machine mkI",  "production_speed_factor": 0.75, "energy_consumption": 270 },
      { "tier": 2, "name": "assembling machine mkII", "production_speed_factor": 1,    "energy_consumption": 540 },
      { "tier": 3, "name": "assembling machine mkIII","production_speed_factor": 1.5,  "energy_consumption": 1080 },
      { "tier": 4, "name": "recomposing assembler",   "production_speed_factor": 3,    "energy_consumption": 2700 }
    ]
    // ... smelter, chemical plant, matrix lab, etc.
  }
}
```

---

## How Calculations Work

### Rate computation

For a target item at rate `R` items/min using a building with speed factor `S`:

```
ratePerBuilding = (outputQty / recipeTime) × 60 × S   [items/min per building]
buildingsNeeded = R / ratePerBuilding
cyclesPerMin    = R / (outputQty × S × 60/recipeTime)
ingredientRate  = ingredientQty × cyclesPerMin         [items/min needed]
```

The system recurses through all ingredients, accumulating rates for shared dependencies.

### Alternate recipes & by-products

- Items with multiple recipes show a dropdown — changing it instantly recalculates the full chain
- By-products (e.g., hydrogen from oil refining) are tracked and displayed but don't recurse further

### Building tiers

Each building type has 1–4 tiers with different `production_speed_factor` values. Selecting a higher tier means fewer buildings are needed for the same output rate.

---

## Save / Load Format

Configs are saved as JSON:

```json
{
  "selItem": "electromagnetic matrix",
  "rate": 60,
  "recipeCh": { "graphene": 42 },
  "tierCh": { "iron ingot": 1 }
}
```

- `selItem` — target item name (lowercase)
- `rate` — items per minute
- `recipeCh` — map of item → chosen recipe ID (index in recipes array)
- `tierCh` — map of item → chosen tier index (0-based)

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Blank page / no data | Ensure `DSP_data.json` is in the same folder and you're using HTTP, not `file://` |
| Icons not showing | Place icon images at the paths referenced in `DSP_data.json` (e.g., `icons/25px-Icon_Iron_Ore.png`) |
| Graph is very wide | Use scroll-to-zoom out, or click ⟳ to reset/fit the view |
| Rate looks wrong | Check building tier selection — higher tiers produce more per building |

---

## Technology

- **React 18** (via CDN — no build step)
- **Babel Standalone** (JSX transform in browser)
- **Pure SVG** graph rendering with pan/zoom/drag
- Zero dependencies to install

---

## License

MIT — use freely.
