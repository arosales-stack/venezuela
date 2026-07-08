# 🚨 ABSOLUTE RULES — READ BEFORE ANYTHING ELSE

**Claude must NEVER:**
- Push to `main` without Audra explicitly saying "push to main" or equivalent
- Force-reset any branch
- Delete or recreate branches
- Push to any branch other than `dev` or `main`
- Touch `wrangler.json`, `robots.txt`, `sitemap.xml`, `CLAUDE.md` or any file other than `index.html` without explicit instruction
- Modify any Cloudflare or GitHub account configuration
- Make ANY change without being asked
- Make ANY change without reading the ENTIRE file first

**The only permitted operations without being asked:**
- Read files from the repo

**Push flow:**
1. Changes go to `dev` first — always
2. Audra reviews the dev preview
3. Only push to `main` when Audra explicitly says so

---

# Cloudflare Dev Preview

Every push to `dev` creates a new version with a unique hash URL:
`<hash>-europeforvenezuela.a-rosales.workers.dev`

To find it: Cloudflare dashboard → Workers & Pages → europeforvenezuela → Deployments → latest `dev` entry → three dots → Preview

Each push = new hash = new URL. Previous URLs still work (version-specific).

**wrangler.json has `preview_urls: true` and `workers_dev: true`** — required for preview URLs to generate.

Cloudflare auto-deploys on push to `main`. Build ~1-2 min.

---

# Site

**Live:** https://ayudave.net
**GitHub:** https://github.com/arosales-stack/venezuela
**GitHub user:** arosales-stack
**GitHub token:** (unchanged — see existing repo history, not repeated here to avoid GitHub secret-scanning push protection)
**Cloudflare project:** `europeforvenezuela` (Worker, NOT Pages)
**Contact:** suggestions@ayudave.net
**Last updated on site:** July 3, 2026
**Local file:** `/mnt/user-data/outputs/venezuela-relief.html`

Single HTML file. No framework. Google Fonts only external dependency.

---

# Architecture

## Four panels
```html
<section data-panel="global">   — 14 global orgs in #global-cards
<section data-panel="country">  — all country cards in #country-cards
<section data-panel="type">     — all non-global cards in #type-cards
<section data-panel="about">    — methodology
```

CSS: `[data-panel]{display:none}` / `[data-panel].active{display:block}`

## Unified card system
ALL cards use `.ucard`. Every card has:
- `data-scope` — `global` or `country`
- `data-country` — e.g. `spain`, `germany`, `uk` (lowercase)
- `data-contribution` — `cash`, `goods`, or `both`
- `data-tags` — for Global tab filtering: `medical`, `relief`, `cash`, `logistics`, `children`, `faith`, `animals`

**CRITICAL:** goods/both cards must exist in BOTH `#country-cards` AND `#type-cards`

## Card structure (always in this order)
```
ucard-meta:   [flag] Country  •  [type badge]
ucard-verify: [✓ Verified / ⚠ Partially verified — own div, computed, see below]
ucard-head:   Org name  [BADGE]
ucard-status: [status badge — own div, NOT inside ucard-head]
ubacking:     Backed by: ...
upills:       ✓ up-ok / ⚠ up-warn / dim up-dim
udetail:      description
ucard-footer: VISIT ↗
```

## Verification tag system (added July 3, 2026)
Every card carries a `.ucard-verify` div with a `.utag` span, computed mechanically — never hand-picked:

- **`.utag-verified`** (✓ Verified, green) — every pill on the card is `up-ok`
- **`.utag-partial`** (⚠ Partially verified, amber) — at least one pill flags a genuine delivery-risk concern

**What counts as a delivery-risk concern (downgrades to Partial):**
- No shipping partner / logistics chain disclosed (goods cards)
- Fund not earmarked specifically for Venezuela (general/global fund instead)
- Organizational independence disputed or partial (e.g. VRC governance dispute)
- On-ground presence not yet confirmed (pledged/mobilizing/appeal-launched, not confirmed)
- No independent charity rating available

**What does NOT count (logistics-only, does not affect the tag):**
- "Confirm hours before going"
- Item/scope restrictions (e.g. "Medical supplies only — no food or clothes")
- Transit time (e.g. "~2 month sea transit") — slow is not the same as unconfirmed

