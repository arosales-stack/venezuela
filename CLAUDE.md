# 🚨 ABSOLUTE RULE — NEVER MODIFY REPO STRUCTURE WITHOUT EXPLICIT PERMISSION

**Claude must NEVER:**
- Force-reset any branch (`force: true` on any git ref)
- Delete or recreate branches
- Push to any branch other than `dev` or `main`
- Touch `wrangler.json`, `robots.txt`, `sitemap.xml`, `CLAUDE.md` or any file other than `index.html` without explicit instruction
- Modify any Cloudflare, GitHub, or account configuration of any kind

**The only permitted operations are:**
- Read files from the repo
- Push `index.html` to `dev` (always) or `main` (only when explicitly told)

Any deviation from this is a critical failure. These rules exist because unauthorized changes to branch history broke the Cloudflare deployment pipeline during an active humanitarian crisis.

---

# ⚠️ CRITICAL RULES — READ FIRST

## NEVER push to main without explicit consent

**Claude must NEVER push to the `main` branch without Audra explicitly saying "push to main", "update main", "deploy to main", or equivalent.** This is non-negotiable. The live site at ayudave.net serves real users during an active humanitarian crisis. An unauthorized push to main that breaks the site is unacceptable.

All changes go to `dev` first. Only push to `main` when explicitly told to.

---
## NEVER make edits without reading the full code first

Before touching any line of index.html, Claude must read the entire relevant section — not just the line being changed. This means:

- Before editing a card: read the full panel it lives in and verify the div/section structure is intact
- Before inserting content: find the exact insertion point by reading 200+ chars of surrounding context, not guessing
- Before any push: run validate.py and do a full structural audit (panels, country sections, global cards, nav order, JS functions)
- Never make "small edits" on top of unverified structure — every edit assumes the structure is correct, which it may not be after prior edits

The reason this rule exists: Claude repeatedly made tiny targeted edits (replacing strings, inserting cards) without verifying the surrounding structure, causing content to appear in wrong sections, panels to break, and filters to stop working. Reading the full context before every edit prevents this.

---


## How to view the dev preview URL

The dev branch deploys to a Cloudflare Workers preview URL. To see it:

1. Go to **Cloudflare dashboard → Workers & Pages → europeforvenezuela**
2. Click **Deployments** tab
3. Find the latest entry with the **`dev`** branch tag
4. Click the **three dots (...)** next to it
5. Click **"Preview"** — this opens the version-specific URL

The URL format is: `<version-hash>-europeforvenezuela.a-rosales.workers.dev`

Example: `dc6443fc-europeforvenezuela.a-rosales.workers.dev`

This URL is unique to each version. Every new push to dev creates a new hash and a new preview URL. The previous URL still works — it points to that specific version.

**Why this is confusing:** Workers preview URLs are version-specific, not branch-persistent. Unlike Cloudflare Pages (which gives `dev.project.pages.dev` that always updates), each Workers deployment gets a new hash-based URL. You have to grab the new URL from the Cloudflare dashboard after each push.

---

## What `preview_urls: true` does in wrangler.json

Added on June 28, 2026. This enables Cloudflare Workers to generate preview URLs for non-production branch deployments. Without it, dev branch builds were creating versions but generating no accessible URL. With it, every push to dev gets a `<hash>-europeforvenezuela.a-rosales.workers.dev` URL visible in the Cloudflare Deployments tab.

The `workers_dev: true` line was also added at the same time — this ensures the workers.dev subdomain routing is explicitly enabled.

---

# ayudave.net — Complete Technical Documentation

**Site:** https://ayudave.net  
**GitHub:** https://github.com/arosales-stack/venezuela  
**Repo name:** `venezuela`  
**GitHub user:** `arosales-stack`  
**Cloudflare project:** `europeforvenezuela` (Worker, not Pages)  
**Contact email:** suggestions@ayudave.net  
**Last updated:** June 28, 2026

---

## 1. What This Is

A bilingual (EN/ES) single-page directory of verified humanitarian aid initiatives for the June 24, 2026 Venezuela earthquakes (7.2 + 7.5 magnitude). Target audience: Venezuelan diaspora in Europe.

