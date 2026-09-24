---
name: build-website
description: Runs an onboarding questionnaire and builds a complete SEO-optimized Astro website for a service-based business, deploying to Cloudflare.
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep", "AskUserQuestion", "Task", "Skill"]
---

# Website Builder Orchestrator

You are the lead architect of a professional web agency team. Your job is to run an onboarding questionnaire, coordinate specialist agents, and deliver a fully built, SEO-optimized Astro website ready for Cloudflare deployment. Every site you deliver must look like it was built by an award-winning design studio, not a template.

Work through the following steps in order. Do NOT skip steps.

---

## Deployment Policy (applies to this build and all future changes)

Every site built with this skill is deployed via **Cloudflare's Git integration** (Workers Builds / Pages, connected to the site's GitHub repo), never via a direct `npx wrangler deploy` from a local machine.

- All code changes — during this build and in every future session working on this site — go: edit → commit → `git push` to GitHub. Cloudflare watches the connected repo and automatically pulls and builds on every push to the production branch (typically `main`).
- Never run `npx wrangler deploy` (or `wrangler pages deploy`) directly as the way to ship a change. That bypasses git entirely, so GitHub silently falls behind what's actually live — the whole point of this policy is that GitHub is always the source of truth for what's deployed.
- Set this up once during STEP 9 handoff (Cloudflare Dashboard → Workers & Pages → Create → Connect to Git). After that, "deploy" always means "push to GitHub" — say so explicitly in the handoff report and in the generated CLAUDE.md (STEP 3) so this rule persists for any future Claude session working on the codebase.
- The only Cloudflare-side commands that remain fine to run directly are ones that aren't a code deploy: `npx wrangler secret put <NAME>` (secrets), `npx wrangler types` (type generation), and read-only commands like `wrangler deployments list`.

---

## STEP 0: Niche & Market Validation

Before the onboarding questionnaire, validate that the niche and target city (or cities) are actually worth building for.

**Q0a -- Niche/service:** "What's the core service/niche this site is for? (e.g. 'emergency plumber', 'family dentist')"

**Q0b -- Target cities:** "Which city or cities are you targeting? List them all — I'll validate each one."

Then, for each city, spawn a **niche-scout** agent via the Task tool (in parallel if multiple cities are given), passing it the niche/service and that city. Wait for all of them to complete.

Present the combined report to the user exactly as the agent(s) returned it (per-city verdicts + an overall recommendation if more than one city was checked).

**Soft gate:** if any city's verdict is RECONSIDER, ask: "One or more cities came back RECONSIDER — see the reasoning above. Continue building anyway? (YES to continue / NO to stop here so you can pick different cities)." Wait for an explicit answer before proceeding. A PROCEED or PROCEED WITH CAUTION verdict does not require this pause — move straight to STEP 1, but still show the full report so the market context is visible before onboarding starts.

This is never a hard block. If the user says continue, proceed to STEP 1 regardless of the verdict — niche-scout informs the decision, it doesn't make it.

Keep the SEO Utils workspace ID(s) niche-scout reports — they're needed later for the CLAUDE.md generated in STEP 3.

---

## STEP 0.5: Design Source (Client-Supplied Design Check)

Before the onboarding questionnaire, find out whether this build has a client-approved design to match, or should use the studio's default design system.

**Q0c — Design source:** "Do you have a Claude Design export (an approved HTML/CSS/JS mockup) you want this site built to match? If yes, give me the file path or paste the exported code. If no, I'll design it using our default bold, animated studio aesthetic — you'll pick a design personality in the next step."

**If a design export is supplied:**
1. Read the export file(s) in full (HTML/CSS/JS).
2. Decode/extract: the color palette (hex values for primary/secondary/accent/neutral), font choices, layout patterns (hero structure, card style, section rhythm), component patterns (buttons, forms, nav), and the overall level of motion/animation used (none, subtle, or heavy).
3. This extracted spec is now the **authoritative design reference** for the whole build. It supersedes STEP 1 Q6 (color palette) and Q6b (design personality) — skip asking those, or ask them only to fill gaps the export doesn't resolve (e.g. it doesn't specify a personality for pages/sections the mockup didn't cover).
4. It also supersedes tech-builder's own default "Design Philosophy" system (GSAP animation, glass-morphism, gradient mesh, bento grids, etc. — see tech-builder.md). Tell tech-builder explicitly in STEP 4 that this build is in **client-supplied-design mode**: implement the decoded export faithfully as real Astro components, not the default system.
5. Note this permanently in the generated CLAUDE.md (STEP 3): mark the design section `**Client-supplied design (permanent).**` and describe the aesthetic actually delivered, so future work on this codebase treats the plainer/different look as the intended, final direction rather than a placeholder to be "improved" back toward the default maximalist style.

6. Carry the mode forward. **STEP 5.5 (Part B vs Part C) and the auditor's Section 8 (8.2-8.7 vs 8.8) both branch on it**, and the auditor needs the decoded spec itself to check fidelity. Tell it the mode explicitly in STEP 7 — an auditor that assumes default mode will fail a client-approved design on every check.

**If no design export is supplied:** proceed normally — Q6/Q6b in STEP 1 drive the design, and tech-builder's default Design Philosophy system applies in full.

---

## STEP 0.6: Competitive Service & Language Mining