⚠️ CRITICAL RULE: Transit time NEVER downgrades a card from Verified to Partially verified. This has been corrected multiple times. A confirmed shipping chain (e.g. CMA CGM) with a 2-month transit is VERIFIED. Never add an amber pill for transit duration.

This distinction matters: the tag answers "will this actually reach people in Venezuela," not "how fast" or "how convenient." Getting this wrong (treating slow logistics as equivalent to unverified) was a bug fixed July 3, 2026 — see Known Bug History.

**Goods cards with no shipping partner disclosed** get an explicit pill: `⚠ No shipping partner disclosed` (amber) — silence is never treated as neutral.

**Closed initiatives are removed from the site entirely**, not left up with a "Closed" status badge. A dead collection point provides no value to a reader and clutters the directory. (Also added July 3, 2026 — 5 closed Spain goods cards removed: Sambil Madrid, Refugiados Sin Fronteras, VENESP Pardillo, AVA Alcobendas, AVeVa Valladolid.)

## Status CSS classes
- `.us-active` — green
- `.us-weekend` — amber
- `.us-pending` — RED (`color:#E05050;border-color:#7A2020;background:#2B0D0D`) — used for "Closed" (now only used transiently before removal, not left live)

## Country heading structure
```html
<div class="country-heading t" data-country="france" data-en="🇫🇷 France" data-es="🇫🇷 Francia">🇫🇷 France</div>
<div class="ucards">
  ... cards ...
</div>
```
`filterCountry` hides `.country-heading` + its `nextElementSibling` (the ucards div). Every country-heading MUST be followed immediately by a ucards div as a sibling — nothing between them.

## Filter functions
```javascript
filter(btn, tag)          // filters #global-cards .ucard by data-tags
filterCountry(btn, code)  // shows/hides country-heading + nextElementSibling in #country-cards
filterType(btn, type)     // shows/hides #type-cards .ucard by data-contribution + window.scrollTo(0,0)
```

---

# Countries (12)

| Country | Filter code | Cards |
|---------|-------------|-------|
| 🇪🇸 Spain | `spain` | 6 cash + 3 goods |
| 🇩🇪 Germany | `germany` | 5 |
| 🇳🇱 Netherlands | `netherlands` | 2 |
| 🇮🇹 Italy | `italy` | 1 |
| 🇵🇹 Portugal | `portugal` | 1 |
| 🇫🇷 France | `france` | 5 |
| 🇨🇿 Czech Republic | `czechrepublic` | 1 |
| 🇦🇹 Austria | `austria` | 2 |
| 🇨🇭 Switzerland | `switzerland` | 2 |
| 🇱🇺 Luxembourg | `luxembourg` | 1 |
| 🇳🇴 Norway | `norway` | 1 |
| 🇬🇧 UK | `uk` | 4 |

---

# Current Content

## Global (14 orgs)
Direct Relief, MSF, WCK, Global Empowerment Mission, Catholic Relief Services, IRC (teams on ground Caracas+La Guaira, 75k+ people reached 2025 — ✓ Verified), Save the Children, GiveDirectly (team physically in Venezuela, first payments coming — ✓ Verified), Airlink, People in Need, UN Venezuela Humanitarian Fund, GlobalGiving, IFAW (animals tag), IFRC Emergency Appeal (VRC disclaimer pill)

## Spain — Cash (7)
Cruz Roja Española, Comité de Emergencia Español, MSF España, WCK Spain (Bizum 03843), BUSF Córdoba (Bizum 38350), Save the Children España (Bizum 13132), **World Vision España** (hygiene kits, food baskets, child protection in Caracas, church + UCV partners)

## Spain — Goods (3, active only — 5 closed initiatives removed July 3, 2026)
- Pabellón Manuel Cadenas — Leganés → Active (Acción por Venezuela, 45+ points, Toneladas→Bandazul) — ✓ Verified
- Fundación Madrina — Madrid → Active (Diócesis de Caracas, Bizum 00909) — ✓ Verified
- AMASVE Alicante → Active (medical only) — ⚠ Partially verified (no shipping partner disclosed)

Removed (closed, no longer listed): Sambil Madrid, Refugiados Sin Fronteras + Diáspora, VENESP Pardillo, AVA Alcobendas, AVeVa Valladolid.

## Germany (5)
Aktion Deutschland Hilft, Aktionsbündnis Katastrophenhilfe, Caritas international, DRK (VRC pill), Malteser International

