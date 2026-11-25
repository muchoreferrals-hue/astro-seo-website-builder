# SEO Utils — Capability Digest

Running notes on SEO Utils features that materially affect ranking work, built from the
changelog at https://help.seoutils.app/changelog.

**Last reviewed:** 2026-11-25 — covered through **v2.18.0** (reviewed v2.6.0–v2.18.0).

Maintained by a scheduled cloud routine that re-reads the changelog every 3 days. When new
releases land, the routine appends what matters here and bumps the "Last reviewed" line
above. Read this before substantial SEO Utils work — it records capabilities that are easy
to miss and limitations that can be mistaken for real findings.

---

## Surface globally

**Always pass an explicit country/location code on SEO Utils keyword and SERP lookups
(MCP actions or UI).** A bug fixed in v2.18.0 meant a keyword/SERP lookup whose location
code was sent as a bare number, or with no country specified at all, silently ran against
the United States instead of erroring. A Canadian (Ontario) lookup made that way could have
been stored as US data with nothing to flag it as wrong — the same "looks like a real
result but isn't" trap as the existing `labs` source rejecting Ontario. It's fixed now
(malformed calls are refused outright rather than silently defaulting), but since every site
this skill builds targets Ontario, always specify the country/location explicitly on every
keyword or SERP call rather than relying on a default. If any keyword or SERP data pulled
before Nov 24, 2026 looks off for a Canadian market, re-check it — it may be mislabeled US
data from this bug.

---

## Highest-value capabilities to actually use

### GMB Rank Tracker — Google Business Profile performance data & alerts (v2.18.0)
Connect a Google account and GMB Rank Tracker shows live Business Profile results next to
grid ranks: calls, website clicks, direction requests, profile views, a Performance tab, an
Actions (30 days) column, how many searches surfaced the profile, reviews, and profile-edit
history — a profile can be tracked for this data without running rank tracking at all. New
alerts (falling actions, a low-star review, a profile change) can email or webhook through a
new automation trigger. GMB PDF/HTML reports can include this data, with a results-only
PDF/CSV/Excel export and a toggle for whether client-published reports show it. All of it is
also readable through MCP/AI-app actions.

**Why it matters:** this is the first time GMB Rank Tracker ties grid/Local-Pack position to
the thing that actually pays the bills — calls, website clicks, and direction requests — in
one place, and it costs nothing beyond the existing Google account connection (no DataForSEO
credit). On a rank-and-rent site, "we hold position 2 in the pack" is a weak pitch; "we hold
position 2 and it generated 40 calls this month" is the pitch, and the client-report toggle
makes that easy to hand to a renting client. The alerts also catch a bad review or an
unauthorized profile edit — a known way a competitor or a Google-suggested edit quietly
costs a month of calls — before it shows up as an unexplained rank or traffic drop.

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

### GMB Rank Tracker — Local Pack and Local Finder results (v2.16.0)
GMB Rank Tracker now tracks three distinct Google local result types, not just Maps: a list
switcher moves the whole report — history, Compare, report groups, exports — between Google
Maps, the Local Pack, and the Local Finder.

**Why it matters:** "map-pack ranking" for a local service search usually means the 3-pack
shown on a plain "service + town" search (Local Pack), which is a different, usually harder
result to hold than general Google Maps placement. Until now GMB Rank Tracker could only
confirm Maps position. Check Local Pack position specifically on every GMB Progress check
going forward — a site can look strong on Maps while still missing the 3-pack that actually
drives local calls.

### Rank-result "poisoning" re-check (v2.12.0, opt-in verification extended v2.17.0, pricing change v2.18.0)
Automatically flags and re-checks rank results that look corrupted or personalized
(poisoned) instead of accepting them as-is. v2.17.0 adds an opt-in, paid extension of this:
the Organic Rank Tracker can check each fetched results page against Google's own Web
results and replace organics that look poisoned (off by default, costs one extra results
page per keyword), and Check SERP / SERP Similarity / the keyword SERP card can do the same
live-lookup verification for $0.002 + ~20 extra seconds per lookup. GSC Rank Tracker's own
"Check live SERP" now runs this same live verification too.

**Why it matters:** rank position is the core number Andy watches per site. A single
poisoned scrape showing a fake drop (or fake gain) is indistinguishable from a real ranking
change unless the tool catches it — this closes that gap automatically for free by default.
Turn on the paid verification only when a specific result looks like a fluke worth
double-checking (e.g. a sudden unexplained jump/drop before telling Andy about it) — leaving
it on for every tracker/lookup adds real credit cost for marginal benefit on small sites.
**As of v2.18.0 this got more expensive:** the live verification includes converting
Google's redirect links, which DataForSEO starts billing separately from Nov 30, 2026
($0.004/keyword at depth 100) — keep it strictly opt-in, per-keyword, not a standing setting.

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

