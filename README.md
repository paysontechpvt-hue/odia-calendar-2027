# Odia Calendar 2027 — Odisha Festivals & Panji

A bilingual (Odia / English), mobile-first web calendar for the year **2027**. It shows each
Gregorian date next to its **Odia solar (Panji) date**, highlights every major Odia festival and
public holiday with colours, icons and tooltips, and offers month / year / festival views, search,
festival details, print and PDF export.

The whole app ships as **one self-contained `index.html`** — no build step, no server, no
dependencies to install. Host the file anywhere and it runs.

---

## 1. What's included

| File | Purpose |
|------|---------|
| `index.html` | The complete frontend (HTML + CSS + React). This is the whole app. |
| `README.md` | This document — deployment and customization. |

**Features**

- **Dual dates** — every day shows the Gregorian date and the corresponding Odia solar month + day (e.g. *Ashadha 21, Sāla 1434*).
- **Views** — Month (full grid), Year (12 mini-calendars), Festivals (searchable, filterable list).
- **Navigation** — previous/next month with automatic year roll-over; click any month in Year view to open it.
- **Festivals & holidays** — 46 entries for 2027: Ratha Yatra, Raja Parba, Nuakhai, Durga Puja, Kumar Purnima, Kartika Purnima, Dola Purnima, Diwali, Saraswati Puja, Makar Sankranti and every national / state / other-faith public holiday.
- **Details & tooltips** — click any festival for date (Gregorian + Odia), weekday, significance and traditions; hover any calendar chip for a tooltip.
- **Bilingual** — full Odia ⇄ English toggle for every label, festival name, significance and tradition.
- **Today widget** — current Gregorian date, Odia date, weekday and countdown to the next festival.
- **Print / PDF** — print-optimised stylesheet (Print → *Save as PDF* in any browser) plus a one-click **PDF** button that generates a formatted festival list via jsPDF.
- **Design** — Jagannath / Konark palette (sindoor vermillion, turmeric haldi, indigo, palm-leaf parchment), Konark-wheel hero mark, Odia + Latin typography, light/dark aware, accessible (keyboard, focus, `aria`, reduced-motion).

---

## 2. Deploy (fastest → most production-grade)

### Option A — just host the file
Because everything is inlined, deployment is literally *"put `index.html` on the web."*

- **GitHub Pages** — commit `index.html` to a repo, enable Pages → the site is live.
- **Netlify / Vercel / Cloudflare Pages** — drag-and-drop the folder, or point it at the repo. No build command, publish directory = project root.
- **Amazon S3 + CloudFront** — upload `index.html`, enable static website hosting.
- **Any web server (Nginx / Apache / cPanel)** — copy `index.html` into the web root (e.g. `/var/www/html/`).

Nginx example:

```nginx
server {
    listen 80;
    server_name odiacalendar.example.com;
    root /var/www/odia-calendar;
    index index.html;
    location / { try_files $uri /index.html; }
}
```

### Option B — production hardening (recommended before launch)
The published file uses in-browser Babel and Google-hosted fonts for zero-config convenience. For a
public, high-traffic site, remove those two runtime costs:

1. **Pre-compile the JSX** so browsers never run Babel:
   - Copy the code inside `<script type="text/babel">` into a file `app.jsx`.
   - `npx babel app.jsx --presets @babel/preset-react -o app.js`
   - Replace the `<script type="text/babel">…</script>` block with `<script src="app.js"></script>` and delete the `babel.min.js` line.
2. **Self-host the fonts** (Noto Serif/Sans Oriya, Marcellus, Mukta, Yatra One) with `@font-face` so there's no third-party request. Download the WOFF2 files, drop them beside `index.html`, and swap the Google `<link>` for local `@font-face` rules.
3. Keep **React**, **ReactDOM** and **jsPDF** either as the current CDN links or self-hosted copies.
4. Add a `sitemap.xml`, `robots.txt` and a real `og:image` (see §6).

This turns the page into a fully static, self-contained bundle that loads in one round trip.

### Option C — move into a Next.js / React project (scalable codebase)
The code is already componentised, so lifting it into Next.js is mechanical:

```
odia-calendar/
├─ app/
│  ├─ layout.tsx            # <html>, fonts, metadata (SEO)
│  └─ page.tsx              # renders <Calendar/>
├─ components/
│  ├─ Masthead.tsx          # KonarkWheel + title + toggles
│  ├─ TodayBand.tsx
│  ├─ Toolbar.tsx
│  ├─ MonthView.tsx
│  ├─ YearView.tsx          # + MiniMonth
│  ├─ FestivalView.tsx
│  └─ FestivalModal.tsx
├─ lib/
│  ├─ panji.ts              # SANKRANTI table + odiaDate() engine
│  ├─ i18n.ts               # T / WEEK / GMON strings
│  └─ exportPdf.ts
├─ data/
│  └─ festivals-2027.ts     # the FESTIVALS array
└─ styles/
   └─ globals.css           # the CSS from <style>
```