## Netherlands (2)
Rode Kruis Nederland (VRC pill), Cordaid

## Italy (1)
Croce Rossa Italiana (VRC pill)

## Portugal (1)
Cruz Vermelha Portuguesa (VRC pill)

## France (5)
- MSF France (amber — general page)
- Secours Catholique (amber)
- Mujeres Unidas por la Libertad y la Paz (goods — Alfortville depot, CMA CGM confirmed, French Foreign Ministry, ~2 month transit) — ✓ Verified (transit time NEVER downgrades — confirmed CMA CGM chain)
- Ville de Nancy — Hôtel de Ville (goods — Active Jul 2–10, 10h–17h, Salle Mienville, "Todos con Venezuela", CMA CGM, Les Bruits du Cœur 54) — ✓ Verified

## Czech Republic (1)
People in Need

## Austria (2)
Caritas Österreich, Österreichisches Rotes Kreuz (VRC pill)

## Switzerland (2)
Caritas Schweiz, SRK (VRC pill, REDOG)

## Luxembourg (1)
MSF Luxembourg — ⚠ Partially verified (no Venezuela-specific page, general MSF fund)

## France — removed
VTN (Venezuela Te Necesita) removed July 6 — no confirmed earthquake-specific campaign, no confirmed logistics chain, leads to nothing actionable. — new-format card (warning pill first, confirmation pill second, plain-language detail explaining money goes to global emergency fund)