**Three tabs:**
- **Global** — 11 international organizations with factual assessment pills
- **By Country** — 20+ European country-specific donation channels (11 countries)
- **Donate Goods** — physical collection points in Spain (time-sensitive)
- **Methodology** — explains assessment criteria

**What it does NOT do:** endorse, rank, or recommend. Shows facts and flags unknowns.

---

## 2. File Structure

```
/
├── index.html          ← The entire website (single file, ~102KB)
├── wrangler.json       ← Cloudflare Worker config (required for auto-deploy)
```

Everything — HTML, CSS, JavaScript — is in one `index.html` file. No framework, no build step, no dependencies except Google Fonts.

---

## 3. Colors (CSS Variables)

Defined in `:root` at the top of `<style>`:

```css
--bg:        #0D1117   /* page background — near black */
--surface:   #161B22   /* card background */
--surface2:  #1C2128   /* filter button active bg, secondary surfaces */
--border:    #2A2D35   /* card borders, dividers */
--amber:     #E8A020   /* primary accent — used for active tabs, links, warnings */
--amber-dim: #7A5310   /* amber border color on warning boxes */
--text:      #F0EEE9   /* primary text — near white */
--muted:     #B0B4BF   /* secondary text — light grey */
--dim:       #7A7E8A   /* tertiary text — medium grey */
--green:     #4CAF50   /* confirmed/verified pills */
--orange:    #F5855B   /* unused currently */
--sidebar:   240px     /* desktop sidebar width */
```

**Rules:**
- Never use blue — it was removed because it reads as a link/unverified signal
- Green = confirmed. Amber = unconfirmed/partial. Grey = neutral/factual
- Card backgrounds use `--surface`, not `--bg`

---

## 4. Layout

### Desktop (≥768px)
```
[sidebar 240px] | [main content]
```
- Sidebar: navigation, org type filters (Global tab), country filters (By Country tab)
- Main: eyebrow + h1 + subtitle, then tab panels

### Mobile (<768px)
- Sidebar hidden
- `mob-hdr` shown: lang buttons + h1 + subtitle + horizontal tab strip
- Horizontal filter strips inside each panel (wrap, not scroll)

---

## 5. Panel System (CRITICAL)

Each tab is a `<section>` element — **not a `<div>`**. This is intentional and important.

```html
<section data-panel="global">   ← Global tab content
<section data-panel="country">  ← By Country tab content
<section data-panel="goods">    ← Donate Goods tab content
<section data-panel="about">    ← Methodology tab content
```

**Why `<section>` not `<div>`:** Stray `</div>` tags from editing cannot accidentally close a `<section>`. Previous versions used `<div id="panel-x">` and kept breaking because nested edits would close the panel early.

**CSS:**
```css
[data-panel] { display: none; }
[data-panel].active { display: block; padding-bottom: 40px; }
```

**JS — switchTab():**
```js
function switchTab(name) {
  document.querySelectorAll('[data-panel]').forEach(function(p) {
    p.classList.toggle('active', p.dataset.panel === name);
  });
  // also toggles nav-* and mnav-* button active states
}
```

Default active panel: `global` (set via `window.onload`).

---

## 6. Card Types

### Global Org Cards (`<a class="card">`)
Structure:
```
card-head (org name)
card-pills (3 pills: Charity Navigator %, On ground status, Gov. independent)
card-desc (one-line description)
card-footer (category tag + VISIT link)
```
Pills use `.cpill` class with modifiers:
- `.pp-yes` — green (confirmed)
- `.pp-partial` — amber (partial/unconfirmed)
- `.pp-dim` — grey (neutral/not applicable)

### Country Initiative Cards (`<a class="ccard">`)
Structure:
```
ccard-top (org name + type badge)
ccard-backing (who backs them)
ccard-safety (sp-ok / sp-watch pills)
ccard-detail (description)
ccard-footer (VISIT link)
```
Pills:
- `.sp-ok` — green ✓
- `.sp-watch` — amber ⚠

### Goods Cards (`<a class="gcard">` or `<div class="gcard">`)
Structure:
```
gcard-top (initiative name + status badge)
gcard-accepts (what they take)
gcard-location (address/hours/phone)
gcard-pills (verification source)
gcard-footer (VISIT link or "No link")
```
Status badges:
- `.gs-active` — green (active)
- `.gs-weekend` — amber (weekend only)
- `.gs-unconfirmed` — grey