Migration steps: (1) paste the CSS into `globals.css`; (2) split each `function XView` into its own
component file with `export`; (3) move `SANKRANTI`, `odiaDate`, `T`, `FESTIVALS` into `lib/` and
`data/`; (4) put SEO tags in `app/layout.tsx` `metadata`; (5) `next dev`. Nothing about the logic
changes — Next.js gives you routing, image optimisation, and pre-rendering for even better SEO.

---

## 3. Customize the festival data

All festivals live in one array, `FESTIVALS`, inside `index.html`. Each entry:

```js
{
  date:"2027-07-05",     // ISO Gregorian date (IST civil date)
  cat:"jagannath",       // jagannath | state | national | hindu | seasonal | faith | observance
  hol:true,              // true = gazetted public holiday (strong red highlight + "Public holiday" tag)
  ic:"🛕",               // emoji icon shown on the chip / card / modal
  en:"Ratha Yatra (Car Festival)",
  or:"ରଥଯାତ୍ରା",         // Odia name
  sigEn:"…", sigOr:"…",  // significance, English & Odia
  trEn:"…",  trOr:"…"    // traditions & rituals, English & Odia
}
```

- **Add a festival** — copy any object, edit the fields, drop it into the array. It appears automatically in every view (the app rebuilds its date index and sorts on load).
- **Category colours & labels** live in the `CATS` object and the `.cat-*` CSS classes — change one place, it updates the chip, card border, legend and modal tag.
- **Two festivals on the same day** (e.g. Independence Day + Milad-un-Nabi on 15 Aug) are fully supported — just give both the same `date`.

## 4. Change the year (e.g. build a 2028 edition)

Two things drive the whole app:

1. **`SANKRANTI` table** — the 12 solar-ingress dates that anchor the Odia months. Replace the
   2027 dates with the target year's (plus the preceding December's *Dhanu Sankranti* so early-January
   dates map correctly). Sankranti dates move only ±1 day year to year.
2. **`FESTIVALS` dates** — update each `date`. Solar festivals (Pana Sankranti 14 Apr, Raja 15 Jun,
   Makar Sankranti) barely move; lunar ones (Ratha Yatra, Durga Puja, Diwali, Kumar/Kartika Purnima)
   shift ~11 days earlier each year — take them from a panchang.

The `odiaDate()` engine, views, search, PDF and everything else then work unchanged.

## 5. Customize the look

- **Colours** — every colour is a CSS variable in `:root` (and its dark-mode override). Edit
  `--sindoor`, `--haldi`, `--indigo`, `--leaf`, `--maroon`, `--paper` to re-skin the whole site.
- **Fonts** — swap the Google Fonts `<link>` and the `--f-display / --f-serif / --f-body / --f-odia*`
  variables. Keep a real Odia webfont (Noto Serif/Sans Oriya) so the script renders on all devices.
- **Language strings** — UI labels are in the `T` object (`T.en` / `T.or`); weekday and month names in
  `WEEK_*` / `GMON_*`. Add a third language by adding a key and a toggle button.

## 6. SEO & performance

- Title, meta description, keywords, canonical, Open Graph, Twitter cards and JSON-LD (`WebSite`) are
  already in `<head>`. **Before launch**, set the real domain in the `canonical` and `og:url`, and add
  an `og:image` (1200×630) plus `sitemap.xml` and `robots.txt`.
- For richer search results, extend the JSON-LD with an `ItemList` of `Event` objects generated from
  `FESTIVALS` (name, `startDate`, `location: Odisha`).
- Performance: follow **§2 Option B** (pre-compile JSX, self-host fonts). The app has no images, so it
  is already lightweight; those two steps remove the only render-blocking third-party requests.

---

## 7. Accuracy & sources

Festival dates were compiled from panchang and government sources (Drik Panchang, Samvat, the DoPT
2027 gazetted-holiday list, and Odia-specific Panji references) and cross-checked. Please note:

- **Lunar festivals** (Ratha Yatra, Durga Puja, Diwali, Kumar/Kartika Purnima, Prathamastami, etc.)
  and **Islamic festivals** (Eid-ul-Fitr, Eid-ul-Adha, Muharram, Milad-un-Nabi) can vary by **±1 day**
  with local Panji or moon sighting.
- **Makar Sankranti** is listed on 14 January; some panchangs observe it on 15 January in 2027.
- Always confirm **office closures** against the **official Odisha Government gazette** before relying
  on a date for administrative purposes.

---

*Built as a single-file React app. Odia solar (Panji) dates are computed from the 2027 Sankranti table;
the Vilayati Sāla year rolls over on Kanya Sankranti (~17 September).*