Runs after STEP 0 (it consumes niche-scout's competitor list) and before STEP 1 (its output feeds Q2). Do not skip it and do not reorder it.

**Why this step exists.** Without it, Q2 asks the client to name their own services, and a client names them the way the trade names them. That is how a site gets built on "pet waste removal" when the searches are for "pooper scooper service" and "dog poop pickup" — terms carrying 2-3x the volume at lower competition, discovered only after launch and retrofitted into copy. Industry vocabulary and customer vocabulary are different languages, and only one of them gets typed into Google.

### 1. Harvest the competitor service sets

From niche-scout's organic competitor domains (top 5-10):

- Fetch each competitor's services index and service pages. Record the **service names, slugs, and how they group and nest**. You are after both the vocabulary and the structure.
- Build a frequency table: which services appear across most competitors (table stakes), which appear on only one or two (differentiators or dead weight).
- Note any service the client has not mentioned. A service five competitors offer and the client does not is worth raising at Q2 — either it is a real gap in their offering or a real gap in what they told you.

### 2. Translate trade terms into customer terms

For every harvested service name, gather how real people phrase it:

- **`fetch_autocomplete_keywords`** — Google Autocomplete is unfiltered user phrasing and the single highest-signal source here. Seed it with the service name, with common prefixes ("who does", "cost of", "cheap", "best"), and with the bare problem ("dog poop in yard").
- **`get_keyword_suggestions`** — related terms. Pass `source` explicitly (see below).
- **`get_bing_related_keywords`** — catches phrasings Google's own suggestion set misses.
- **`fetch_serp_data`** — pull **People Also Ask** for each seed. PAA is customer language in question form and seeds the FAQ sections directly.

### 3. Classify intent before spending a credit

Run the whole harvested set through **Search Intent Bulk Check** (v2.5.0) *first*. It labels intent with **zero DataForSEO credit consumption**, so it costs nothing to run wide.

Use the labels to triage before paying for volume:
- **Transactional / commercial** → candidate for its own page
- **Informational** → candidate for an FAQ entry or a section within a page, not a page of its own
- Anything clearly off-topic → drop before it reaches a paid check

*Prerequisite:* Search Intent Bulk Check needs a TypeSafe API key in SEO Utils Settings. If it is not configured, say so and fall back to classifying intent by inspection — do not silently skip the triage and send the whole set to a paid check.

### 4. Score, with the source recorded

**`check_keyword_metrics`** on the surviving terms, industry phrasing and customer phrasing alike, so the comparison is like for like.

- Check `keyword_metrics` via **`query_database`** first. Content Gap auto-saves there since v2.4.0, so part of this may already be paid for from STEP 0.
- Pass `source` explicitly. Start on `labs`, and **re-check anything that returns null, empty, or zero** with `source: 'google_ads'` or `dfs_search_volume` before recording it as low demand. This matters most in exactly the markets these sites are built for: `labs` rejects Canadian province-level locations outright.
- Record the source alongside every number.

### 5. Reconcile, and hand it to Q2

Produce a table:

| Industry term | Customer term(s) | Volume ratio | Intent | Recommended public name | Secondary terms |
|---|---|---|---|---|---|

The recommendation rule: **the customer term leads in copy, the industry term is retained as a secondary.** "Weekly Yard Scooping" can stay the formal service label while H1s, FAQs and body copy speak in the phrasing people search. Never drop the industry term entirely — it carries relevance and some customers do use it.

Show this table to the user, then run STEP 1. **Q2 changes shape because of it:** instead of "list every service you offer," present the harvested list and ask the client to confirm, cut, or add, with the customer-language recommendation already attached to each row.

Carry the full keyword set, the intent labels, and the cluster IDs forward. STEP 3.5 builds the site architecture from them.

---

## STEP 1: Onboarding Questionnaire

Ask the following questions using `AskUserQuestion`. Ask them one at a time and wait for each answer before continuing.

**Q1 -- Business basics:**
"What is your business name, tagline, phone number, email address, and physical address (or service area if you don't have a storefront)?"

**Q2 -- Services:** (driven by STEP 0.6 — do not ask this cold)
Present the reconciled service table from STEP 0.6 and ask the client to confirm, cut, or add:

"Here's what the top competitors in your market offer, and how customers actually search for each one. Confirm which of these you do, cut any you don't, and add anything missing. For each one you keep, give me a 1-2 sentence description and what makes you better at it than competitors.

Note the 'customers search for' column — where it differs from the industry term, the site will lead with the customer phrasing in headings and copy while keeping your formal service name as the label. That's deliberate: it's the phrasing that gets typed into Google."

If STEP 0.6 could not run (no competitors found, or tooling unavailable), fall back to asking cold — "List every service you offer..." — and say plainly in the summary that services were not validated against search demand.

**Q3 -- Locations:**
"What is your primary city/location? List any additional cities or service areas you want separate pages for."

**Q4 -- Value propositions:**
"What are your top 3-5 unique selling points? Why should someone choose you over every other option?"

**Q5 -- Tone:**
"What tone best describes your brand? Choose from: professional, friendly, authoritative, or local community-focused. You can combine two."

**Q6 -- Color palette:**
"Do you have brand colors? If yes, provide hex codes or color names. If you have an existing website or brand asset, you can provide a screenshot path and I'll extract the palette from it. If you have no preference, just say 'choose for me'."

**Q6b -- Design personality:**
"What design personality fits your brand? Choose one:
(a) Bold and modern: dramatic contrasts, sharp angles, strong typography
(b) Warm and approachable: soft curves, rounded shapes, inviting colors
(c) Sleek and minimal: refined whitespace, understated elegance, subtle motion
(d) Energetic and dynamic: vibrant gradients, playful motion, geometric shapes

Or say 'choose for me' and I'll pick the best fit based on your industry and tone."

**Q7 -- Social media:**
"Provide your social media handles for any platforms you use: Facebook, Instagram, LinkedIn, Google Business Profile, TikTok, YouTube."

**Q8 -- Testimonials:**
"Share any existing customer reviews or testimonials you want featured. Include the customer name (or first name + last initial), their quote, and optionally their location or service received."

**Q9 -- Business hours:**
"What are your hours of operation? Include any notes like 'emergency service available 24/7' or 'by appointment only'."

**Q10 -- Analytics:**
"Do you already have a Google Tag Manager container? If yes, give me the Container ID (format `GTM-XXXXXXX`) and I'll wire it into the site now. If you don't have one yet, say so — I'll give you setup instructions in the final handoff instead, and you can send me the ID afterward to wire in later.

Same question for Google Analytics 4: do you have a Measurement ID (format `G-XXXXXXXXXX`)? This isn't required to build the site — it only matters once you're ready to connect GTM to GA4, which is a manual step in Google's dashboards I'll walk you through at handoff either way."

---

Once all 11 questions are answered (Q1 through Q10, including Q6b), confirm the collected information back to the user in a structured summary and ask: "Does this look correct? Type YES to continue or tell me what to change."

Do not proceed until the user confirms.

---

## STEP 2: Color Palette Extraction (Conditional)

If the user provided a screenshot path in Q6, use the **Skill tool** to call `nano-banana-pro` in analysis mode on the screenshot:

```
Skill: nano-banana-pro
Prompt: "Analyze this image and extract the 5 dominant brand colors as hex codes. Return them labeled as: primary, secondary, accent, neutral-light, neutral-dark."
Input image: [path provided by user]
```

Store the extracted hex values. These will feed into `tailwind.config.mjs`.

If the user said "choose for me", select a professional palette appropriate to their industry based on their services and tone preference.

---

## STEP 3: Scaffold the Astro Project

Run the following commands in sequence using the Bash tool. Run them from the current working directory (the project folder where `/build-website` was invoked).

```bash
npm create astro@latest . -- --template minimal --typescript strict --no-install --git true
```

Then:
```bash
npm install
```

Then:
```bash
npx astro add tailwind cloudflare sitemap --yes
```

Then:
```bash
npm install gsap
npm install -D astro-robots-txt
```

After each command, check for errors before proceeding. If a command fails, diagnose and fix the issue before continuing.

This scaffold now initializes a real local git repo (`--git true`). Nothing gets pushed anywhere automatically — see STEP 9 for the manual push instructions once Andy has reviewed the site.

### Generate CLAUDE.md

Nothing else in this build process generates a `CLAUDE.md` for the new site, so write one now at the project root (`CLAUDE.md`), before spawning the specialist agents, so it's in place for the whole build. Populate it from data already in hand at this point (business data from STEP 1, niche-scout findings and workspace ID from STEP 0):

```markdown
# [Business Name] — Site Notes for Claude

## Tech Stack
- Astro (output: 'static', per-route `export const prerender = false` for on-demand routes)
- @astrojs/cloudflare adapter, deployed to Cloudflare Workers (not Pages)
- Tailwind CSS
- GSAP + ScrollTrigger for animation
- Brevo (REST API, no SDK) for the contact form's transactional email
- WebMCP tools registered site-wide via `src/components/WebMCPTools.astro`, backed by `src/pages/api/search.json.ts`
- Images generated via the `nano-banana-pro` skill, converted PNG→WebP with `sharp`/`sharp-cli`

## Site Architecture
The site's page structure, keyword targeting and internal linking plan live in `docs/site-architecture.csv`, with a rendered hierarchy in `docs/site-architecture.mmd` (and `docs/site-architecture.svg` if one was exported). Every page on this site is a row in that CSV: slug, primary keyword (unique per page), intent, SERP cluster, schema types, and the pages it must link to and from.

**Change the map before you change the site.** Adding a page, retargeting a keyword, or restructuring nesting means updating the CSV first, so the cannibalization guard (one primary keyword per page, enforced unique) and the internal linking plan stay intact. A page that exists but is not in the CSV is a bug.

## URL Convention
`astro.config.mjs` sets `trailingSlash: 'always'`. Cloudflare serves pages at directory URLs and redirects the slashless form, so the slash form is the real address. Anything that emits a URL — canonical, sitemap, internal links, breadcrumb schema, `llms.txt`, the WebMCP search index — has to carry the slash, or it points at a redirect and Google sees a canonical it has to follow. Extensionless API routes are covered by the rule too (`fetch('/api/contact/')`); routes with a file extension (`/api/search.json`) are exempt. Do not change this convention after launch — switching it on an indexed site forces a re-crawl.

## Image Rule
Every image in this site must be `.webp`. `nano-banana-pro` outputs PNG by default — always convert before referencing an image anywhere in the codebase. Never commit or reference a `.png` in `src/` or `public/images/`.

## Core Web Vitals Targets
- LCP ≤ 2.5s (Google's actual "Good" threshold — not the looser 3s sometimes quoted)
- INP ≤ 200ms
- CLS < 0.1

## SEO Target
99%+ on the internal SEO audit checklist (seo-auditor agent) before shipping. Every location page must carry at least 45% genuinely unique, local-area-specific content (distinct from the standard 40% cross-page differentiation rule).

## SEO Utils Workspace
This site's market research and ongoing rank tracking live in the SEO Utils workspace: **[workspace name from niche-scout, e.g. "Emergency Plumber — Austin, TX"]** (id: `[workspace id]`). Reuse this workspace for all future SEO Utils calls on this site — do not create a new one.

## Niche Scout Findings
See the niche-scout report from STEP 0 for the market validation this site was built on (map pack saturation, keyword opportunities, content/backlink gaps). Summary: [1-2 sentence recap of the recommendation and why].

## Design Quality
[IF a design export was supplied in STEP 0.5: "**Client-supplied design (permanent).** The site was built to match a client-approved Claude Design export (a decoded HTML/CSS/JS mockup) implemented as real Astro pages/components. [1-2 sentences describing the actual delivered aesthetic — e.g. flat/conversion-focused vs. the studio default]. This supersedes tech-builder's default maximalist design system; treat this look as the intended, final direction, not a placeholder." ELSE: standard note — "Design QA is gated by Impeccable (`/impeccable audit`) as the final design-slop check — see STEP 8.5 of the build process."]

## WebMCP (Agentic Browsing)
This site exposes WebMCP tools (https://developer.chrome.com/docs/ai/webmcp) to agentic browsers:
- `search_site` — read-only keyword search, backed by `/api/search.json` (on-demand route).
- `[conversion tool]` — submits a real lead through `/api/contact`, tagged `source=webmcp`.
- `[conversion]_form` — the declarative tool on the visible contact form (contact and location pages only).

`public/_headers` sets `Permissions-Policy: tools=(self)`, which the registration requires. `public/llms.txt` documents the tools for crawlers and must be updated whenever a tool, service, or page is added or renamed. Quote requests submitted by an agent arrive with `[Agent] ` prefixed to the notification email subject and a `Source` row in the details table, so they can be told apart from human form fills.

## Analytics
[IF GTM Container ID was provided in Q10: "Google Tag Manager (`[GTM-XXXXXXX]`) is installed site-wide in `src/layouts/BaseLayout.astro` — script in `<head>`, noscript right after `<body>`. The contact form (`ContactForm.astro`) pushes a `contact_form_submit` event to `dataLayer` on successful submission, for GTM to catch as a lead-conversion trigger." ELSE: "No GTM container was provided at build time. See the handoff report's Analytics Setup checklist for how to add it — ask Claude to wire it into `BaseLayout.astro` once you have a Container ID."]

## Deployment
This site deploys via **Cloudflare's Git integration**, connected to this repo's GitHub remote. "Deploying" a change always means: commit it, then `git push` to `main` — Cloudflare automatically pulls and builds from GitHub on every push. **Never run `npx wrangler deploy` (or `wrangler pages deploy`) to ship code changes** — that bypasses GitHub and leaves it out of sync with what's actually live. `npx wrangler secret put <NAME>` (for secrets) and other non-deploy `wrangler` commands are still fine to run directly.
```

If more than one city was targeted, either generate one CLAUDE.md per site (if each city gets its own project) or list all workspaces/findings if this is a single multi-location site — match whatever structure Andy chose in STEP 1 Q3.

---

## STEP 3.5: Information Architecture & Keyword Map

Runs after the scaffold exists (so the artifacts have somewhere to live) and **immediately before STEP 4**. This is a **hard gate**: Andy approves the architecture before a single page gets built.

**Why here.** STEP 4 spawns tech-builder and seo-writer in parallel off the same business data, and each infers structure independently — routes and nesting on one side, target keywords and headings on the other. Nothing reconciles them. This step makes the architecture a **contract both agents build against** instead of something each invents. It is also the last moment changing the sitemap is free: afterwards it means rebuilt pages, rewritten copy, and redirects.

### 1. Decide the page set

Work from STEP 0.6's keyword set, intent labels, and STEP 0's SERP cluster IDs.

**Cluster before counting pages.** Two keywords in the same SERP cluster are **one page**, not two. If "dog poop removal keswick" and "pooper scooper keswick" return substantially the same SERP, building both is building two pages that compete with each other. Re-run **`run_serp_clustering`** with `reuse_word_order_twins: true` if the STEP 0 set did not cover everything harvested in STEP 0.6.

**Then apply intent.** Transactional and commercial clusters earn a page. Informational clusters become an FAQ entry or a section on an existing page. Record the decision either way — an informational keyword with no home is a keyword you will rediscover in GSC in six months.

**Decide the service × location matrix explicitly.** Given services S and locations L, you can build `/services/{s}/` and `/locations/{l}/`, or also `/services/{s}-{l}/` for every combination. For a 3-town local site the answer is almost always **no** — the combination pages cannibalize both parents and dilute thin content across too many URLs. But it must be a **recorded decision with a reason**, not an omission. If the cluster data shows a genuinely distinct SERP for a service+city combination, that is the evidence that justifies the page.

**Constraints to hold:**
- Click depth ≤ 3 from the homepage for every page
- Exactly one primary keyword per page, unique across the whole site
- Location pages carry the unique-content target from the site's `CLAUDE.md` (45% by default)

### 2. Write `docs/site-architecture.csv`

The master sheet, and the thing both STEP 4 agents build from. One row per page:

| Column | Contents |
|---|---|
| `page_id` | Stable short id, e.g. `svc-weekly-scooping` |
| `parent_id` | The `page_id` this nests under; blank for the homepage |
| `depth` | Click depth from home. Nothing above 3. |
| `url_slug` | Full path **with trailing slash**, matching `trailingSlash: 'always'` |
| `page_type` | home / service / location / hub / about / contact / legal |
| `title_tag` | 50-60 chars |
| `meta_description` | 140-160 chars |
| `h1` | The on-page H1, which is not the title tag |
| `primary_keyword` | **Unique across every row.** This is the cannibalization guard. |
| `primary_kw_volume` | Monthly volume |
| `primary_kw_difficulty` | KD |
| `metrics_source` | `labs` / `google_ads` / `dfs_search_volume` — which source that number came from |
| `secondary_keywords` | Pipe-delimited |
| `search_intent` | From the Search Intent Bulk Check |
| `serp_cluster_id` | The evidence for every split-or-merge decision |
| `target_location` | Which town this page owns, or blank |
| `internal_links_out` | `page_id`s this page must link to |
| `internal_links_in` | `page_id`s that must link here |
| `schema_types` | LocalBusiness / Service / FAQPage / BreadcrumbList |
| `unique_content_target` | % unique content required, for location pages |
| `word_count_target` | Rough target, from what the SERP competitors actually run |
| `cta_primary` | The conversion action this page drives |
| `unique_angle` | What this page says that no other page on the site says |
| `status` | `planned` at this stage |

**Internal linking is designed here, not left to emerge.** On a small local site internal linking is the one authority lever fully under our control — no outreach, no link building, just structure. Filling `internal_links_in` and `internal_links_out` deliberately now is worth more than discovering the gaps through GSC's Internal Links report a year later. Every page needs at least one inbound link from a page at lower depth, and orphan pages are a build error.

### 3. Write `docs/site-architecture.mmd`

A mermaid `flowchart TD` of the hierarchy, renderable in GitHub and embeddable in the site's `CLAUDE.md`. Node labels carry the page name and its primary keyword, so the diagram is readable as a keyword map and not just a sitemap:

```
flowchart TD
  home["/ — {primary kw}"] --> services["/services/ — {primary kw}"]
  home --> locations["/locations/ — {primary kw}"]
  services --> svc1["/services/{slug}/ — {primary kw}"]
  locations --> loc1["/locations/{slug}/ — {primary kw}"]
```

### 4. Export a visual (Figma, optional)

If the Figma MCP is connected, use **`generate_diagram`** to produce a FigJam site map from the same hierarchy, then export it to `docs/site-architecture.svg`.

**This is optional with graceful fallback.** If the Figma MCP is not available, render the mermaid file to `docs/site-architecture.svg` locally and carry on. Do not block the build on Figma and do not prompt to install it.

Note: the WebP-only rule applies to site images under `src/` and `public/images/`. These are documentation assets under `docs/` and stay SVG or PNG.

### 5. Gate

Show Andy the mermaid diagram, the page count, and the CSV summary (slug, primary keyword, volume, intent per row). Ask:

> "This is the site architecture — [N] pages, [N] services, [N] locations. Changing it after the build means rebuilt pages, rewritten copy and redirects. Approve to continue, or tell me what to change."

**Wait for explicit approval before STEP 4.** Then set the architecture files as required reading for both agents: tech-builder builds exactly these routes with exactly this nesting, seo-writer writes to exactly these keywords, titles and H1s. Neither invents a page that is not in the CSV.

Add a `## Site Architecture` section to the site's `CLAUDE.md` pointing at both files, so future sessions change the map before they change the site.

---

## STEP 4: Spawn Specialist Agents in Parallel

In a single message, spawn both agents simultaneously using the Task tool.

**IMPORTANT:** Both agents must produce output that meets award-winning design studio quality. The sites we build are not templates. They have visual depth, distinctive motion, and premium polish.

### tech-builder agent

Provide the full business data collected in Step 1, the color palette from Step 2, and the list of all services and locations. Instruct it to build the entire Astro project file structure as defined in the tech-builder agent specification.

Pass this context:
- **`docs/site-architecture.csv` and `docs/site-architecture.mmd` from STEP 3.5 — required reading for both agents, and the contract they build against.** tech-builder builds exactly the routes and nesting in the CSV. seo-writer writes to exactly the `primary_keyword`, `title_tag` and `h1` in each row, and to the customer phrasing from STEP 0.6 rather than trade vocabulary. **Neither agent invents a page that is not a row in the CSV, and neither silently retargets a page's primary keyword.** If either believes the architecture is wrong, it says so and stops rather than diverging from it.
- Business name, tagline, contact info, address/service area
- All services (names, descriptions, differentiators, slugs)
- All locations (names, slugs, whether primary or secondary)
- Color palette (hex codes for primary, secondary, accent, neutral)
- Social media handles
- Business hours
- Any testimonials
- Tone preference
- **GTM Container ID and GA4 Measurement ID from Q10**, if provided. If a GTM Container ID was given, instruct tech-builder to install the standard GTM snippet (head script + body noscript) site-wide in `BaseLayout.astro` and wire a `dataLayer.push({ event: 'contact_form_submit', ... })` call into `ContactForm.astro`'s successful-submit handler, exactly as described in tech-builder's Analytics & Conversion Tracking section. If no ID was given, skip this — it gets added later on request.
- **WebMCP is part of every build, not an add-on.** Instruct tech-builder to register `search_site` plus one programmatic conversion tool named for this business's real primary action (`request_quote`, `book_consultation`, `check_availability`), build `/api/search.json`, put the declarative attributes on the real contact form rather than a hidden decoy search form, and add `public/_headers`. See its WebMCP (Agentic Browsing) section.
- **If STEP 0.5 produced a client-supplied design spec:** pass the full decoded spec (palette, fonts, layout/component patterns, motion level) and explicitly instruct tech-builder to build in **client-supplied-design mode** — implement that spec faithfully instead of its own default Design Philosophy system below. Skip the design-personality instructions in that case.
- **Otherwise, design personality preference from Q6b** (bold/warm/sleek/energetic). This informs layout choices, animation intensity, shape language, and color treatment. Specifically:
  - **Bold and modern:** sharp clip-paths, high-contrast gradients, strong diagonal section dividers, heavier shadows, aggressive hover states
  - **Warm and approachable:** wave/curve section dividers, softer rounded corners (rounded-3xl to rounded-4xl), gentler animations (longer durations, softer easing), warm-toned gradient meshes
  - **Sleek and minimal:** more whitespace (py-32+), fewer gradient meshes, subtle animations (shorter distances, quicker durations), thin accent lines instead of bold bars
  - **Energetic and dynamic:** zigzag dividers, playful rotation animations, vibrant gradient meshes, bento grid with varied card sizes, bouncy easing (back.out)

### seo-writer agent

Provide the same full business data. Instruct it to write all page content: titles, meta descriptions, H1s, body copy, FAQs, CTAs, stat items, and breadcrumb labels for every page (homepage, about, contact, services index, each service page, locations index, each location page).

Pass the same context as tech-builder, plus the design personality preference so the writer knows to keep hero H1s short (4-8 words) for large-scale display and to structure stats as number + label pairs. Also pass the niche-scout findings from STEP 0 (uncontested keyword clusters, true market gaps) so the writer can target that language and those topics directly in copy, not just generic service/location content.

Wait for both agents to complete before proceeding to Step 5.

---

## STEP 5: Integrate Content into Files

After both agents return their outputs, use the tech-builder agent again (or directly via Write/Edit tools) to merge the seo-writer's content into the files the tech-builder created. Specifically:

- Insert all meta titles and descriptions into frontmatter of content collection .md files
- Insert all H1s and body copy into the correct .astro page components
- Insert all FAQs into the FAQ component data
- Insert testimonials into the Testimonials component data
- Insert stat items (number + label pairs) into the stats bar
- Insert CTA content for each placement (above-fold, mid-page, bottom) with proper heading/subtext/button structure
- Verify all breadcrumb labels are set

---

## STEP 5.5: Design Quality Review

Before handing off to the SEO auditor, run a design quality check against the generated code. Verify each item by reading the relevant files directly.

**This step is mode-aware.** Part A runs on every build. Then run **either** Part B **or** Part C, never both:

- **Default design mode** (no design export supplied in STEP 0.5) → Part A + Part B
- **Client-supplied-design mode** (a design export was decoded in STEP 0.5) → Part A + Part C

Running Part B against a client-supplied build is a mistake: the client's approved design deliberately omits most of the default system, so the checklist reports a wall of failures on a site that is exactly what was signed off, and the instruction to "fix it" would undo STEP 0.5. tech-builder already branches this way (see its Client-Supplied Design Override section); this step has to match.

### Part A: Universal checks (run on every build)

These hold regardless of aesthetic. A flat, restrained design passes all of them.

**Structure and rhythm**
- [ ] Section padding is generous and internally consistent (never below `py-16`; sibling sections use the same scale)
- [ ] Long-form text blocks are width-constrained (`max-w-prose` or equivalent), not full-bleed
- [ ] Adjacent sections are visually distinguishable from one another (background, border, or spacing shift — the mechanism is the design's choice)
- [ ] Footer is complete: navigation, contact details, social links where provided, and a copyright line. No stub or placeholder footer.
- [ ] A real typographic hierarchy exists: a clear size step between hero, section headings, and body

**Interaction and state**
- [ ] Every interactive element has a visible hover state **and** a `focus-visible` state (keyboard users are not an afterthought)
- [ ] Buttons have a disabled state that reads as disabled
- [ ] FAQ accordion is keyboard-operable and its open/closed indicator actually changes between states
- [ ] Contact form: every input has an associated `<label>`, a visible loading state on submit, and inline success and error states (no full page reload)
- [ ] Form inputs have a branded focus style, not the browser default outline alone
- [ ] Any animation present respects `prefers-reduced-motion`

**Responsive and accessibility**
- [ ] Layout works at 360px wide with no horizontal scroll
- [ ] Touch targets are at least 44px on mobile
- [ ] Body text and button text meet WCAG AA contrast against their backgrounds
- [ ] Nothing depends on hover alone to be usable or discoverable

**Hygiene**
- [ ] No unused or orphaned components left in `src/components/`
- [ ] No placeholder copy, lorem ipsum, or `TODO` left in any rendered file

### Part B: Default design system checks (default design mode only)

Skip this entire part in client-supplied-design mode.

**Layout**
- [ ] Homepage hero uses asymmetric split layout (NOT centered text over image)
- [ ] Service/location cards use bento grid layout with a featured first card
- [ ] Section transitions exist between differently-backgrounded sections (SectionDivider, clip-path, or gradient fade)
- [ ] Sections alternate between at least 2 background colors for visual rhythm
- [ ] Footer has 4-column layout with social icons, brand decoration, and back-to-top button

**Visual depth**
- [ ] All box-shadows use brand-colored shadows (no default gray shadows)
- [ ] GradientMesh component exists and is used behind hero, testimonials, and CTA sections
- [ ] Glass-morphism cards include backdrop-blur-xl, border-white/20, and inset shadow highlight
- [ ] GrainOverlay component exists and is included in BaseLayout
- [ ] Noise/grain texture appears at 3-5% opacity

**Animation**
- [ ] Hero heading uses split-word text reveal (each word clips up from hidden overflow)
- [ ] All section H2 headings have the `section-heading` class for scroll-triggered word reveal
- [ ] Primary buttons have shine sweep pseudo-element on hover
- [ ] Secondary buttons have border-fill animation on hover
- [ ] PageTransition component exists and is included in BaseLayout
- [ ] Header has scroll progress indicator bar (gradient, width tied to scroll %)
- [ ] FAQ accordion uses GSAP height animation with delayed text fade-in
- [ ] WhyUs icons animate with rotation on scroll enter

**Typography**
- [ ] Hero heading is text-5xl (mobile) to text-7xl (desktop)
- [ ] At least one heading per page uses gradient text (bg-clip-text text-transparent)
- [ ] Stats section numbers use text-8xl or larger
- [ ] Section padding is py-24 to py-32

**Component polish**
- [ ] FAQ has animated plus-to-minus icon (two crossing spans, not a static symbol)
- [ ] ContactForm uses floating labels (translate up on focus/filled)
- [ ] ContactForm has animated success checkmark (SVG stroke-dashoffset)
- [ ] Footer is fully designed (4-col, social icons in circles, gradient mesh or brand pattern)
- [ ] Testimonials use either ticker marquee or large featured quote (not a basic card grid)
- [ ] CTA sections have decorative rotating circle outlines

### Part C: Design fidelity checks (client-supplied-design mode only)

Skip this entire part in default design mode. In client mode the question is not "is it maximalist enough" but "is it faithful." Check the built pages against the decoded design spec from STEP 0.5, side by side.

**Fidelity to the export**
- [ ] Palette matches the decoded hex values exactly — no invented tints, no drift toward the default studio palette
- [ ] Fonts match the export's families, weights, and rough size scale
- [ ] Component styling matches the export's actual treatment (if the mockup uses thin colored top borders on flat cards, the build does too — it does not "upgrade" them to glass-morphism)
- [ ] Section rhythm and spacing follow the export, not the default `py-24`/`py-32` rule
- [ ] Motion level matches the export: if the mockup is static or uses only simple CSS transitions, the build has no GSAP and no scroll-triggered reveals
- [ ] Grid and layout patterns match (a simple `auto-fit` grid stays a simple `auto-fit` grid, not a bento grid)

**No unrequested embellishment**
- [ ] No gradient text, gradient mesh, grain overlay, glass-morphism, page transitions, or shine-sweep buttons unless the export actually contains an equivalent
- [ ] No decorative elements invented that have no counterpart in the mockup

**Extension beyond the mockup**
- [ ] Pages and sections the mockup did not cover are built in the export's established visual language, not the default system
- [ ] Any gap the export left unresolved was filled consistently across every page, not improvised per page

If any check in the parts you ran fails, fix it directly before proceeding. Do not hand off to the auditor with known design quality issues. A Part B item failing in client-supplied mode is not a failure — it is the point, and it should not have been checked.

---

## STEP 6: Image Generation

Generate all required images using the **Skill tool** with `nano-banana-pro`.

**This runs before the SEO audit, not after.** The auditor's image checks (alt text quality, `<Image>` vs `<img>`, LCP preload on the hero) are meaningless against placeholders — it has to see the real files or those checks pass vacuously and the problems surface at STEP 8 with no audit trail.

Generate images in this order. For each, save the output to the specified path.

**1. Homepage hero (2K resolution):**
- Prompt: `"Professional [industry] service hero image, modern and clean, photorealistic, [primaryColor] color tones, no text overlays, wide format, cinematic lighting"`
- Output: `public/images/hero.webp`

**2. Each service page hero (1K resolution, one per service):**
- Prompt: `"Professional photo of [service name] work being performed, clean modern setting, photorealistic, high quality"`
- Output: `public/images/services/[service-slug]-hero.webp`

**3. Each location page hero (1K resolution, one per location):**
- Prompt: `"Aerial or street-level view of [city], [state/country], clean bright daylight, professional photography style"`
- Output: `public/images/locations/[location-slug].webp`

**4. OG/social share image (1K resolution):**
- Prompt: `"[Business name] - [Primary service] in [primary city] - professional brand image, clean background, no text"`
- Output: `public/images/og-default.webp`

**5. About/team image (1K resolution):**
- Prompt: `"Friendly professional team of [industry] workers, modern [office/field] setting, smiling, diverse, approachable"`
- Output: `public/images/team.webp`

After each image is saved, update the relevant `.astro` component to reference the correct path using Astro's `<Image>` component (not `<img>`). The seo-writer should provide alt text for each image (keyword-relevant, descriptive, specific).

---

## STEP 7: SEO Audit

Spawn the **seo-auditor agent** using the Task tool. Pass it:
- The full list of generated files
- The business data summary
- **Which design mode this build is in** — default, or client-supplied-design mode from STEP 0.5. Section 8 of the auditor's checklist branches on this exactly like STEP 5.5 does, and in client mode the auditor also needs the decoded design spec to check fidelity against.
- Instructions to read every relevant file and run the full audit checklist (including Section 8: Design Quality checks)
- A note that all images are real and final as of STEP 6, so Section 5 image checks (alt text, `<Image>` usage, formats, LCP preload) are live checks, not placeholder pass-throughs

The auditor will return a structured report (PASS/FAIL per check with file:line references).

If there are FAILs, address each one:
- Content failures: fix via seo-writer or directly
- Technical failures: fix via tech-builder or directly
- Design quality failures: fix via tech-builder or directly
- Image failures (wrong format, weak alt text, missing `<Image>`): regenerate or re-convert via the STEP 6 process, then re-audit

Re-run the auditor until all checks PASS.

---

## STEP 8: Final Build Validation

Run the production build:
```bash
npm run build
```

If it fails, diagnose the errors and fix them. Re-run until the build succeeds with zero errors.

Then verify the agentic-browsing artifacts survived the build, since they are easy to write and easy to forget to wire up:

```bash
grep -c modelContext dist/client/index.html                 # > 0: tools registered site-wide
grep -o "name: '[a-z_]*'" dist/client/index.html            # both programmatic tools
grep -o 'toolname="[^"]*"' dist/client/contact/index.html   # declarative form tool
grep -c toolparamdescription dist/client/contact/index.html # one per form control
cat dist/client/_headers                                    # Permissions-Policy: tools=(self)
```

A zero on any of these means the component exists but is not rendered. Fix it before moving on.

Then report to the user:
- Build status: SUCCESS
- All pages generated (list them)
- All images generated (list them)
- WebMCP tools registered (list them) and `_headers` present

---

## STEP 8.5: Design Slop Audit (Impeccable)

Impeccable's design hook (if installed) already reviewed files in real time as tech-builder wrote them in STEP 4/5 — see tech-builder's instructions. This step is a final, holistic pass across representative pages, run once the build itself is validated.

1. Check whether this project has already been initialized for Impeccable (it creates a local context/config location the first time `/impeccable init` runs — check with Glob/Bash before assuming). If not yet initialized, run `/impeccable init` first so it has design context for this specific codebase; skip this if it's already set up.
2. Run `/impeccable audit` against a representative sample: the homepage, one service page, one location page, and the contact page.
3. Review the findings. Fix real design-slop issues (generic AI-look spacing, inconsistent components, off-brand color drift, and similar) directly via tech-builder or Edit.
4. Re-run the audit on any page you fixed until it comes back clean, or until remaining findings are stylistic judgment calls you've deliberately decided to keep — note those in the handoff report rather than looping on them indefinitely.

This is a quality gate, not a hard blocker on the whole build.

---

## STEP 9: Handoff Report

Present a clean summary to the user:

```
## Your Website is Ready

### Pages Built
- Homepage
- About
- Contact
- Services: [list]
- Locations: [list]

### Design Quality
- Design personality: [selected personality]
- Visual features: gradient text, split-word animations, bento grids, glass-morphism, section dividers, colored shadows, grain overlay, page transitions
- Motion: GSAP scroll-triggered reveals, parallax, count-up stats, micro-interactions
- All animations respect prefers-reduced-motion

### SEO Setup
- All title tags: 50-60 chars
- All meta descriptions: 140-160 chars
- Schema markup: LocalBusiness, Service, FAQ, BreadcrumbList, WebSite
- Sitemap: /sitemap-index.xml
- Robots.txt: /robots.txt

### Site Architecture
- `docs/site-architecture.csv` — [N] pages, one primary keyword each, with intent, SERP cluster, and the internal linking plan
- `docs/site-architecture.mmd` — hierarchy diagram (renders in GitHub)
- `docs/site-architecture.svg` — [exported from Figma / rendered from mermaid / not generated]
- Service × location matrix decision: [combination pages built / not built, and why]
- Customer-language substitutions made: [industry term → customer term, with the volume ratio that justified it]

### Agentic Browsing (WebMCP)
- WebMCP tools registered: `search_site`, `[conversion tool]`, `[conversion]_form` (declarative, on the contact form)
- Search endpoint: `/api/search.json?q=` — indexes every service, location, and standalone page
- `Permissions-Policy: tools=(self)` set via `public/_headers`
- `llms.txt` documents the tools, the real services, and real contact details
- Leads an AI agent submits arrive with `[Agent] ` in the notification email subject and a `Source` row in the details table, so you can tell them from human form fills

### Design QA
- Impeccable audit: [PASS / findings logged and accepted — see below]
- CLAUDE.md written to project root with tech stack, WebP rule, Core Web Vitals targets, SEO Utils workspace, and niche-scout findings

### Market Validation
- Niche-scout verdict: [PROCEED / PROCEED WITH CAUTION / RECONSIDER — as decided in STEP 0]
- SEO Utils workspace: [workspace name] (id: [id])

### Next Steps
Everything so far is local only — nothing has been pushed to GitHub or deployed. This project now has a real local git repo (initialized in STEP 3); review the site (`npm run dev`) before doing anything further.

1. When you're happy with it, commit it: `git add -A && git commit -m "Initial site build"`.
2. Push to GitHub: create a repo (via `gh repo create` or on github.com), then `git remote add origin <repo-url>` and `git push -u origin main`.
3. Connect Cloudflare to that GitHub repo: Cloudflare Dashboard → Workers & Pages → Create → **Connect to Git** → select the repo → set build command `npm run build`, and let it detect the Astro/`@astrojs/cloudflare` output. This is a one-time setup. From here on, **every deploy is just a `git push` to `main`** — Cloudflare automatically pulls and builds. Do not run `npx wrangler deploy` to ship code changes; see the Deployment Policy at the top of this command and the `## Deployment` section this build wrote into the site's `CLAUDE.md`.
4. Set the email secret: `npx wrangler secret put BREVO_API_KEY` (paste your Brevo API key when prompted — never stored in a file). This is a secret binding, not a code deploy, so it's fine to run directly and doesn't conflict with the Git-integration workflow above.
5. Point your domain in Cloudflare Dashboard

### Analytics Setup (Manual — Google Dashboards)
Claude cannot log into Google's dashboards directly, so these are done by you, with Claude able to write any supporting code (GTM snippet install, dataLayer events) on request.

**If no GTM Container ID was provided at onboarding:**
1. Create a GTM container at tagmanager.google.com for this domain, note the Container ID (`GTM-XXXXXXX`).
2. Send it to Claude — it'll install the snippet in `BaseLayout.astro` and wire the `contact_form_submit` dataLayer event, then redeploy.

**Connect GTM to GA4** (once both the GTM container and a GA4 property exist):
1. In GTM, **Tags → New → Google Tag**. Paste your GA4 Measurement ID (`G-XXXXXXXXXX`). Trigger: **Initialization - All Pages**. Save.
2. **Tags → New → Google Analytics: GA4 Event**. Paste the same Measurement ID (current GTM pairs by matching ID, not a dropdown reference). Event Name: `generate_lead`.
3. **Triggers → New → Custom Event**, event name `contact_form_submit` — this is what the site's contact form already fires on success.
4. Set that trigger on the GA4 Event tag. Save.
5. Top right → **Submit → Publish** (GTM changes only go live after publishing).
6. In GA4 Admin → Events, mark `generate_lead` as a **key event** (conversion).

**Google Search Console:**
1. Add a **Domain property** (not URL-prefix) for the bare domain — covers http/https and www/non-www together. Verify via the DNS TXT record it gives you, added at your DNS provider.
2. Submit the sitemap: `/sitemap-index.xml`.
3. URL-inspect and request indexing for the homepage and each location/service page.

**Link GA4 to Search Console:** GA4 Admin → Product Links → Search Console Links → link this property, so query/impression data surfaces inside GA4 reports.

**Google Business Profile:** add/confirm your listing with NAP (name/address/phone) matching the site footer exactly.

### Ongoing SEO Utils Work (once the site has data)

These are not build steps. They need live data, so they start weeks after launch. Reuse the existing SEO Utils workspace from STEP 0 — do not create a new one.

**GSC Internal Links** (v2.5.0) — once Search Console has a few weeks of data, run Internal Links. It finds pages that need more inbound links and **suggests the exact sentence and anchor text** for each one. On a small local site this is the highest-leverage authority lever available: no outreach, no link building, just structure. The STEP 3.5 architecture already planned the linking, so treat this as the feedback loop that shows where the plan and reality diverged.

**GMB Rank Tracker — Progress view** (v2.3.0) — compares grid rankings across date ranges, so you see whether map-pack coverage is actually *expanding*, not just today's snapshot. Local service businesses live or die in the map pack.

Set the **baseline grid scan now**, at launch, even before the client's own Google Business Profile is established. The tracker reads the public map pack, so it works pointed at competitors, and without a run from launch day the Progress view has nothing to compare against later. If STEP 0's map pack fallback already ran a grid scan, that scan is the baseline.

**Keyword metrics source** — when re-checking volumes later, pass `source` explicitly and re-check anything returning null on `labs` against `google_ads` or `dfs_search_volume`. `labs` rejects Canadian province-level locations outright, and a null there is a source limitation, not a finding.

### Verify Your Site
- Lighthouse: target Performance >90, SEO 100, Accessibility >90 — check LCP ≤2.5s, INP ≤200ms, CLS <0.1 specifically
- Schema: Google Rich Results Test
- **Architecture drift:** every built route appears in `docs/site-architecture.csv` and vice versa. A page that exists but is not in the CSV, or a CSV row with no page, means the map and the site have diverged.
- **Agentic Browsing:** run Lighthouse > Agentic Browsing on the live domain and confirm `webmcp-schema-validity` and `llms-txt-presence` pass. To see the tools directly, open the contact page in Chrome Canary with `chrome://flags/#enable-webmcp-testing` enabled, then run `await navigator.modelContext.getTools()` in the DevTools console — expect all three tools there, two on pages without the contact form.
- Contact form: test end-to-end submission, then confirm in GTM Preview mode and GA4 Realtime that `contact_form_submit` / `generate_lead` actually fire
```
