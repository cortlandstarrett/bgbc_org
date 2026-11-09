---
cursor:
  subagentId: "bc-8f905af8-9fe8-5cc7-bbeb-6efc664089b7"
---

# BGBC WordPress → Jekyll / GitHub Pages Migration Analysis

**Church:** Battle Ground Bible Church (bgbc.org)  
**Heritage:** https://bgbc.org (WordPress.com Atomic)  
**Target repo:** https://github.com/cortlandstarrett/bgbc_org  
**Design note:** `/workspace/doc/bgbc_org.adoc` (also `doc/bgbc_org.adoc` in repo)  
**Scope:** Design-note §4.1 Analysis of Existing Artifacts and §4.2 Recommendations  
**Inspected:** 2026-11-09

---

## 1. Executive summary

The heritage site is a small, parishioner-facing WordPress.com church site whose real content is (1) static identity/about pages, (2) a deep sermon audio archive organized by month and by series, and (3) ministry/schedule information. The target GitHub repo today is essentially empty (license, README, design note only)—no Jekyll, no Actions, no content.

**Jekyll + GitHub Pages remains the right hosting/stack choice** for Markdown-first maintenance. The main risk is **sermon audio volume**: ~455 audio files, with recent samples ~27–44 MB each, implying on the order of **~10–15+ GB** if the full archive is kept. That will not fit cleanly as ordinary git blobs on GitHub Pages. Audio must be stored by date in a maintainer-friendly layout, but **served from a strategy that does not explode the Pages repo** (see §4.3).

---

## 2. §4.1 Findings — heritage site (bgbc.org)

### 2.1 Platform and branding

| Item | Finding |
|------|---------|
| Host | WordPress.com Atomic (`data-provider="wordpress.com" data-service="atomic"`) |
| Theme | **Radcliffe 2**, style pack **vintage** |
| Fonts | Theme ships Libre Baskerville; Jetpack Custom Fonts override body/UI to **Open Sans** |
| Logo | Pillar-in-circle mark: `wp-content/uploads/2018/10/cropped-pillar-circle-white-backgorund-512px.jpg` (+ transparent PNG variants) |
| Tagline | “a passion for God and compassion for people” |
| Contact | 2430 E 600 North, W Lafayette, IN 47906 · (765) 567-2844 · battlegroundbible@gmail.com |
| Social | Facebook group, Instagram; FB LiveStream linked from nav |
| Hero | Theme hero-area background: building exterior photo `…/2026/07/website-front-page-photo-spring-fall-2026-ariel-28.jpeg` (~1 MB) |
| Homepage body | Page slug `home-page` content is effectively empty (`…`); home is theme chrome + hero + menus |

**Live color cues (customizer + vintage pack):**

- Page/chrome background often **`#cccccc`** (custom background)
- Vintage pack defaults: parchment **`#eae8dc`**, accent blue **`#2b6e9d`**, border taupe **`#c7c4b4`**, text **`#222` / `#342707`**
- Custom link/nav greys around **`#4f4f4f` / `#707070`**

Net look today: utilitarian church site with vintage Radcliffe styling, grey custom colors, Open Sans, pillar logo, and a large exterior hero photo—not a marketing microsite.

### 2.2 Information architecture / navigation

Primary nav (flattened):

1. **Home**
2. **Who We Are** → Our Doctrine (`/what-we-believe/`), Our Values (`/core-values/`), Our Vision (`/our-vision/`)
3. **Our Staff** → Senior Pastor & Elders (`/staff/`), Deacons, Trustees, Support Staff (`/paid-positions/`)
4. **Sermons**
   - **By Date** → year → month pages (2018-07 through 2026-11; ~99–101 month pages; a few broken/odd slugs: `/temp`, `/august-2109`, `/2793-2`, `/may-2025-2`, mislinked Oct 2023→`/october-2024/`)
   - **By Series** → ~15 topical series index pages (e.g. Biblical Marriage, Ephesians, John, Psalm 119, …)
5. **Worship Playlist** (external YouTube)
6. **Current song lyrics PDF** (rotating weekly upload under `wp-content/uploads/YYYY/MM/`)
7. **Ministries** (`/ministries-at-bgbc/`)
8. **Core Class** → Google Sheet schedule + lesson pages (Bible Storyline, Fundamentals of the Faith, Spiritual Disciplines)
9. **Resources** → mostly external links + Nicene Creed page
10. **FB LiveStream**

