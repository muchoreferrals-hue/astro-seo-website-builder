# Astro SEO Website Builder

A Claude Code plugin that acts as a full web agency team. Run one command and get a complete, production-ready, SEO-optimized Astro website for any service-based business, deployed to Cloudflare.

## What it builds

- A pre-build market validation pass: Google Maps map-pack saturation check + SERP/keyword gap analysis for your target niche + city, before any code is written
- Full Astro project with Tailwind CSS, Cloudflare Workers adapter, sitemap, robots.txt (with a real local git repo)
- Every page: homepage, about, contact, service pages, location pages
- Schema markup: LocalBusiness, Service, FAQ, BreadcrumbList, WebSite
- GSAP animations: hero entrance, scroll reveals, stat counters, card staggering
- Contact form with Cloudflare Workers backend via Brevo
- AI-generated WebP images for every hero, OG image, and team photo
- A `CLAUDE.md` generated for every new site with its tech stack, image/WebP rule, Core Web Vitals targets, SEO Utils workspace, and market-validation findings
- A final AI-design-slop check via Impeccable before handoff
- `llms.txt` for AI crawler readability

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- Node.js 18+
- A Cloudflare account (free tier works)
- A [Brevo](https://www.brevo.com) account for contact form email delivery
- The `nano-banana-pro` skill active in Claude Code (for AI image generation)
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) connected (`claude mcp add playwright npx @playwright/mcp@latest`) — used for the pre-build Google Maps map-pack check
- An SEO Utils MCP connection (your own SEO Utils account/server) — used for SERP, keyword, and content/backlink gap analysis during market validation and ongoing rank tracking
- [Impeccable](https://impeccable.style/) installed globally (`npx impeccable install --scope=global --providers=claude`, or via `claude plugin marketplace add pbakaus/impeccable` + `claude plugin install impeccable`) — used as the final design-slop QA gate

## Install

One command:

```bash
curl -sL https://raw.githubusercontent.com/NicoSKOOL/astro-seo-website-builder/main/install.sh | bash
```

Restart Claude Code after installing.

## Usage

1. Create a new empty folder for your client's project
2. Open it in Claude Code
3. Run `/build-website`
4. Answer the niche + target city/cities question, and review the market validation report
5. Answer 9 onboarding questions about the business
6. Watch the agents build the full site

The plugin spawns four specialist agents:

| Agent | Role |
|-------|------|
| `niche-scout` | Pre-build market validation: map-pack saturation + SERP/keyword gap analysis per city |
| `tech-builder` | Builds all Astro files, components, schemas, config |
| `seo-writer` | Writes all content: titles, metas, body copy, FAQs, CTAs |
| `seo-auditor` | Runs a full PASS/FAIL audit before images are generated |

## Market validation (STEP 1)

Before onboarding, the plugin checks whether your target niche + city is worth building for:

1. **Map-pack check:** looks at the top 3 Google Maps results for your niche + city. If all 3 have more than 40 reviews, that's a saturation signal.
2. **SERP + keyword gap analysis:** regardless of the map-pack verdict, runs organic SERP research, keyword clustering, and competitor content/backlink gap analysis via SEO Utils to find uncontested opportunities.
3. **Verdict:** PROCEED, PROCEED WITH CAUTION, or RECONSIDER, with reasoning. This is a soft gate — on RECONSIDER you're asked whether to continue anyway, but it never blocks automatically.

This also sets up (or reuses) a dedicated SEO Utils workspace for the site, named `"{Niche} — {City}, {State}"`, which stays in use for the site's ongoing rank tracking and GMB monitoring after launch.

## What gets asked in onboarding

1. Business name, tagline, phone, email, address
2. Services offered (with descriptions and differentiators)
3. Locations/cities served
4. Value propositions and USPs
5. Brand tone (professional, friendly, authoritative, local)
6. Color palette (hex codes, screenshot, or "choose for me")
7. Social media handles
8. Existing reviews/testimonials
9. Hours of operation

## Review, git, and deploy

Everything the plugin builds stays local until you say otherwise — scaffolding initializes a real local git repo, but nothing is committed, pushed, or deployed automatically. Review the site (`npm run dev`) first.

When you're ready:

```bash
git add -A && git commit -m "Initial site build"
```

Then push to GitHub if you want a remote (create a repo via `gh repo create` or on github.com):

```bash
git remote add origin <repo-url>
git push -u origin main
```

Then deploy to Cloudflare Workers:

```bash
npx wrangler deploy
```

Then set the email secret (never put this in `wrangler.jsonc` — it stays out of source control this way):

```bash
npx wrangler secret put BREVO_API_KEY
```

Then in Cloudflare Dashboard:
- Point your domain
- Submit `/sitemap-index.xml` to Google Search Console

## Quality targets

After deploy, run Lighthouse. The plugin is built to hit:

- Performance: >90
- SEO: 100 (99%+ on the internal SEO audit checklist)
- Accessibility: >90
- Core Web Vitals: LCP ≤ 2.5s, INP ≤ 200ms, CLS < 0.1
- Every location page: at least 45% genuinely unique, local-area-specific content
- A clean (or knowingly-accepted) Impeccable design-slop audit

## Updating the plugin

Run the install script again to get the latest version:

```bash
curl -sL https://raw.githubusercontent.com/NicoSKOOL/astro-seo-website-builder/main/install.sh | bash
```

## Plugin structure

```
.claude/plugins/website-builder/
├── plugin.json
├── commands/
│   └── build-website.md      (orchestrator)
└── agents/
    ├── niche-scout.md
    ├── tech-builder.md
    ├── seo-writer.md
    └── seo-auditor.md
```

## License

MIT
