# SEO Utils — Capability Digest

Running notes on SEO Utils features that materially affect ranking work, built from the
changelog at https://help.seoutils.app/changelog.

**Last reviewed:** 2026-11-13 — covered through **v2.15.0** (reviewed v2.6.0–v2.15.0).

Maintained by a scheduled cloud routine that re-reads the changelog every 3 days. When new
releases land, the routine appends what matters here and bumps the "Last reviewed" line
above. Read this before substantial SEO Utils work — it records capabilities that are easy
to miss and limitations that can be mistaken for real findings.

---

## Highest-value capabilities to actually use

### Keyword metrics source selector (v2.4.0, extended v2.5.0)
Six selectable DataForSEO sources: `labs` (default, cheapest, has gaps), `google_ads`
(live Google Ads), `labs_clickstream`, `labs_bing`, `dfs_search_volume`, `clickstream_bulk`.
Selectable per check and per GSC property; Keyword Explorer can now run live Google Ads
searches directly.

**Why it matters:** the default `labs` source has real coverage holes — it rejects
Canadian province-level locations outright ("DataForSEO Labs database does not support
Ontario,Canada"). When a location or keyword returns no data on `labs`, that is a source
limitation, NOT proof of zero volume. Re-check with `source: 'google_ads'` or
`dfs_search_volume` before concluding a keyword or market has no demand. `google_ads`,
`dfs_search_volume` and `clickstream_bulk` require Andy's own DataForSEO credentials.

### Internal Links, in Google Search Console (v2.5.0)
Identifies pages that need more internal links and suggests specific linking opportunities.

**Why it matters:** internal linking is the one authority lever fully under our control on
a small local site — no outreach, no link building, just structure. On 5–15 page local
service sites this is disproportionately effective. Run it once a site has GSC data.

### Search Intent Bulk Check (v2.5.0)
Labels keyword intent in bulk with **no DataForSEO credit consumption**.

**Why it matters:** free. Classify large keyword lists (informational vs commercial vs
transactional) before spending credits on volume checks, and use intent to decide which
keywords deserve a page, which belong in an FAQ, and which to skip.

### GMB Rank Tracker — Progress view (v2.3.0)
Compares grid rankings across date ranges.

**Why it matters:** local service businesses live or die in the map pack. This shows
whether grid coverage is actually expanding over time, not just today's snapshot.

### SERP Clustering — word-order twin reuse (v2.5.0)
Shares scraped SERP results between keywords differing only in word order, cutting cost.

**Why it matters:** clustering runs get cheaper, so clustering a wider keyword set is more
affordable. Local keyword sets are full of word-order variants ("pet waste removal keswick"
vs "keswick pet waste removal").

### GSC Rank Tracker (v2.10.0, built out through v2.14.2)
A new rank tracker built on Search Console data instead of DataForSEO scraping: daily
positions, Movers, Alerts, a Pages tab, a Before/after tab, CTR gap and Opportunity
columns, an Opportunity-scoring improvement (v2.14.2), and a comparison view against the
existing DataForSEO-based Organic Rank Tracker report.

**Why it matters:** this is free (GSC data, no DataForSEO credit) and gives a second,
independent read on rankings to cross-check the paid Organic Rank Tracker. The CTR gap /
Opportunity columns are a genuine opportunity-finder for a small site: they surface pages
that already rank but underperform on CTR, which on a 5–15 page local site is exactly the
kind of low-cost win (rewrite a title/meta) worth chasing before spending credits on new
keyword research. Not yet wired into the build process below — worth adding to STEP 15
ongoing work alongside GMB Progress and GSC Internal Links.

### Rank-result "poisoning" re-check (v2.12.0)
Automatically flags and re-checks rank results that look corrupted or personalized
(poisoned) instead of accepting them as-is.

**Why it matters:** rank position is the core number Andy watches per site. A single
poisoned scrape showing a fake drop (or fake gain) is indistinguishable from a real ranking
change unless the tool catches it — this closes that gap automatically.

---

## Where each of these is wired into the build

As of website-builder v1.5.0 these are no longer just "available" — they have a home in the
process. If a capability below is not listed here, it is still unused.

| Capability | Wired into |
|---|---|
| Keyword metrics `source` selector | niche-scout Step 2, STEP 3 step 4, STEP 15 ongoing work |
| Search Intent Bulk Check | STEP 3 step 3 (free triage before any paid check) |
| SERP Clustering `reuse_word_order_twins` | niche-scout Step 2, STEP 7 step 1 |
| Content Gap metrics auto-save | niche-scout Step 2 reports it; STEP 3 / 7 read `keyword_metrics` first |
| GMB Rank Tracker (grid scan) | niche-scout Step 1 fallback when Playwright is blocked |
| GMB Progress view | STEP 15 ongoing work, with a launch-day baseline scan |
| GSC Internal Links | STEP 15 ongoing work |
| `submit_url_for_google_indexing` / `check_google_indexing_status` | STEP 16 step 2 |
| `trigger_indexing_action` (inspection scan) | STEP 16 step 2 |
| IndexNow: `submit_url_to_index_now` / `check_index_now_status` | STEP 16 step 4 |

**Known live defect this closed:** niche-scout called `get_keyword_suggestions` and
`check_keyword_metrics` with no `source`, so every market validation silently ran on `labs`
— the one source documented to reject Ontario. Every site this skill builds is in Ontario.

**Historical defect, now fixed (v2.14.0):** the Organic Rank Tracker could mark a page that
was actually ranked as unranked. If any site's history shows a sudden "lost ranking" that
dropped to unranked before Oct 30, 2026 and recovered with no corresponding site change,
treat it as this bug, not a real ranking loss.

---

## Lower priority / situational

- **AI Visibility** (v2.8.0, agent actions v2.10.0, exports/client report v2.9.0) — tracks
  brand/business share of voice in AI chat answers (ChatGPT, Gemini, etc.) against named
  competitors. More useful for established brands people ask AI assistants about by name
  than for a brand-new rank-and-rent site with no name recognition yet; worth revisiting
  once a site has built up some direct-search volume.
- **Content Gap — Shared keywords mode** (v2.14.2) — shows keywords multiple competitors
  rank for in common. Useful for finding the "everyone in town has a page for this" keyword
  set a new local site is missing.
- **Internal Links (GSC) agent/MCP actions** (v2.7.0) — the existing Internal Links
  capability (already wired into STEP 15) can now be triggered via MCP action instead of
  only through the UI, which makes it automatable.
- **Scheduled LLM Rank Tracker credit-waste fix** (v2.14.1) — scheduled runs no longer
  repeat and burn credit when an engine returns no answer. Only matters if LLM Rank
  Tracker / AI Visibility scheduled checks are in use.
- **Organic Rank Tracker imports** (v2.5.0) — from True Ranker, SEO PowerSuite, Agency
  Analytics. Relevant when migrating an existing client off another tool.
- **Content Gap + Ads Vision auto-save keyword metrics** (v2.4.0) — metrics from these
  tools now persist to `keyword_metrics`, so they're queryable later via `query_database`.
- **GSC automatic checks spread over 30-day cycles** (v2.4.0) rather than batch processing.
  Explains why GSC-derived data trickles in rather than landing all at once.
- **Remote Access** (v2.2.0) + **custom domains** (v2.3.0, Agency plan) — browser access to
  SEO Utils from mobile; data stays local.
- **Per-report run logs** (v2.2.0) — "View Log" on long operations, auto-removed after 30
  days. Useful for diagnosing a stuck or failed run.
- **SERP Data Importer background mode** (v2.2.0).
- **Stability fixes** (v2.3.0) — Organic Rank Tracker no longer hangs when DataForSEO
  credentials are missing.
- **GA4 Traffic integration** (v2.15.0) — syncs Google Analytics 4 (all channels, not just
  organic) into Your Sites: sessions, key events, and revenue by channel/source/month vs. an
  earlier period. The GSC Rank Tracker now shows sessions and key events per keyword's page,
  and Organic/GSC Rank Tracker exports and published client reports get a Traffic tab.
  **Why it matters:** only worth turning on once a site has GA4 plus key-event tracking
  (call/lead-form tracking) actually configured — then it ties a specific ranked page to
  real conversions instead of just position, which is the real test of whether that page
  deserves more internal links or content work, and gives something concrete to show a
  renting client. Not useful on a brand-new site with no conversion tracking set up yet.