Footer repeats address / phone / email.

### 2.3 Content inventory (REST + sitemap)

| Type | Approx. count | Notes |
|------|---------------|-------|
| Pages | **133** | Sitemap lists ~134 URLs |
| Month sermon pages | **~101** | Dominant content type |
| Beliefs / vision / values / creed | **4+** | Doctrine is long (~9.6k HTML chars, 16 articles) |
| Staff / officers / ministries | **5** | Staff bios + photos |
| Series / lesson indexes | **~20** | Overlap with sermon audio |
| Blog posts | **1** | Legacy “Current Sunday Morning Series” (2018) — ignore or fold into home |
| Audio media | **455** | Mostly `audio/mpeg` (.mp3) and some .m4a |
| Images | **39** | Logo variants, staff photos, hero, a few others |
| Application (PDF etc.) | **7** | Song sheets / handouts |
| Video | **1** | One `.mov` (Dec 2020) in video sitemap |

**Sermon page pattern** (month and series pages alike): unordered list of dated entries with title, Scripture, series name, speaker, inline audio URL, and “Click to Download”. Native HTML5 `<audio>` appears on recent pages. Filenames often follow `YYYYMMDD-SundayAM.mp3` (or `.m4a`), which is already a strong dating convention.

**Sample sizes (HEAD):**

- `20261101-SundayAM.mp3` → **~44.1 MB**
- `20260809-SundayAM.m4a` → **~27.1 MB**

At hundreds of files in that range, a full mirror is multi-gigabyte.

### 2.4 Plugins / WordPress infrastructure to eliminate

Observed dependencies (drop or replace):

| Dependency | Role today | Migration stance |
|------------|------------|------------------|
| Jetpack (stats, likes, Sharedaddy, social menu, Facebook embed/likebox, sitemaps, Photon/`i0.wp.com`) | Analytics, sharing, CDN images, social | **Drop.** Optional: plain social links; privacy-friendly analytics only if desired |
| Radcliffe 2 + vintage style pack | Layout/look | **Replace** with minimal custom theme or simple Jekyll theme |
| Jetpack Custom Fonts / WP.com font API | Open Sans | Self-host or use system/Google Fonts link in Jekyll |
| Gutenberg / block library CSS | Residual | Unused in static site |
| Comments / likes | Present in theme chrome; largely irrelevant | **Eliminate** |
| Media library + month upload folders | Audio/PDF hosting | Replace with dated static media strategy (§4.3) |
| Google Sheets embed (Core Class schedule) | Live schedule | Keep as external link **or** convert to a Markdown table maintainers edit in-repo |
| External links (YouTube playlist, Bible plans, IFCA, Study Bible) | Resources | Keep as Markdown links |
| WordPress menus / pages | IA | Recreate as Jekyll `collections` + `_data/navigation.yml` |

No contact-form plugin surface was central; contact is mailto/phone. No e-commerce.

### 2.5 What parishioners actually use

Priority order for a clean rebuild:

1. Find this week’s / recent sermon audio and download it  
2. Service times & ministries (Sunday morning + midweek)  
3. Who we are / doctrine / staff  
4. Series archives and older months  
5. Lyrics PDF / worship playlist / livestream / resources  

---

## 3. §4.1 Findings — target repository

**Path:** `/agent/repos/bgbc_org` · remote `github.com/cortlandstarrett/bgbc_org` · branch `main`

| Present | Absent |
|---------|--------|
| `README.md` (“website source”) | `_config.yml`, Gemfile, layouts, includes |
| `LICENSE` (CC0) | Any Markdown content pages |
| `doc/bgbc_org.adoc` (design note; matches `/workspace/doc/bgbc_org.adoc`) | `_posts` / collections / sermon data |
| | `assets/` media |
| | `.github/workflows/` (no Actions yet) |
| | GitHub Pages config artifacts |

**Gap vs design-note requirements:** essentially 100% of §3 and §4.3–4.4 still to do. Clean-start is appropriate; nothing to preserve except the design note and license.