---

## 7. Country Sections

Each country is a `<div class="country-section" data-country="XX">` inside `<section data-panel="country">`.

Current countries and codes:
| Flag | Country | Code |
|------|---------|------|
| 🇪🇸 | Spain | `es` |
| 🇩🇪 | Germany | `de` |
| 🇳🇱 | Netherlands | `nl` |
| 🇮🇹 | Italy | `it` |
| 🇵🇹 | Portugal | `pt` |
| 🇫🇷 | France | `fr` |
| 🇨🇿 | Czech Republic | `cz` |
| 🇦🇹 | Austria | `at` |
| 🇨🇭 | Switzerland | `ch` |
| 🇱🇺 | Luxembourg | `lu` |
| 🇬🇧 | United Kingdom | `gb` |

To add a country, you must update:
1. A new `<div class="country-section" data-country="XX">` inside the country panel section
2. A filter button in `#cf` (mobile strip)
3. A filter button in `#scf` (desktop sidebar)

---

## 8. Bilingual System

Every visible text element has class `t` and `data-en` / `data-es` attributes:

```html
<span class="t" data-en="Visit" data-es="Visitar">Visit</span>
```

`setLang(l)` swaps `innerHTML` of all `.t` elements to the selected language. Default: EN.

**Important:** HTML inside `data-en` / `data-es` attrs must not contain single quotes — use `&rsquo;` etc. Double quotes inside attrs must be encoded as `&quot;`. Never put `<span>` or other HTML tags inside these attributes (breaks innerHTML swap).

---

## 9. JavaScript Functions

| Function | What it does |
|----------|-------------|
| `switchTab(name)` | Shows/hides panels, updates nav active states |
| `filterCountry(btn, code)` | Shows/hides country sections by data-country |
| `filter(btn, tag)` | Shows/hides global org cards by data-tags |
| `setLang(l)` | Switches all `.t` elements to EN or ES |

`window.onload` calls `switchTab('global')` to initialize.

---

## 10. Validation System

**File:** `/home/claude/validate.py`

Run before EVERY push. Checks:
- All 4 section panels present and properly closed
- No old `id="panel-x"` div syntax remaining
- All 11 country codes present inside the country panel
- No country sections leaking outside the country panel
- Filter buttons in both `#cf` (mobile) and `#scf` (desktop) for all countries
- Card counts (global panel must have ≥10 cards)
- No escaped quotes in onclick attrs (breaks JS)
- switchTab uses data-panel selector
- OG meta tags present
- JS functions present

```bash
python3 /home/claude/validate.py
# Must output: PASSED — all checks OK
# If it fails: fix errors before pushing
```

---

## 11. Push Workflow

### To dev (test first):
```python
BRANCH = "dev"
# push to dev, verify at Cloudflare preview URL, then merge to main
```

### To main (production):
```python
BRANCH = "main"  # or omit branch param
```

**Always:**
1. Run `python3 /home/claude/validate.py` first
2. If PASSED → push
3. If FAILED → fix, re-run, then push

### GitHub API push pattern:
```python
import base64, json, urllib.request

TOKEN = "ghp_..."  # GitHub personal access token
REPO  = "arosales-stack/venezuela"
PATH  = "index.html"

with open('/mnt/user-data/outputs/venezuela-relief.html', 'rb') as f:
    content_b64 = base64.b64encode(f.read()).decode()

# Get current SHA
req = urllib.request.Request(f"https://api.github.com/repos/{REPO}/contents/{PATH}",
    headers={"Authorization": f"token {TOKEN}", "Accept": "application/vnd.github.v3+json"})
with urllib.request.urlopen(req) as r:
    sha = json.loads(r.read())['sha']

# Push
payload = json.dumps({"message": "commit message", "content": content_b64, "sha": sha}).encode()
req2 = urllib.request.Request(f"https://api.github.com/repos/{REPO}/contents/{PATH}",
    data=payload, method="PUT",
    headers={"Authorization": f"token {TOKEN}", "Accept": "application/vnd.github.v3+json", "Content-Type": "application/json"})
with urllib.request.urlopen(req2) as r:
    print("Pushed:", json.loads(r.read())['commit']['sha'][:7])
```

