# Build The Ultimate Ski Weather Dashboard (And Have Fun Doing It)

I’ve always loved the mix of engineering and mountain stoke. This season, I set out to create the ultimate Big White snow dashboard—live conditions, slick charts, share-ready images, even a ChatGPT trip planner. The whole thing turned into a vibe-coded, zero-dependency HTML page you can drop on Netlify or any static host. Here’s the complete recipe so you can remix it for your home mountain.

→ **Live Demo:** https://bw-report.netlify.app/  
→ **Repo:** https://github.com/jessekorzan/snowlab

---

## Why a DIY Dashboard?

- Resort sites bury data across multiple pages.
- Weather APIs are richer than ever (shoutout Open-Meteo).
- Creating your own view lets you surface what matters: freezing levels, wind holds, storm trends, historical comparisons.

Most importantly, this project is pure front-end HTML/CSS/JS—no build step, just vibes.

---

## Stack + Ingredients

- **Tailwind CDN** for fast styling without setup.
- **Inter font** for clean typography.
- **Open-Meteo API** (free, weather + historical).
- **Chart.js** (CDN) for interactive charts.
- **Plain JavaScript modules** for data transforms, caching, and rendering.
- **Social image generator** via Pillow to produce OG/Twitter cards.
- **Optional analytics** (Microsoft Clarity snippet) for scroll heatmaps.

Everything sits inside one file (`index.html`) except the generated images. It’s refreshingly old-school.

---

## Project Structure

```
.
├── index.html
└── assets/
    └── social/
        ├── og-image.png
        └── twitter-card.png
```

That’s it—no `package.json`, no bundler.

---

## Step 1: Bootstrapping the Page

Start your HTML with the usual head tags, but give SEO some love:

- `<meta>` description + keywords.
- Canonical link pointing to your live URL.
- Open Graph + Twitter cards referencing the social images you’ll generate later.
- JSON-LD schema: `SportsActivityLocation` with coordinates, publisher name, and social profiles.

Then drop the utility CDNs:

```html
<script src="https://cdn.tailwindcss.com"></script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
```

Finally, add a tiny CSS block for custom pieces (mega stats, horizontal chart scroll, mobile font scaling).

---

## Step 2: Layout the Dashboard

The page uses a responsive grid: main content + a right column for webcams/7-day forecast.

Key left-column sections:

1. **Current Conditions** – hero stats with big typography.
2. **24-Hour Forecast** – horizontal scroll cards using live data.
3. **Mountain Insights** – visibility, cloud cover, freezing level, liquid precip.
4. **Snow Trend Chart** – historic vs forecast snowfall line chart.
5. **Temperature Profile** – actual vs feels like with sub-freezing shading.
6. **Freezing Level Tracker** – freezing level with elevation bands.
7. **Wind vs Gusts** – dual-axis line, hold threshold band.
8. **Yesterday vs Today** – quick numeric comparison.

On the right: webcams CTA, sunrise/sunset, 7-day forecast.

Pro tip: wrap chart canvases in `.chart-scroll` containers to enable horizontal swipe on mobile.

---

## Step 3: Fetching Weather Data

We rely on Open-Meteo (no API key). Build a single URL that requests:

- Current: `temperature_2m`, `apparent_temperature`, `weather_code`, `wind_speed_10m`, `wind_gusts_10m`, `snowfall`, `snow_depth`.
- Hourly: add `precipitation`, `snowfall`, `precipitation_probability`, `freezing_level_height`, etc.
- Daily: high/low temps, snowfall sum, precipitation, wind maxima, sunrise/sunset.
- `timezone=America/Vancouver` (adjust per resort).
- `windspeed_unit=kmh`, `precipitation_unit=mm`.

> 🔁 **Swap resorts fast:** Update the `APP_CONFIG` object near the top of `index.html` with your latitude, longitude, timezone, resort name, and share URL. Be sure to sync the `<head>` metadata (title, description, JSON-LD, canonical link) so it reflects your mountain.

Add a simple retry helper with exponential backoff.

---

## Step 4: Local Cache

To keep the UI snappy, cache the entire payload (forecast + historical) in `localStorage` for 10 minutes. On load:

1. Try reading cached snapshot; if present, render it instantly with a “cached snapshot” badge.
2. Fetch fresh data in the background and update everything.

---

## Step 5: Render Helpers

Implement a render pipeline:

```js
function renderAll(forecast, historical, options) {
  updateCurrentConditions(...);
  updateHourlyForecast(...);
  updateMountainInsights(...);
  updateSunlight(...);
  updateTemperatureProfileChart(...);
  updateFreezingLevelChart(...);
  updateWindGustChart(...);
  updateSnowTrendChart(...);
  updateHistoricalComparison(...);
  updateChatPlanner(...);
  update7DayForecast(...);
}
```

Each updater:

- Pulls data out safely (null checks).
- Applies formatting helpers (`formatValue`, `formatKm`, `formatDuration`).
- Uses Chart.js for visuals (register custom plugins for freezing-level bands and wind hold shading).
- Handles placeholders when data is missing.

---

## Step 6: Historical Data & Comparisons

Open-Meteo’s archive API lets us grab the last seven days:

```
https://archive-api.open-meteo.com/v1/archive?latitude=...&start_date=YYYY-MM-DD&end_date=YYYY-MM-DD&daily=temperature_2m_max,temperature_2m_min,snowfall_sum&...
```

Use this to:

- Draw the Snow Trend chart (observed vs forecast).
- Populate the “Yesterday vs Today” block with current vs previous day snowfall/temp/gusts.
- Provide historical context for the ChatGPT planner.

---

## Step 7: ChatGPT Trip Planner Integration

This was the surprise killer feature.

1. Create a `ChatGPT Trip Planner` card with a “Copy SnowLab JSON” button and “Open ChatGPT” link.
2. Build a structured payload combining:
   - Current conditions.
   - Hourly next 24 hours (temperature, snowfall, wind, freezing level).
   - Upcoming 7 days.
   - Historical past 7 days snowfall/precip.
   - Metadata (resort, generated timestamp, share URL).
3. Render the JSON in a `<pre>` for preview, auto-update whenever data refreshes.
4. Use the Clipboard API (with fallback) to copy the JSON.
5. Suggest a prompt: “Use this SnowLab data to plan tomorrow’s ski day.”

Now anyone can paste into ChatGPT and ask for gear recs, lap order, or safety advice.

---

## Step 8: OG/Twitter Images

Generate social share cards so the site looks pro when linked. With Python + Pillow:

- Create a 1200x630 gradient background.
- Draw stylized mountains + stat cards.
- Save as `assets/social/og-image.png` and `assets/social/twitter-card.png`.

It’s a quick script that runs once and makes your meta tags real.

---

## Step 9: Deployment & Polishing

- Drop the repo into Netlify, Vercel, or GitHub Pages.
- Update `<link rel="canonical">`, OG URL, and JSON-LD `url` to match the live domain.
- Check the page in mobile emulators to ensure horizontal chart scroll + header CTA look clean.
- Run Google’s Rich Results Test to verify structured data.
- Optional: add analytics snippet before the closing `</head>`.

---

## Vibe Coding Tips

- Keep everything in one file while iterating; swap to components only if needed later.
- Use `console.log` generously to inspect API responses before wiring to the DOM.
- When designing charts, test with extreme values (big storm, zero snow, high winds).
- Think like a skier: what questions do you ask at 6 AM? Surfacing those answers should guide layout decisions.

---

## Next Ideas to Explore

- Add user-configurable resort coordinates to support multiple mountains.
- Toss in historical trend comparisons (e.g., this week vs same week last season).
- Integrate push notifications via PWA for storm alerts.
- Layer in avalanche bulletin feeds or grooming reports if your resort publishes them.

---

## Why This Matters

Weather apps are everywhere, but most are generic. Building your own lets you:

- Prioritize the metrics you ride on.
- Share a polished tool with friends or social followers.
- Learn APIs, data viz, and UX in a weekend.
- Create a unique asset for your personal brand or ski crew.

Plus, it just feels rad to load up your dashboard and see everything you care about—no distractions, all vibes.

---

## Ready To Build Yours?

Grab the repo at **https://github.com/jessekorzan/snowlab**, clone the structure, swap in your resort coordinates, and start tweaking. Once you’ve got your data flowing, don’t forget to:

1. Generate the ChatGPT JSON snapshot.
2. Add your own flavor to the charts.
3. Ship some social images.
4. Share it with your mountain family.

If you do spin up your own, tag me—I’d love to see what you surface for your hill. Happy coding, and even happier pow hunting!

---

## License

Released under the [MIT License](LICENSE).