**Note:** Design note §4.3 text says `cortlandstarrett/bggc_org` (typo); correct repo is `bgbc_org`.

---

## 4. §4.2 Recommendations

### 4.1 Layout (IA for Jekyll)

Keep the mental model parishioners already know; simplify the WP mega-menu.

**Suggested top-level nav (6 items):**

1. Home  
2. About (Doctrine, Values, Vision — section or subpages)  
3. Staff  
4. Ministries  
5. Sermons (landing: latest + browse by date / by series)  
6. Resources (playlist, plans, creed, livestream, contact)

**Proposed content model (Markdown-first):**

```text
/
  index.md                          # home: brand, times, CTA to latest sermon + ministries
  about/
    doctrine.md
    values.md
    vision.md
  staff.md                          # or staff/_*.md if preferred
  ministries.md
  resources.md
  sermons/
    index.md                        # latest + year index
    series/
      biblical-marriage.md          # lists sermons in series (generated or hand-linked)
      ...
_sermons/                           # Jekyll collection (one file per sermon)
  2026-11-08-plan-for-sorrowful-heart.md
  ...
_data/
  navigation.yml
  site.yml                          # address, phone, email, social URLs, service times
assets/
  images/                           # logo, hero, staff (optimized)
  docs/                             # lyrics PDFs, handouts
  # audio: see §4.3 — do not dump 10GB+ into git unchecked
```

**Sermon front matter example:**

```yaml
---
title: "A Plan for Dealing with a Sorrowful Heart"
date: 2026-11-08
speaker: Pastor Kenneth Loehe
scripture: Psalm 119:25-32
series: psalm-119
audio: 2026/11/20261108-SundayAM.mp3   # path relative to audio root
audio_format: mp3
---
```

Month and series index pages can be **generated** from the collection (Liquid/`group_by`) so maintainers only add one Markdown file + one audio file per Sunday—meeting §3.3 and §3.6 better than hand-maintaining 100 month pages.

### 4.2 Color scheme and look & feel

**Goal (design note §3.1):** clean, utilitarian, parishioner-first—ease of use over ornament.

**Recommend a restrained “church utilitarian” system**, echoing heritage brand tokens without cloning Radcliffe vintage chrome:

| Token | Suggested value | Role |
|-------|-----------------|------|
| `--bg` | `#f4f4f2` soft warm gray (not parchment card chrome) | Page background |
| `--surface` | `#ffffff` | Content column |
| `--text` | `#1f2428` | Body |
| `--muted` | `#5c6570` | Secondary |
| `--accent` | `#2b6e9d` (heritage vintage blue) | Links, focus, small rules |
| `--accent-dark` | `#1f5275` | Hover / header bar |
| `--border` | `#d5d8dc` | Hairlines only where needed |
| Brand mark | Existing pillar logo (migrate PNG/JPG; prefer transparent PNG) | Header identity |

**Typography:** Prefer readable utilitarian pairing—e.g. **Source Serif 4** (or Libre Baskerville if continuity is prized) for page titles only; **Source Sans 3** for UI/body. Avoid Inter/Roboto defaults; avoid decorative “AI brochure” looks.

**Layout principles:**

- Single content column (~40–42 rem), sticky compact header with logo + name + simple nav  
- Home: logo/name as primary brand signal, one short welcome line, service times, one CTA to latest sermon, one exterior photo (migrated hero)—no card grids, no stat strips  
- Sermon lists: date · title · scripture · speaker · native `<audio controls>` + download link  
- Staff: simple photo + bio blocks, not cards-with-shadows  
- Mobile: hamburger or wrapping text nav; large tap targets on audio controls  

**Theme choice:** A minimal custom theme (few layouts + one CSS file) is preferable to a heavy third-party Jekyll theme. If a theme is used for bootstrap, use something sparse (e.g. minima as a starting scaffold) and immediately restyle to the tokens above.

### 4.3 Meeting each design-note requirement

#### Markdown-first / simplified maintenance (§3.3)

- All pages and sermons as Markdown + YAML front matter.  
- Shared strings (address, phone, times, social) in `_data/`.  
- No page builders; no HTML except rare tables or embeds.  
- Document a one-page `CONTRIBUTING.md`: “add sermon = drop audio + copy Markdown template.”

