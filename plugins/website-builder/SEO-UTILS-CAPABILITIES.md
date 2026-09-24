# SEO Utils — Capability Digest

Running notes on SEO Utils features that materially affect ranking work, built from the
changelog at https://help.seoutils.app/changelog.

**Last reviewed:** 2026-09-24 — covered through **v2.5.0** (reviewed v2.2.0–v2.5.0).

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

**Known live defect this closed:** niche-scout called `get_keyword_suggestions` and
`check_keyword_metrics` with no `source`, so every market validation silently ran on `labs`
— the one source documented to reject Ontario. Every site this skill builds is in Ontario.

---

## Lower priority / situational

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