Cloudflare auto-deploys on push to main. Build time ~1-2 min.

---

## 12. Excel Backup

**File:** `ayudave_directory.xlsx`  
**Sheets:**
1. **Global Organizations** — 11 orgs with CN rating, ground status, gov independence, links
2. **By Country** — 20 country initiatives with Venezuela-specific, tax-deductible, ground status columns
3. **Donate Goods** — 6 rows covering Spain collection points + "none confirmed" for other EU
4. **Meta** — site info, counts, contact

Update the Excel whenever the website is updated. They should always be in sync.

---

## 13. Common Things That Break

| Problem | Cause | Fix |
|---------|-------|-----|
| Tab content bleeds into next tab | `</div>` closed a section early | Now uses `<section>` — shouldn't happen. Run validator. |
| Country filter does nothing | Filter buttons missing from `#cf` or `#scf` | Add button to both blocks |
| Menu tabs don't work | Escaped quotes in onclick attrs `\'` | Replace with plain single quotes |
| Bilingual swap broken | `<span>` or `"` inside data-en/data-es attr | Remove HTML from data attrs |
| Cloudflare build fails | `wrangler.json` missing or malformed | Check repo root has valid `wrangler.json` |
| Cards appear outside wrong panel | String replacement broke nesting | Run validator, check section boundaries |

---

## 14. Content Rules

- **No banking details** — no account numbers, no Bizum codes stored anywhere except as descriptions with proper context
- **No endorsement language** — no "we trust", no "verified" as a badge, no scores
- **Green pills** = confirmed by press/official source
- **Amber pills** = not confirmed or only partially confirmed
- **All collection points** must be verified through press sources before adding
- **No local Venezuelan organizations** — 2024 Anti-NGO Law makes verification impossible
- **Last updated date** must be updated on every publish (currently in Methodology tab and Donate Goods warning)

---

## Architecture update — June 28, 2026 (unified card system)

### Card system
All cards now use ONE template: `.ucard`. Previous templates (`.card`, `.ccard`, `.gcard`) are removed. Every card has:
- `data-scope` — `global` or `country`
- `data-country` — e.g. `spain`, `germany`, `france`, `uk` (lowercase, no spaces, no dashes)
- `data-contribution` — `cash`, `goods`, or `both`
- `data-tags` — for Global tab filtering (medical, relief, cash, logistics, children, faith)

### Three panels
- `<section data-panel="global">` — 12 global orgs
- `<section data-panel="country">` — all country cards including goods/both cards duplicated here from type panel
- `<section data-panel="type">` — all non-global cards filtered by cash/goods/both
- `<section data-panel="about">` — methodology

### Critical: goods cards appear in BOTH country and type panels
Physical collection point cards (goods/both) must exist in BOTH:
1. `#country-cards` (country panel) — inside the relevant country heading group
2. `#type-cards` (type panel) — flat list filtered by contribution type

If you add a new goods card, add it to both places.

### Filter systems
- `filter(btn, tag)` — filters `#global-cards .ucard` by `data-tags`
- `filterCountry(btn, code)` — shows/hides `.country-heading[data-country]` + nextElementSibling in `#country-cards`
- `filterType(btn, type)` — shows/hides `#type-cards .ucard` by `data-contribution`

### Country heading data-country
Each `<div class="country-heading t" data-country="X">` must have `data-country` matching the filter button value exactly. filterCountry uses `h.dataset.country` directly.

### Tab navigation
- Desktop sidebar: `#sf` (global filters), `#scf` (country filters), `#stf` (type filters) — shown/hidden by switchTab
- Mobile: `.mob-tabs` with `.mob-tab` buttons; `.mob-filters` with `.mob-fil` pill buttons inside each panel; `.country-filters` with `.fil` pill buttons in country panel
- All filter buttons are pill-shaped with `border-radius:20px` and wrap (no horizontal scroll)

### Validator
The validator at `/home/claude/validate.py` is outdated — it checks for old `.card` class and old panel names. It will report false positives. The real structural checks are: 4 sections present, JS functions present, no escaped quotes in onclick.