#### GitHub Actions auto-build (§3.3, §3.4)

- Use official **GitHub Pages + Actions** flow: workflow `jekyll-gh-pages` (or `actions/jekyll-build-pages`) on push to `main`.  
- Set repo Pages source to **GitHub Actions**, not “Deploy from branch” legacy.  
- Keep `Gemfile` pinned to `github-pages` gem **or** a small modern Jekyll 4 Gemfile consistent with the Actions builder—pick one path and stick to it.  
- Soft rule from §4.4: every commit on `main` should leave Actions green; develop on feature branches / PRs.

#### Migrate media; drop heritage references (§3.4)

- Copy logo, hero, staff photos, PDFs into `assets/` with stable paths.  
- Rewrite all `bgbc.org/wp-content/...` and `i0.wp.com` URLs to local assets.  
- After cutover, DNS for `bgbc.org` → GitHub Pages; then decommission WordPress.  
- Keep a temporary redirect map for high-traffic month/series slugs if possible (Cloudflare or Pages redirects) so old bookmarks still work.

#### Eliminate WordPress infrastructure (§3.5)

- Drop Jetpack entirely (likes, stats, Photon, Sharedaddy, FB likebox).  
- Replace menus with data-driven nav.  
- Replace WP audio shortcodes with HTML5 audio.  
- Comments/likes: omit.  
- Core Class Google Sheet: keep external link initially (lowest friction); optionally later a Markdown schedule.  
- One legacy `.mov`: link or omit unless still needed.

#### Dated sermon audio (§3.6) — **critical**

**Maintainer layout (easy upload / locate / replace):**

```text
audio/
  2026/
    11/
      20261101-SundayAM.mp3
      20261108-SundayAM.mp3
    10/
      ...
```

Mirror the existing `YYYYMMDD-SundayAM` naming. Replacement = overwrite same filename and push (or re-upload to object store with same key).

**Hosting strategy (choose explicitly before bulk migrate):**

| Option | Pros | Cons | Fit |
|--------|------|------|-----|
| **A. Object storage + CDN** (Cloudflare R2, S3, Backblaze B2) with `audio/` key prefix | Scales to 10–15 GB; cheap; easy replace-by-key; Jekyll only stores Markdown pointers | Extra account/config | **Recommended for full archive** |
| **B. GitHub Releases assets** (tag per year/month) | Stays in GitHub ecosystem; no LFS Pages issues | Awkward UX for “replace one file”; scripting needed | Acceptable hybrid |
| **C. Git LFS in repo** | Familiar git workflow | GitHub Pages **does not serve LFS pointers as binaries** without extra build steps; still large | Weak for Pages |
| **D. Commit audio as normal git files** | Simple for tiny sets | **~10–15 GB**, exceeds practical Pages/repo soft limits; slow clones | **Reject for full archive** |

**Pragmatic path:**

1. Start Jekyll with **recent N months** of audio only (or even remote URLs still pointing at WP during dual-run).  
2. Provision R2/B2 (or similar) bucket `bgbc-audio` with public read; keys = `2026/11/20261108-SundayAM.mp3`.  
3. Set `audio_base_url` in `_config.yml`.  
4. Bulk-download from WP media library (REST `media_type=audio`) into the dated tree; upload to bucket; generate `_sermons/*.md` from month-page HTML or REST.  
5. After DNS cutover, delete WP.

Optional: transcode future uploads to a consistent moderate bitrate MP3 (e.g. 64–96 kbps mono for speech) to cut size ~2–4×—policy decision for leadership, not required for launch.

---

## 5. Proposed incremental migration plan

Aligned with design-note §4.3 Clean Start and §4.4 Incremental Update.

### Phase 0 — Decisions (short)

1. Confirm **audio hosting option A** (object storage) vs hybrid B.  
2. Confirm custom domain will remain `bgbc.org`.  
3. Confirm which odd month slugs/broken menu links to fix vs skip.  
4. Confirm whether historical `.m4a` is kept as-is or normalized to `.mp3`.

### Phase 1 — Clean Jekyll skeleton (repo green)