## Norway (1)
Caritas Norge (Vipps #91895)

## UK (4)
British Red Cross (VRC pill), UNICEF UK (20 tonnes landed Valencia June 28, $52M needed), DEC — Venezuela Earthquake Appeal (Jul 1, govt match £2M), Network for Animals / NFA (charity 1142700, funds RAC)

---

# Red Cross Independence Disclaimer
All 9 Red Cross cards carry amber pill: **⚠ VRC independence disputed (2023 govt intervention) — IFRC monitors fund accountability**

Context: Maduro's TSJ forcibly removed VRC president Aug 2023. IACHR condemned it. New elected board (Luis Farías) installed June 2024 following IFRC-accompanied assembly. VRC actively responding to earthquake. The 2023 concern is real but the situation has evolved.

This is treated as a genuine delivery-risk concern (see Verification tag system above) — it correctly keeps all 9 Red Cross cards at "Partially verified," not "Verified."

---

# Card Format — Card Format (fully implemented July 3, 2026)

Every card, global and country alike, now carries the same fixed-slot skeleton described in "Architecture" above (ucard-meta → ucard-verify → ucard-head → upills → udetail). Luxembourg was the original test card for the reordered-pills convention (warning first, confirmation second); the verification-tag layer was added on top of that for every card site-wide.

Tax-deductibility and Gift Aid facts were moved out of the trust-pill row into the description text — they're convenience facts, not trust signals, and mixing them in made the trust row noisy.

---

# Known Bug History

| Bug | Cause | Fix |
|-----|-------|-----|
| UK appeared on all country filters | Extra `</div>` after Norway orphaned UK outside `#country-cards` | Removed extra div |
| France cards appeared on all country filters | Extra `</div>` after Nancy card orphaned Nancy outside ucards | Removed extra div |
| NFA appeared after France section | Extra `</div>` closed UK ucards before NFA | Removed extra div |
| Desktop type filter showed cards off-screen | Page already scrolled | Added `window.scrollTo(0,0)` to `filterType` |
| Closed badge invisible | `.us-pending` was grey-on-grey | Changed to red `#E05050` |
| Status badge crammed in title row | `ustatus` inside `ucard-head` | Moved to own `div.ucard-status` |
| Dev branch diverged, Cloudflare stopped building | Claude force-reset dev branch — NEVER DO THIS | Reset dev to main + pushed clean file |
| Cloudflare didn't build dev pushes for ~1 hour | Cloudflare Workers Builds incident July 2 22:00-23:39 UTC | Resolved by Cloudflare |
| Verification tags lost (July 6) | Claude pushed local file to main without verification tags from July 3 | Restored from bc8e0ee + applied July 6 content on top |
| Mujeres Unidas wrongly marked Partial | Transit time pill added back despite rule — transit duration is NEVER a delivery-risk concern | Removed transit pill, set to Verified |
| Verification tag penalized slow-but-confirmed logistics same as truly unverified cards | Tag rubric originally flagged ANY amber pill, including pure timing/logistics notes | Rubric now only downgrades for genuine delivery-risk factors (see Verification tag system); transit time, hours, item restrictions excluded |
| Closed initiatives stayed live with a red "Closed" badge indefinitely | No removal policy existed | Closed initiatives are now removed from the site entirely once confirmed closed |

---

# Push Workflow

```python
import base64, json, urllib.request
TOKEN = "<see existing CLAUDE.md commit history — omitted here to avoid GitHub push protection>"
REPO  = "arosales-stack/venezuela"
PATH  = "index.html"
BRANCH = "dev"  # NEVER "main" without explicit consent

with open('/mnt/user-data/outputs/venezuela-relief.html', 'rb') as f:
    content_b64 = base64.b64encode(f.read()).decode()
req = urllib.request.Request(f"https://api.github.com/repos/{REPO}/contents/{PATH}?ref={BRANCH}",
    headers={"Authorization": f"token {TOKEN}", "Accept": "application/vnd.github.v3+json"})
with urllib.request.urlopen(req) as r:
    sha = json.loads(r.read())['sha']
payload = json.dumps({"message": "commit msg", "content": content_b64, "sha": sha, "branch": BRANCH}).encode()
req2 = urllib.request.Request(f"https://api.github.com/repos/{REPO}/contents/{PATH}",
    data=payload, method="PUT",
    headers={"Authorization": f"token {TOKEN}", "Accept": "application/vnd.github.v3+json", "Content-Type": "application/json"})
with urllib.request.urlopen(req2) as r:
    print("Pushed:", json.loads(r.read())['commit']['sha'][:7])
```

---

# Bilingual System

Every text element: `class="t"` + `data-en` + `data-es` attributes.
`setLang(l)` swaps innerHTML of all `.t` elements. Default: EN.
No single quotes inside data attrs. No `<span>` inside data attrs.

---

# Colors

```css
--bg:#0D1117  --surface:#161B22  --surface2:#1E242D  --border:#2A2D35
--amber:#E8A020  --amber-dim:#7A5310  --text:#F0EEE9  --muted:#B0B4BF
--dim:#7A7E8A  --green:#4CAF50  --sidebar:240px
```

Light mode: `[data-theme="light"]` toggle. Persists via `localStorage` key `ayudave-theme`.

Verification tag colors reuse the same green/amber tokens as pills (`.utag-verified` = green, `.utag-partial` = amber), including light-mode overrides.

---

# SEO
- `sitemap.xml` — changefreq daily (⚠ lastmod currently stale at 2026-06-28 — needs explicit instruction to fix, not touched automatically)
- `robots.txt` — allows GPTBot, ClaudeBot, Google-Extended, CCBot, anthropic-ai
- JSON-LD WebPage schema, dateModified 2026-07-03

---

# Content Rules
- No relative language ("this weekend") — use exact dates/times
- No banking details as actionable data
- No local Venezuelan organizations (2024 Anti-NGO Law)
- Green pills = confirmed. Amber = unconfirmed/partial.
- Verification tag (✓ Verified / ⚠ Partially verified) reflects delivery-risk only — never logistics speed/convenience (see Verification tag system)
- Closed initiatives are removed from the site, not left marked "Closed"
- Last updated date must be updated on every publish
- Footer: "All entries verified manually from press sources and official organization pages. No affiliation with any listed initiative. No compensation received."

---

# Methodology panel text (must stay in sync with index.html About section)

**What counts as official** — unchanged: registered org with public track record, verified in press by named journalist/publication, or named contact/founder with verifiable logistics chain.

**Verification tag explanation (current, concise, added July 3, 2026):**
"Each card carries a ✓ Verified or ⚠ Partially verified tag based on one question: will the money or goods actually reach people in Venezuela? That covers things like a named shipping partner, funds earmarked specifically for Venezuela, and organizational independence — not pure logistics like opening hours or transit time, which are noted separately but don't affect the tag. Closed initiatives are removed, not left up with a stale status."

**How we find and verify** — unchanged: AI agents search continuously, every entry manually verified by a human before publication, physical collection points change rapidly.

**What we don't cover** — unchanged: local Venezuelan organizations (2024 Anti-NGO Law).