**Historical defect, now fixed (v2.17.0):** Search Console Performance totals, the chart,
and the keyword/page lists did not match Google's real numbers whenever a filter (country,
device, etc.) was applied — the filtered view read stale/wrong underlying data instead of
what Google itself uses. This also broke the CSV export, which largely ignored filters. Any
GSC analysis you ran filtered by country (e.g. Canada-only) or device before Nov 18, 2026 is
suspect — re-pull it rather than trusting the old numbers; filtered views are now correct.

**Historical defect, now fixed (v2.18.0):** GMB Rank Tracker double-counted a business
stored twice at the same grid point, producing negative averages, "N/a" values, and shares
over 100% on some grid reports. Any GMB grid/Progress report showing those symptoms before
Nov 24, 2026 reflects this bug, not a real local-ranking collapse — re-run the grid scan
rather than trusting the old numbers.

**Historical defect, now fixed (v2.18.0):** Sitemap pulls in Search Console and IndexNow
silently saved zero pages when the sitemap's `<lastmod>` dates used certain formats (a date
alone on its own line, or a short form like `2026-11`). If a site's sitemap/IndexNow
coverage looked empty before Nov 24, 2026, that's a reason to re-pull it before concluding
the sitemap itself is broken or missing.

**Historical defect, now fixed (v2.18.0):** the Search Console Indexing page's link crawler
saved Chrome error pages as if they were real page text and deleted the links a page had
previously been found to have, instead of keeping the last good crawl. If GSC Internal Links
data for a page looks thin or wrong, re-crawl it rather than trusting an old crawl.

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
- **Organic Rank Tracker imports** (v2.5.0, extended v2.16.0) — from True Ranker, SEO
  PowerSuite, Agency Analytics, and now Semrush Position Tracking (CSV: rank, ranking page,
  volume, CPC, intent, tags). Relevant when migrating an existing client off another tool.
- **Log File Analysis bot-list fix** (v2.16.0) — Claude-User and Perplexity-User hits were
  being counted as human traffic; both are now checked against their companies' published
  bot IP ranges before being counted as human. Also: Claude-SearchBot is now recognized,
  ClaudeBot moved from the AI Answer category to AI Training, and Google-Extended was
  dropped from the bot list. Only matters if Log File Analysis is in active use — fixes a
  historical overcount of "human" traffic on sites getting AI-answer-engine crawler visits.
  v2.17.0 adds a genuine human-page-views count (a Humans view on the Pages tab, plus a Logs
  vs GA4 vs Search Console comparison card on the Overview) on top of this fix — existing
  reports need logs re-uploaded or the log source re-run to show it.
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
- **Keyword Explorer Check Metrics fix** (v2.18.0) — Check Metrics on Autocomplete Keywords
  used to buy DataForSEO metrics for keywords hidden by your filters (wasted credit), and the
  Intent filter did nothing; both are fixed, and Check Metrics now waits for a running
  autocomplete job instead of racing it. Only matters if you use Autocomplete + Check Metrics
  together, but worth knowing since the old behavior silently spent credit on keywords you
  thought you'd excluded.
- **LLM Rank Tracker PDF export + automation** (v2.18.0) — a one-click PDF (key figures,
  visibility trends, top search terms, brands, competitors, citations, sentiment) and a "run
  completed" automation trigger that can export it, email it, or call a webhook. Same
  caveat as AI Visibility above: matters once a site has brand-name recognition worth
  tracking in AI answers, not for a brand-new rank-and-rent site.
- **GA4 Traffic integration** (v2.15.0) — syncs Google Analytics 4 (all channels, not just
  organic) into Your Sites: sessions, key events, and revenue by channel/source/month vs. an
  earlier period. The GSC Rank Tracker now shows sessions and key events per keyword's page,
  and Organic/GSC Rank Tracker exports and published client reports get a Traffic tab.
  **Why it matters:** only worth turning on once a site has GA4 plus key-event tracking
  (call/lead-form tracking) actually configured — then it ties a specific ranked page to
  real conversions instead of just position, which is the real test of whether that page
  deserves more internal links or content work, and gives something concrete to show a
  renting client. Not useful on a brand-new site with no conversion tracking set up yet.