1. Initialize Jekyll site in `cortlandstarrett/bgbc_org` (layouts: default, page, sermon; includes: header, footer, audio-player).  
2. Add `_config.yml`, Gemfile, `.gitignore`.  
3. Add GitHub Actions Pages workflow; enable Pages → Actions.  
4. Publish stub Home with brand, tagline, contact from `_data/site.yml`.  
5. Commit design note under `doc/` unchanged.  
**Exit:** `https://cortlandstarrett.github.io/bgbc_org/` (or custom domain later) builds on every `main` push.

### Phase 2 — Static identity pages

1. Migrate Doctrine, Values, Vision, Staff (+ photos), Deacons, Trustees, Support Staff, Ministries, Nicene Creed, Resources.  
2. Migrate logo + hero; apply utilitarian CSS tokens.  
3. Wire final nav.  
**Exit:** About/Staff/Ministries usable; no WP chrome.

### Phase 3 — Sermon content model (without full audio bulk)

1. Add `_sermons` collection + sermon layout with HTML5 player.  
2. Manually migrate **latest 1–2 months** + current series pages as proof.  
3. Generate year/month and series indexes from collection.  
4. Document maintainer workflow.  
**Exit:** Sermons IA works; uploading a new Sunday is a known recipe.

### Phase 4 — Media pipeline

1. Stand up audio storage; set `audio_base_url`.  
2. Scripted export of WP audio → dated tree → upload.  
3. Scripted scrape/convert of month + series pages → `_sermons/*.md` (review diffs).  
4. Migrate PDFs (lyrics/handouts) into `assets/docs/`.  
5. Fix/skip known bad slugs.  
**Exit:** Archive browsable; no `wp-content` URLs in Markdown.

### Phase 5 — Cutover

1. Add `CNAME` / DNS for `bgbc.org` → GitHub Pages; enforce HTTPS.  
2. Optional redirects for old paths.  
3. Parallel-run briefly; verify audio playback on mobile.  
4. Remove WordPress.com site; cancel WP plan.  
5. Final pass: search repo for `wp-content`, `wp.com`, `i0.wp.com`, Jetpack leftovers.

### Phase 6 — Hardening

1. `CONTRIBUTING.md` + sermon template.  
2. Optional bitrate policy for new recordings.  
3. 404 page; basic SEO titles/descriptions.  
4. Keep Actions green as the quality bar for every incremental PR (§4.4).

---

## 6. Risks and open questions

| Risk | Mitigation |
|------|------------|
| Audio size breaks GitHub | Do not commit full archive as blobs; use object storage (or Releases) |
| Menu/slug debt (typos, `/temp`) | Clean during generation; don’t reproduce broken IA |
| Dual audio formats (.mp3/.m4a) | Support both in player; prefer `.mp3` going forward |
| Rate limits on WP REST during export | Throttle/backoff scripts; prefer sitemap + month-page scrape |
| Custom domain / DNS ownership | Confirm who controls bgbc.org DNS before cutover |
| Google Sheet dependency | Acceptable external link; not WP infra |
| Homepage currently empty of body copy | Rewrite a short parishioner-focused home in Markdown |

---

## 7. Recommendation summary (top actions)

1. **Proceed with Jekyll + GitHub Actions Pages** in this empty repo—clean start.  
2. **Ship a utilitarian custom look** using heritage blue `#2b6e9d` + pillar logo; drop Radcliffe/Jetpack aesthetics.  
3. **Model sermons as a Jekyll collection** with dated filenames; generate month/series indexes—don’t hand-maintain 100 month pages.  
4. **Store audio in `YYYY/MM/YYYYMMDD-…` keys on object storage (preferred)**, referenced from Markdown—not as a multi-GB git tree.  
5. **Migrate incrementally:** skeleton → identity pages → recent sermons → bulk audio/archive → DNS cutover → decommission WordPress.

---

## 8. Sources inspected

- Design note: `/workspace/doc/bgbc_org.adoc`, `/agent/repos/bgbc_org/doc/bgbc_org.adoc`  
- Live site HTML/CSS: homepage, `/november-2026/`, `/staff/`, Radcliffe `vintage.css`  
- WordPress REST: `/wp-json/wp/v2/{pages,posts,media,categories}`  
- Sitemaps: `/sitemap.xml`, `/sitemap-1.xml`, image & video sitemaps  
- Target git tree on `main` (LICENSE, README, design note only)
