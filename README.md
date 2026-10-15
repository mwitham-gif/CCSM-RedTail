# Red Tail Crossing Resource Guide

Static, mobile-first community resource directory for Community Corporation of Santa Monica, serving residents around Red Tail Crossing (8333 Airport Blvd, Westchester, CA 90045) and the surrounding Westchester, Playa del Rey, Inglewood and LAX-area neighborhoods.

It is a copy of the Santa Monica Resource Guide (https://github.com/mwitham-gif/CCSM) with its own Google Sheet. Design and behavior are the same; only the place names, the sheet and the report-email inbox differ.

The site is designed for a QR-code journey: residents scan a flyer, search or choose a support area, then call or visit a provider directly.

Live site:

https://mwitham-gif.github.io/CCSM-RedTail/

## Project Files

- `index.html` - page shell, metadata, CSP, and static layout
- `styles.css` - visual design, responsive layout, accessibility states
- `app.js` - Google Sheet loading, CSV parsing, search/filter/share behavior
- `analytics.js` - Google Analytics bootstrap and custom events, kept out of `index.html` so the
  Content Security Policy does not need to allow inline scripts
- `ccsm_logo_web.png` - logo and social preview image
- `favicon-32.png`, `apple-touch-icon.png` - browser tab and home-screen icons

## How It Works

1. Staff update the Google Sheet.
2. The sheet is published as CSV.
3. `app.js` fetches the CSV, normalizes rows, and renders resource cards.
4. GitHub Pages serves the static files from `main`.

The app intentionally does not use a backend, build step, external font request, or client-side framework. The only third-party script is Google Analytics. Google Tag Manager was removed in October 2026: its three tags duplicated every link tap and leaked search text.

## Privacy And Security Defaults

- Google Analytics 4 (property `G-FT2K5G6SYR`, shared with the Santa Monica site; filter reports by page path `/CCSM-RedTail/` to separate them) records page views. It sets its
  own cookies and reports to Google. `analytics.js` also sends three custom
  events with `gtag()`: `resource_click` (call, website, email, and report
  links on a card), `filter_select` (support-area filter buttons), and
  `resource_search` (the search box, 1.5s after typing stops). Search terms
  are sent to Google as typed, so anything a visitor enters there leaves the
  site, with these exceptions: clicks on a domestic violence resource send
  only the category (name and link are `(withheld)`); searches about abuse,
  immigration status, self-harm, or sexual and reproductive health, and
  searches containing an email address or a phone-length number, send
  `(withheld)` instead of the words typed; and page views of a shared link
  report `?share=(withheld)` with the title "Shared resource", so Google never
  learns which resource was shared. Google signals and ad personalization are
  turned off in the page code. This is the one exception to the rules
  below, and it is worth keeping in mind for an audience that reaches the site
  by scanning a flyer for housing, food, or legal help.
- In GA4, turn off Enhanced measurement → Outbound clicks for this stream.
  `resource_click` covers the same taps with the privacy rules above; the
  built-in `click` event would record domestic violence websites' addresses.
- To see the event parameters in reports, register them as event-scoped
  custom dimensions (Admin → Custom definitions): `resource_name`,
  `resource_category`, `click_type`, `filter_name`, `search_term`.
- No other third-party tracking, and no advertising tags.
- On domestic violence views only (the Domestic violence filter, or a shared
  domestic violence resource), a Quick exit button stays on screen (bottom
  right). It replaces the page in
  history with weather.com, so Back does not return here. There is no
  keyboard shortcut on purpose: Esc already clears search and closes menus.
- No external fonts.
- Search terms are not stored in the URL after initial load.
- A Content Security Policy limits scripts and styles to this site, with
  `googletagmanager.com` allowed for the analytics loader. Analytics code lives
  in `analytics.js` rather than inline, so the policy still forbids inline
  scripts.
- Apart from analytics, live data fetches are limited to the published Google
  Sheet host.
- If analytics is blocked, by a content blocker or a network, the directory
  loads and works normally. Nothing about finding a resource depends on it.
- If the sheet cannot load, the site shows an outage message with Resident Services contact information instead of sample/demo data.

## Recommended Sheet Columns

Use these column headers in row 1:

- `name`
- `name_es`
- `category`
- `description`
- `description_es`
- `address`
- `phone`
- `website`
- `hours`
- `notes`
- `notes_es`
- `tags`
- `tags_es`

Only `name` is required, but each resource works best with `category`, `description`, and either `phone` or `website`.

The importer accepts common variations like `Name`, `Resource Name`, `Phone Number`, `URL`, and `Location`, but the recommended names above are more predictable.

## Recommended Categories

Use one of these values in the `category` column:

- `Food`
- `Mental`
- `Rental`
- `Legal`
- `Benefits`
- `Transportation`
- `Education`
- `Employment`
- `Childcare`
- `Youth`
- `Seniors`
- `Housing`
- `Disability`
- `Medical`
- `Domestic Violence`
- `Other`

The app normalizes close matches like `food pantry`, `mental health`, and `public benefits`, but exact values are best.

## Data Entry Tips

- Put one resource per row.
- Keep descriptions short and plain-language.
- Use readable phone numbers like `(310) 555-1212`.
- Use full website URLs when possible, like `https://example.org`.
- Use Spanish columns only when Spanish text is available.
- Use `tags` for short labels like `Walk-ins welcome`, `Appointment required`, `Westchester residents`, or `Families`.
- Separate multiple tags with commas or vertical bars.
- Leave fields blank if they do not apply.

## The Google Sheet

The source sheet is owned by the mwitham@communitycorp.org Google account. Add its edit link here once created.

It was seeded in October 2026 with 75 bilingual listings: countywide services carried over from the Santa Monica sheet, plus new local listings near Westchester and Inglewood. A copy of that starting data is in `data/seed-resources.csv`. The sheet is the source of truth from then on; the CSV is not read by the site.

Local listings were gathered from 211 LA, the LA County services locator, LAPL, LA Rec and Parks, Westside Food Bank and provider websites. Several of those sources disagreed on hours or phone numbers. Have staff call each local listing to confirm it before the QR flyer goes out.

## Updating The Google Sheet Source

The CSV source lives in `app.js`:

```js
const SHEET_URL = '...';
```

Update that value only when moving to a different Google Sheet, changing the published tab, or setting up publishing for the first time.

## Local Preview

From this folder:

```bash
python3 -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

## Deploying

GitHub Pages updates automatically when changes are pushed to `main`:

```bash
git add index.html styles.css app.js README.md
git commit -m "Describe the change"
git push origin main
```

Pages can take a minute or two to refresh.

### Bump The Asset Version When Markup And Script Change Together

`index.html` loads the assets with a version query:

```html
<link rel="stylesheet" href="styles.css?v=2">
<script src="app.js?v=2" defer></script>
```

**If a change edits `index.html` and `app.js` together, raise both numbers
to the next version in the same commit.**

Pages serves `index.html` with a short cache lifetime, but a visitor's
browser can hold `app.js` and `styles.css` for much longer. Without the
bump, a returning visitor can end up running yesterday's script against
today's markup. Changing the number makes the URL a cache miss, so the two
always arrive as a matched pair.

This matters most when an element is removed from `index.html`. A script
that still expects it will write to something that is not there. `app.js`
guards its lookups so a missing element is skipped rather than fatal, but
the version bump is what keeps the pair honest; the guards are the safety
net underneath it.

A CSS-only or script-only change does not need a bump, though one is
harmless.

## Pre-Flyer QA Checklist

Before printing or distributing a flyer QR code:

- Scan the QR code on an iPhone.
- Scan the QR code on an Android phone if available.
- Confirm the page loads on cellular data.
- Search for `food`.
- Tap a support-area filter.
- Tap a call button.
- Tap a website button.
- Switch to Spanish and back to English.
- Reload once with a hard refresh and once normally, so a cached copy of the
  previous `app.js` would show up.
- Text the live link to yourself and confirm the preview looks right.
- Try a broken shared URL like `?share=missing-resource` and confirm the fallback is helpful.

## If Something Looks Wrong

- If no resources load, check whether the Google Sheet is still published as CSV.
- If analytics stops reporting, check that the CSP in `index.html` still allows
  `googletagmanager.com` under `script-src`, and that `analytics.js` is loaded.
  A console message beginning "Refused to load the script" means the CSP is the
  cause.
- If a resource is missing, make sure the row has a `name`.
- If a filter looks odd, check the `category` spelling.
- If a call or website button is missing, check whether `phone` or `website` is blank.
- If the page looks stale after a push, wait a minute and hard refresh.
- If the page loads but shows no resources and no outage message, suspect a
  cached `app.js` running against newer markup. Open the browser console: a
  `TypeError` naming a null element confirms it. Bump the asset version in
  `index.html` as described under Deploying.
