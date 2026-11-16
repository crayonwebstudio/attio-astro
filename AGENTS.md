# Attio Marketing Website, Implementation Specification for OpenCode

**Source:** https://attio.com/
**Research date:** 2026-11-13
**Rebuild target:** Astro + Tailwind CSS, static output, 1:1 visual recreation of the public marketing site
**Status:** Research complete for Phases 1-2 (sitemap, HTML, CSS tokens, copy). Phases 3-4 (live interaction, responsive) are PARTIAL: see §5, §6 and §10. Do not claim 1:1 until the live-browser checks in §10 are done.

## How to use this document

- **OBSERVED** = seen in the live HTML, sitemap, CSS, or text fetch on 2026-11-13. Treat as ground truth.
- **SEARCH-CLAIM** = third-party reconstruction, partly corroborated. Verify live before hardcoding.
- **NOT VERIFIED** = not visible in text/HTML fetch. Do not invent it. Check it live (Playwright / browser) before building that piece.
- **Do not** copy Attio logos, customer logos, photos, or the Tiempos Text / Inter Display woff2 files into production without a licensing decision. Download reference assets to `public/reference/` for comparison only, rehost or replace for launch. See §7.4.
- **Do not** access, scrape, or recreate authenticated areas (`app.attio.com`, `/go/` sign-in flows beyond linking to them). No form submissions were made during research.
- Full URL list (1,173 URLs): `attio-research/raw/urls.txt` and `raw/sitemap.xml` in this workspace. Interim notes: `attio-research/INTERIM-NOTES.md`.

## 0. Capability verdict (from the pre-research check)

All 9 capabilities the user asked about are available in this environment, with one setup step:

| # | Capability | Verdict |
|---|---|---|
| 1 | Browse and inspect the complete original site | YES. `browser.open` + `browser.browse` (cloud browser) + `curl`. Homepage fetched 200, 1,083,309 bytes. |
| 2 | Navigate every page and capture screenshots | YES via live browser / Playwright, once browsers are installed (see 7). Text fetch alone does not give layout. |
| 3 | Inspect layouts, CSS, fonts, animations | YES via downloaded CSS chunks + computed styles in a live browser. Font files and CSS variables verified (see §3). |
| 4 | Download and organise available assets | YES via `curl` / Storyblok CDN + `/_next/image` proxy. See §7. |
| 5 | Create and edit an Astro + Tailwind project | YES. Node v24.22.1, npm 10.9.4, Astro latest 7.4.11, Tailwind 4.3.5 on npm registry. Precedent in this workspace: Astro 4.16 projects (`crayon-astro`, `biite-astro`). |
| 6 | Run terminal commands and install dependencies | YES. `npm install playwright-core@1.65.0` tested OK in /tmp in 4s. 140 GB disk free. |
| 7 | Run Playwright for browser automation | YES after one install: Playwright itself is NOT preinstalled (no `ms-playwright` cache, no Python Playwright, no system Chromium except `/opt/meta-chromium`). Run `npm i -D playwright && npx playwright install chromium` in the project. |
| 8 | Compare original and local screenshots | YES. Python PIL 10.2.0 + numpy 1.26.4 available; `pixelmatch` 8.0.0 and `sharp` 0.35.5 available on npm. |
| 9 | Iteratively fix visual differences | YES. Astro dev/build + Playwright screenshots + diff loop, as in §9. |

## 1. Complete website sitemap

### 1.1 Totals (OBSERVED, sitemap.xml, 2026-11-13)

**Total: 1,173 URLs.** Grouped by first path segment:

| Segment | Count | Notes |
|---|---|---|
| `/apps/*` | 300 | Integration directory + index. Largest group. |
| `/help/*` | 252 | Help Center reference + Academy courses. |
| `/changelog/*` | 234 | Weekly changelog entries, mostly `/changelog/2026/*`. |
| `/p/*` + 9 locales (`/da/p`, `/de/p`, `/es/p`, `/fr/p`, `/it/p`, `/nl/p`, `/pl/p`, `/pt/p`, `/sv/p`) | 55 + 171 | Localised landing / paid pages. Same 19 slugs per locale. |
| `/blog/*` | 36 | Blog index + posts. |
| `/careers/*` | 22 | Index + 21 role pages + open-application. |
| `/templates/*` | 22 | Workflow templates (see §2.9 anomaly: `/templates` itself serves the Workflows page). |
| `/engineering/*` | 14 | Engineering blog index + 13 posts. |
| `/platform/*` | 14 | Core product pages (see §1.2). |
| `/customers/*` | 13 | Index + 12 stories. |
| `/legal/*` | 12 | Privacy, terms, DPA, cookies, etc. |
| `/agentic-readiness/*` | 4 | Audit + 3 result pages. |
| `/partners/*` | 4 | Index + app / creator / expert partners. |
| `/solutions/*` | 3 | startup-crm, deal-flow-management-software, startup-crm-draft. |
| `/tools/*` | 2 | automation-calculator, crm-roi-calculator. |
| Singletons | 13 | `/`, `/agentic-sales-report`, `/architecture`, `/brand`, `/contact/sales`, `/download`, `/experts`, `/london`, `/next-gen`, `/pricing`, `/privacy-preferences`, `/redefine`, `/refer`, `/setup`, `/startups` |

### 1.2 Core marketing pages (build first, §9)

- `/`, Homepage. Title: `Attio: The CRM for agentic revenue`.
- `/pricing`, Title: `Plans & Pricing | Attio`.
- `/platform/ai`, `/platform/ask`, `/platform/call-intelligence`, `/platform/custom-agents`, `/platform/data`, `/platform/developers`, `/platform/enrichment`, `/platform/mcp`, `/platform/productivity`, `/platform/reporting`, `/platform/sequences`, `/platform/universal-context`, `/platform/web-agent`, `/platform/workflows` (14).
- `/solutions/startup-crm`, `/solutions/deal-flow-management-software`, `/solutions/startup-crm-draft`.
- `/agentic-readiness`, `/agentic-readiness/foundations`, `/agentic-readiness/orchestration`, `/agentic-readiness/scaling`.
- `/agentic-sales-report`, `/architecture`, `/startups`, `/brand`, `/contact/sales`, `/download`, `/experts`, `/london`, `/next-gen`, `/redefine`, `/refer`, `/setup`, `/tools/automation-calculator`, `/tools/crm-roi-calculator`.
- `/customers` + 12 stories: baton, conduit, granola, june, marker-io, modal, passionfroot, railway, seedcamp, snackpass, taskrabbit, union-square-ventures.
- `/partners`, `/partners/app-partners`, `/partners/creator-partners`, `/partners/expert-partners`.
- `/blog` + posts (36 in sitemap; text extract returned 35 items incl. featured duplicates, back to Nov 2021). `/engineering/blog` + 13 posts.
- `/apps` + ~299 detail pages. `/templates/*` (22 workflow templates).
- `/help` (252), `/changelog` (234), `/careers` (22), `/legal` (12).

### 1.3 Pages NOT in the sitemap but linked from the site (OBSERVED in homepage HTML)

`/go/welcome/sign-in` (302 to `https://app.attio.com/welcome/sign-in`), `/context`, `/llms.txt`, `/favicon.ico`. The header/footer also link to help import pages with query strings, e.g. `/help/reference/imports-and-exports/migrate-data-from-another-crm?source=footer_salesforce` (same for hubspot, pipedrive, zoho, excel, csv).

### 1.4 External links (OBSERVED, homepage HTML)

- Docs: `https://docs.attio.com` (3x). Community: `https://www.attio.community/`. Atlas (hero eyebrow): `https://atlas.attio.com`. Trust: `https://trust.attio.com/`. Status: `https://status.attio.com/`.
- Socials (icon-only): `https://twitter.com/attio`, `https://linkedin.com/company/attio`, `https://dribbble.com/attio`, `https://www.youtube.com/@attioCRM`.
- App stores: Apple `https://apps.apple.com/gb/app/attio/id1511545395`, Google Play `https://play.google.com/store/apps/details?id=com.attio&gl=GB`, Chrome Web Store Gmail extension `https://chrome.google.com/webstore/detail/attio/legacbojjmajoedfolbjlekjjkepadph/`.
- SEARCH-CLAIM only (not in homepage HTML, do not present as homepage links): `app.attio.com/authorize`, `api.attio.com`, `mcp.attio.com/mcp`, `developers.attio.com`.

### 1.5 Scope decision for OpenCode

Do NOT hand-build 1,173 pages. Build: (a) all bespoke core pages by hand (§2), (b) collection-driven templates for `/apps/*`, `/blog/*`, `/customers/*`, `/changelog/*`, `/help/*`, `/careers/*`, `/templates/*`, `/legal/*` generated from data files, (c) localised `/p/*` and locale `/xx/p/*` pages only if the user confirms they are in scope; they are paid-landing variants, 19 slugs x 10 locales, and were not individually inspected. Flag them as Phase 3 scope, not Phase 1.

## 2. Page-by-page section inventory

### 2.1 Global chrome (every page, OBSERVED)

**Announcement bar:** `The new architecture for agentic revenue is here` linking to `/architecture`. Present on all pages inspected.

**Header (sticky, behaviour NOT VERIFIED, confirm live):** Logo (Attio wordmark, OBSERVED as `attio.svg` 18x18 mark asset; full wordmark SVG path NOT VERIFIED from homepage HTML alone). Nav: Platform (mega menu), Resources, Company, Pricing, Sign in (`/go/welcome/sign-in`), Start for free (`/go/welcome/sign-in`), Open Attio (`/go/`). Mobile nav: NOT VERIFIED, check live at 390px.

**Platform mega menu (OBSERVED, exact labels):**
- Context: Universal Context, `The AI context layer` (`/platform/universal-context`); Data Model, `Sync and structure your data` (`/platform/data`); Enrichment, `Auto-enrich every record` (`/platform/enrichment`); Call Intelligence, `Record and analyze meetings` (`/platform/call-intelligence`); Reporting & Forecasts, `Insights in real time` (`/platform/reporting`).
- Orchestration: Ask Attio, `Analyze and create with AI` (`/platform/ask`); Workflows, `Orchestrate any revenue motion` (`/platform/workflows`); Custom Agents, `Agents ready for any task` (`/platform/custom-agents`); Web Agent, `Research the web on autopilot` (`/platform/web-agent`); Sequences, `Personalized outreach at scale` (`/platform/sequences`).
- Ecosystem: Apps, `Discover the tools you need` (`/apps`); MCP, `Connect everywhere you work` (`/platform/mcp`); Developer Platform, `Extend and integrate with Attio` (`/platform/developers`).
- Attio for: Slack (`/apps/slack`), ChatGPT (`/apps/chatgpt`), Claude (`/apps/claude`).
- Get started: Attio 101 (`/help/reference/attio-101`), Startup program (`/startups`), Talk to sales (`/contact/sales`).
- Resources: Help Center (`/help`), Academy (`/help/academy`), Developer Docs (`https://docs.attio.com`), Attio Community (`https://www.attio.community/`), Partners, Technology (`/partners/app-partners`), Expert (`/partners/expert-partners`), Creator (`/partners/creator-partners`).
- Company: Changelog (`/changelog`), Announcements (`/blog`), Engineering blog (`/engineering/blog`), Careers (`/careers`), Developers (`/platform/developers`), Customers (`/customers`).

**Standard final CTA (OBSERVED, all pages):** H2 `Agentic revenue runs on Attio.` Buttons: Talk to sales (`/contact/sales`), Start for free (`/go/welcome/sign-in`), Send me a demo (destination NOT VERIFIED, no href captured in text fetch; confirm live). Exception: Workflows page variant `Ask more from CRM. Ask Attio.`

**Footer (OBSERVED, column headings are H2 in source):** Platform (Refer a team, New badge, Changelog, Gmail extension, iOS app, Android app); Company (Customers, Announcements, Engineering blog, New badge, Careers, Manifesto, Brand, Become a partner); Import from (Salesforce, Hubspot, Pipedrive, Zoho, Excel, CSV, all to help import reference pages); Attio for (Startups, Deal flow); Apps (Gmail, Outlook, Segment, Mailchimp, Slack, Outreach, Mixmax, Typeform, Zapier); Resources (Startup program, Help center, Developers, System status, Hire an expert, Downloads, Trust Center). Bottom bar: `© 2026 Attio Ltd. All rights reserved.` + Services Agreement (`/legal/services-agreement`), Privacy Policy (`/legal/privacy`), LLMs (`/llms.txt`), Cookie settings, Your Privacy Choices (`/privacy-preferences`). Exact per-item footer URLs: NOT VERIFIED except the external store links in §1.4; resolve each from the live DOM before building.

### 2.2 Homepage `/` (OBSERVED, most detail)

- Title `Attio: The CRM for agentic revenue`. Meta description: `The system for revenue teams to build pipeline, accelerate deals, and grow accounts around the clock.` OG image `https://a.storyblok.com/f/234930/1200x630/df480627b4/attio-og-image.jpg`. Twitter creator `@attio`.
- H1 (appears TWICE in raw HTML, identical text, likely desktop + mobile markup; rebuild with ONE H1): `Welcome to agentic revenue.`
- Hero eyebrow link: `GTM lessons from Elena Verna and more` to `https://atlas.attio.com`. Hero subcopy: `Attio is the platform that builds pipeline, advances deals, and grows accounts around the clock.` CTAs: Talk to sales, Start for free, Send me a demo.
- Hero visual (OBSERVED as text content, format NOT VERIFIED, video vs animated mock vs static): interactive product demo mock. Contents: `Product Demo w/ GreenLeaf`, 10:00, 28:14, Meeting, Transcript, Speakers Ashley and Sam (dialogue: spreadsheets unmanageable, move team by Monday, six seats on Pro plan, workflow automations, deal history migration, onboarding checklist). Second panel: Slack `# pipeline`, Marcus 10:24, `@Attio what deals should I focus on today?`, Attio reply naming GreenLeaf (verbal yes, no quote out) and Ramp (stalled in Legal 5d). Third panel: agent run `Score companies from recent activity and write the score back to Attio`, `Ran 3 commands: search-notes-by-metadata, search-emails-by-metadata, update-record`, `Ranked companies by engagement drop-off and pushed updated health scores to Attio`, model `Opus 4.8, 1M context`.
- Sections in order:
  1. Hero (above).
  2. Platform tabs. H2: `The intelligent system that never sleeps.` Subcopy: `Picks up leads at 2am. Catches renewals before they slip. Hands you the answer before you ask.` 5 tabs with anchors: Build pipeline (`#home-2026-generate-pipeline`), Convert leads (`#home-2026-convert-leads`), Run sales motions (`#home-2026-run-sales-motions`), Forecast revenue (`#home-2026-forecast-revenue`), Retain and expand (`#home-2026-retain-and-expand`). Tab content (OBSERVED, full copy in research notes):
     - Build pipeline: `Your team, amplified. Agents prospect and reach out when buyers are looking, building a pipeline of deals ready to win.` Product table `Companies to work`, `Ask Attio`, `All Companies`; companies OpenAI, Harvey, Browserbase, Cursor, Notion, Granola, Ramp, Elevenlabs, Linear; ICP Scores 98, 97, 96, 92, 91, 88; owners Isla Harrington, George Wilkes, Theo Marshall, Amelia Carter, Nathan Cole, Daniel Fraser, Maxwell Turner, Samuel Clarke. Research notes: `Opened 40+ GTM roles`, `Hired a new CRO`, `Raised $125M Series C`, etc. Email draft: `Send follow up email to Maya White`.
     - Convert leads: `Free your reps to sell.` / `Agents dig. You close.` / `Speed to lead, every time.` Workflow mock: Trigger Record created, Web agent Enrich lead, Custom agent Score lead for ICP fit, Route lead by segment True/False, Enroll in sequence Enterprise / SMB. Lead card: `ICP: 98`, `New Exec`, `Warm intro via a16z`, `$5B Series H`.
     - Run sales motions: `Run every motion, your way.` Deal table columns Discovery, company, date, USD value, owner, next step, age. Rows: Lumio AI Sep 30 2026 $20,800 Amelia Carter; Verda Labs Sep 10 2026 $36,400 George Hall; Cortexa Aug 28 2026 $54,000 Theo Marshall; Driftwave Jul 31 2026 $31,200 Nathan Cole; Synthred Aug 12 2026 $62,500 Isla Harrington; Pinevox Jul 20 2026 $11,000 Rachel Adams; Northpeak Jun 30 2026 $78,000 Samuel; Westwind Jul 8 2026 $26,400 Paul.
     - Forecast revenue: `Catch changes to the deal.` / `Skip the review scramble.` / `For the people who own the number.` Ask Attio query: `What is the estimate of this week's closed-won deal value`, `2 tools used`, `Attributes searched: 19 results`, `SQL query executed: 1452 rows in 6.2s`. Deals at Risk: Forage, Negotiation, $29,288 (close date passed, no signature); Helix, Procurement/Signing, $18,280 (unsigned since June 1).
     - Retain and expand: `Keep more. Grow more.` Charts: Book Size ARR, US / EMEA, 1.4m / 0.7m / 0.0m, Q1–Q4; Account Health Healthy / Watch / At risk, Count 100/80/60/40/20, Jan–May. Ask Attio top-5 at-risk accounts (only 2 visible in text): 1. Brightloop, $107,000 ARR, High (last two Stripe invoices failed, logins down 40% over 30 days); 2. Vela, $86,000 ARR, High (past due, renews June 30). `Spot the shift early.` Signals: Drew Austin changed roles Head of IT GreenLeaf; New funding round $250M; Joshua Reed upcoming renewal. `The move's ready. You make the call.` Email suggestions for Quanta (+42% usage) and Joshua renewal.
  3. Self-building. Eyebrow `Self-building`. H2: `Live from day one.` Subcopy: `Connect your inbox and calendar. Attio learns your business and builds itself around it, before your first agent even gets to work.` CTA Start for free. Product UI mock: Sarah Johnson compose email, Highlights, Summary (Head of IT, modernise data infrastructure, demo call Aug 29, 75% confidence), LinkedIn sarahjohnson, Demo Call Aug 29 10:40 AM, GreenLeaf Inc. San Francisco CA, Sales Outreach Step Automated email, Activity (Michael Chang meeting 6h ago, Sarah Johnson event 2 days ago, Michael Chang call 4 days ago).
  4. Universal Context. H2 (OBSERVED in HTML as `Universal Context`, and in text as `The only CRM with Universal Context`, confirm live which is the rendered H2): 5 items: `It logs itself. Emails, calls, product, billing, captured automatically.` / `Your tools finally talk. Granola, Slack, your whole stack, always in sync.` / `Gets to know you. So each play is sharper than the last.` / `Ask, and it's there. Any record, any answer, in a second.` / `No agent left guessing. Working from the same facts as your team.`
  5. Signals. H2: `All of the signals, none of the noise.` Subcopy: `Ready to act on.` CTA See more to `https://attio.com/context`. Tabs OBSERVED: Context, Agents + automations, Ecosystem. Copy: `Context. Emails, calls, records, product usage, connected tools. All live.` / `An always-on revenue engine. Prospect, follow-up, research, running 24/7.` / `Attio wherever you work. Browser, inbox, Slack, terminal, agents.` Tab behaviour NOT VERIFIED.
  6. Connectivity. Eyebrow `Connectivity`. H2: `Your whole stack, connected.` Subcopy: `Claude, Slack, Clay, Linear, Notion, and anything your team and agents run on.` CTA Explore the ecosystem to `/apps`. Second block: `SDK. API. MCP.` / `Build anything on Attio.` CTA View docs to `https://docs.attio.com/`. Integration logo strip: NOT VERIFIED in text (logos are images without text).
  7. Quote: `When first opened Attio, instantly got the feeling this was the next generation of CRM.`, Margaret Shen, Head of Business Operations, Modal.
  8. Build to scale. H2: `Run at any scale.` Subcopy: `Production-grade for your team and agents.` 4 stats OBSERVED: 30M MCP tool calls/month, 2.6B API requests/month, 76k active customer agents, 15M emails synced/day.
  9. Customer stories. Eyebrow `Customer stories`. H2: `Trusted by 30,000+ customers.` Subcopy: `From first agent to enterprise scale.` CTA Read more to `/customers`. 1 card OBSERVED in text: `Artificial Intelligence: 83% faster lead triage. How Granola turns product signals into revenue at scale` (`/customers/granola`). Other cards NOT VERIFIED (likely carousel).
  10. Changelog. Eyebrow `Changelog`. H2: `Better as you grow.` Subcopy: `New features every week to keep pace with you.` CTA View all to `/changelog`. 4 cards OBSERVED, all dated October 29, 2026: Activities, `Capture anything that moves a deal.` (`/changelog/2026/activities`); Workflows updates, `Put more of your process on autopilot.`; Your new inbox, `Stay on top of what needs you most.`; Redesigned record pages, `All your data right where you need it.`
  11. Newsletter. H2: `Stay ahead of GTM.` Subcopy: `Product updates in your inbox.` Button Subscribe. Input label/placeholder NOT VERIFIED.
  12. Final CTA + footer (§2.1).
- Customer logo strip: NOT VERIFIED in text fetch. Do not assume a logo row exists from text alone; confirm live.
- Homepage images (OBSERVED): only Storyblok OG image + `attio.svg` in raw HTML. Product UI in sections 2–5 appears to be built HTML/CSS mock UI, not image files, in the text extract. Image filenames for homepage sections: NOT VERIFIED, needs live DOM inspection.

### 2.3 Pricing `/pricing` (OBSERVED)

- Title `Plans & Pricing | Attio`. Meta: `Start for free. Whether you're a startup or enterprise, Attio is the CRM for agentic revenue at every stage of growth.`
- H1: `Revenue around the clock.` Subcopy: `Start for free with plans for every stage of growth, whether startup or enterprise.`
- Billing toggle: Monthly / Annual, `Save 20% with annual billing`. Annual is ~20% less on Plus and Pro; Ultra is billed annually only.
- Tiers (monthly first, annual second):
  - Free: `For individuals to get started with Attio.` Up to 3 users. CTA Start for free. Includes: Ask Attio, your personal agent; Self-builds from email and calendar; Automatic enrichment; Agents and workflows; 50,000 records and 3 objects. Price figure in hero NOT VERIFIED as a number; FAQ states `The Free plan costs nothing for up to three users.`
  - Plus: `For small teams building their first sales process.` $50/mo monthly, $40/mo annual, per workspace, includes 1 user. Add users $44/mo monthly, $35/mo annual each. Everything in Free, and: Email sharing and AI summaries; Send 1,000 emails/mo; Private lists; Basic permissions; 250,000 records and 5 objects.
  - Pro, badge `Most popular`: `For growing teams turning sales into a repeatable engine.` $125/mo monthly, $100/mo annual, per workspace, includes 1 user. Add users $99/mo monthly, $79/mo annual each. Everything in Plus, and: Custom objects; Call intelligence; Sequences; Team and user permissions; Advanced reporting; Track product usage and events; Data rules for reps and agents; 1M records and 12 objects.
  - Ultra: `For teams and orgs running multiple GTM motions at scale.` $1,000/mo per workspace, includes 5 users, annual billing. Add users $189/mo each. Everything in Pro, and: Unlimited objects; Query in SQL with API or MCP; Data connector; Unlimited data rules by team; Single sign-on (SSO); User provisioning (SCIM); Audit logs; Advanced controls; Forward-deployed onboarding.
- Compare plans table (groups OBSERVED; tick/cross matrix NOT VERIFIED, text loses column alignment): Platform features; Agents and AI; Automation; Context and data model; Email and calendar; Platform and developer; Security and admin; Support and services. Key values: Ask Attio credits Free 100, Plus 500, Pro 1,000, Ultra 2,500. Workspace credits Free 250, Plus 1,500, Pro 10,000, Ultra 20,000. Additional workspace credits from $85/mo monthly, $70/mo annual. Records Free 50,000, Plus 250,000, Pro 1,000,000, Ultra 3,000,000. Additional records Pro/Ultra $300/mo monthly, $240/mo annual per 1M. Objects: Pro 12, then $500/mo monthly, $400/mo annual each; Ultra Unlimited. File storage Free 50 GB, Plus 50 GB, Pro Unlimited, Ultra Unlimited. Company enrichment: Free Core profile, Plus + Headcount and ARR, Pro/Ultra + Funding. Reports: Plus 15, Pro 100, Ultra Unlimited. Report types: Free Insights, Plus + Funnels and segments, Pro/Ultra + Targets and trends. Rules: Pro 8 per object, Ultra Unlimited. Emails sent per user: Free 200/mo, Plus 1,000/mo, Pro/Ultra Unlimited. Bulk email: Free 10 at once, Plus/Pro/Ultra 50 at once. Users: Plus 10, Pro/Ultra No limit. Teams: Pro Up to 10, Ultra Unlimited. SSO, SCIM, Audit logs, Admin-only exports: Governance add-on on Pro, included on Ultra. Data connectors: Ultra 1 included.
- Credits packs (FAQ): 5,000 credits $70 annual / $85 monthly; 10,000 $120 / $150; 25,000 $260 / $330; 50,000 $475 / $595.
- Governance add-on: $30 per user/month, billed annually, adds SSO, SCIM, audit logs, advanced data controls on Pro. Included with Ultra.
- Startup discount: venture-backed startups up to 10 users get 80% off annual Pro + credits for year 1, 30% off year 2. Apply within 90 days of workspace creation.
- FAQ (13 questions, OBSERVED): Does Attio have a free plan? Can I try Attio before I pay? Is there a discount for paying annually? Which Attio plan is right for my team? Can I get SSO on the Pro plan? Do I need to talk to sales to buy Ultra? Can I change or cancel my plan at any time? How do credits work in Attio? How much do additional credits cost? What are my payment options? Does Attio offer discounts for startups? Can I use Attio with Claude, ChatGPT, or other AI tools? Can Attio help me migrate from another CRM? Other FAQ facts: 14-day free trial of Pro on every new workspace; MCP server on every plan; Ultra adds SQL via MCP; payments by card, Pro/Ultra also invoice/PO/ACH/check/wire; migration help on paid plans; Ultra includes forward-deployed engineer onboarding; plan changes in workspace settings, upgrades prorated, downgrades at next renewal.
- Other sections: same Margaret Shen quote as homepage; same final CTA; same footer. No separate testimonial grid extracted. Product screenshots on pricing: NOT VERIFIED.

### 2.4 Platform pages (OBSERVED, each: eyebrow, H1, hero CTAs Start for free / Send me a demo / Talk to sales unless noted)

- `/platform/ai`, Title `Embedded intelligence | Attio`. H1 `Every rep, infinitely multiplied.` Subcopy: `A team of agents on every deal. Qualifying, enriching, researching, and moving to close.` Sections: Ask Attio (4 numbered cards: 01 Start every day ready; 02 End calls with action; 03 Context on demand; 04 Customer insights, collected; images `prepare-me-for-the-day-v5.png`, `write-a-follow-up-email-pls-v5.png`, `what-do-you-know-about-the-founders-of-greenleaf-v6.png`, `what-was-their-product-feedback-v5.png`, all ~2096x1252); Quote Eric Weiss, GTM, Plain; Autonomous work in action (Create any workflow with AI; Research, enrich, route; Custom Agents for custom jobs); Smarter calls (3 cards + `trip-1.png`, `customer-signals.png`, `trilogy-2.png`); CRM, meet MCP (3 cards: Customer context, Tools for agents, Secure by design; icons `context.svg`, `agent.svg`, `safe.svg` 20x20); Attio plugs in (8 pills: Ask Attio, AI, Data model, Apps and integrations, Call Intelligence, Workflows, Sequences, Reporting; destinations NOT VERIFIED).
- `/platform/workflows`, Title `Agents and automations | Attio`. H1 `Revenue agents at your team's command.` Subcopy: `Orchestrate your best plays, on every account, over the deepest context layer in agentic CRM.` Hero workflow demo (Running/Completed): new deal created → Web Agent enrich → Custom Agent score/route → Enterprise target list → SMB target list. Sections: Cue the agents (15 distinct agent cards, likely scrolling marquee, count distinct names, do not hardcode marquee behaviour until live: Web, Scoring, Routing, Win/Loss, Onboarding, Coaching, Prospecting, Enrichment, Briefing, Follow-Up, Churn Risk, Expansion, Pipeline Review, Onboarding prep, Outbound); Hire the best rep (5 items: Enrichment, Web, Scoring, Routing, Briefing agents); Quote Ian Ahuja, Head of Sales and Partnerships, Lightdash; From idea to live agent (`Describe it. Test it. Ship it. Run it back.`; steps: Say it in a sentence, Test it block by block, further steps NOT VERIFIED); Powered by Universal Context (5 items: Always online, Made for agents, Patterns not flares, Wired into your stack, + intro); Built for the long run (3 cards + `workflow-webhook.png`, `workflow-builder.png`, `workflow-run-code.png`, 1280x1280); Trust by default (3 cards + `promo-locked-icon.svg` 18x18, `gallery.svg` 20x20, `promo-workflows-icon.svg` 14x14). Final CTA variant: `Ask more from CRM. Ask Attio.`
- `/platform/data`, Title `Data structure & syncing | Attio`. H1 `One context. Every team.` Subcopy: `The right call is always in front of you. Reps, agents, and reports run on one picture.` Hero: scrolling/marquee object cards (Partnership, Invoices, Company, Deal, Person, Marketplaces, Buyer, Seller, Transaction, Workspace, User; Standard/Custom labels; exact count/order NOT VERIFIED). Sections: Blazingly fast, amazingly flexible (4 numbered cards + `data-model.png` 2416x1008, `filter-transparent.png` 2000x2000, `attribute-transparent.png` 2200x1320, `associations.png` 2416x1008); Quote Margaret Shen, Modal; Context that stays current (3 cards + `apps.svg`, `activity.svg`, `status.svg` 18x18); Automatic enrichment (3 cards + `enrichment-new.png` 1472x1008, `relationship-intelligence-transparent.png` 1820x1820, `data-powered-workflows-transparent.png` 2000x2000); Unify your data sources (Data warehouses, Email and calendar sync, Sales engagement tools; full list NOT VERIFIED); From data to decisions, instantly (Sales pipeline, Revenue metrics, Product growth; tab behaviour NOT VERIFIED); Attio plugs in (same 8 pills).
- `/platform/reporting`, Title `Reporting | Attio`. H1 `The live story behind every number.` Subcopy: `Forecasts you can trust, pipeline health that updates itself, and insights only possible in Attio.` Hero dashboard mock values: Weighted pipeline $4.28M, Stuck deals 12 in Negotiation, Slipped deals $286K, Missing follow-ups 8 overdue, Win rate 34%, Sales cycle 41 days, Stage conversion 62%, Pipeline by region/segment $4.28M, Renewal risk 7 accounts, Churn exposure $184K, Revenue by plan $1.24M, Customer ARR $1.24M, Expansion pipeline $620K, New ARR $348K. Layout/chart types NOT VERIFIED. Sections: Reporting engine (4 cards + `reporting-refresh-map-soft-3x.png` 2418x948, `reporting-refresh-reports-transparent-3x.png` 1374x948, `reporting-refresh-collaboration-transparent-3x.png` 1374x948, `reporting-refresh-dashboard-transparent-3x.png` 2418x948); Open the number (Rewind to any day; Know if you're ahead; Down to the record; image `reporting-refresh-history-transparent-3x.png` 2088x2562); Attio does the math (6 metric cards + `reporting-example-N-rounded.png` 1478x800: Daily workspace creations, Average revenue per workspace, Number of users per workspace, Monthly recurring revenue, Customer acquisition cost, Lead channel distribution).
- `/platform/ask`, Title `Ask Attio | Attio`. H1 `Ask Attio.` (curl H1). Full section inventory NOT VERIFIED beyond title/H1; inspect live before building, do not copy `/platform/ai` content into it.
- Remaining platform pages, titles/H1s NOT individually inspected in this pass (curl only for the five above): `/platform/call-intelligence`, `/platform/custom-agents`, `/platform/developers`, `/platform/enrichment`, `/platform/mcp`, `/platform/productivity`, `/platform/sequences`, `/platform/universal-context`, `/platform/web-agent`. Build them from the same platform-page pattern (eyebrow + H1 + hero CTAs + feature cards + quote + plugs-in + final CTA + footer) ONLY after fetching each page's copy live. Do not invent their sections.
- `/platform/developers` is also the Developer Platform destination in the nav; it is distinct from `https://docs.attio.com`.

### 2.5 Solutions

- `/solutions/startup-crm`, Title is generic `Attio: The CRM for agentic revenue` (OBSERVED). Eyebrow `CRM for startups`. H1 `The real-time CRM for scaling startups.` Subcopy: `Infinitely flexible, Attio is built for every stage of the startup journey, from zero to IPO.` CTAs: Try for free (note: `Try for free` here, not `Start for free`), Send me a demo, Talk to sales. Hero image `startup-hero.png` 4528x4160. Sections: From zero to one, `Build your go-to-market engine.` (tabs Contacts, Pipelines, Activity, Reporting; tab content NOT VERIFIED); Pipeline management, `Workflows that work for you.` (image `startups-pipeline-cropped.png` 4528x4160; bullets: Create the perfect sales workflow, Powerful intel on every deal, Collaborative notes for everyone); And that's not all (3 cards + `startup-src-permission-control.png`, `startup-src-collaboration.png`, `startup-src-secure.png`, ~1158x400); Quote Taimur Abdaal, Co-Founder and CTO (company NOT VERIFIED), photo `taimur-abdaal.jpeg` 800x800.
- `/solutions/deal-flow-management-software`, Same generic title. Eyebrow `Deal flow CRM`. H1 `The CRM for custom deal flow.` Subcopy: `Fast, flexible and enriched with hundreds of data sources, Attio puts you in control of your pipeline.` Same CTAs. Hero image `vc-hero.png` 4528x4160. Sections: Organize, `World-class deal tracking.` (tabs Pipelines, Relationships, Recruiting; content NOT VERIFIED); Pipeline management, `Easily build the perfect pipeline.` (image `relationships.png` 4528x4160; bullets: Real-time data for real-time relationships, Custom data tracking); And that's not all (same 3 cards/images as startup-crm); Quote June Angelides MBE, Investor, photo `june-angelides.png` 216x216.
- `/solutions/startup-crm-draft`, In sitemap; content NOT inspected. Check live whether it is a draft duplicate or a redirect before building.

### 2.6 Agentic Readiness (interactive audit, OBSERVED)

- `/agentic-readiness`, Title `Agentic Sales Readiness Audit | Attio`. H1 `The Agentic Sales Readiness Audit`. Intro: `An agentic revenue motion means you stop trading speed, scale, and quality against each other...` 3 steps, all checkboxes (no other input observed): Step 1 Agentic foundations (5 checkboxes: data structured so an agent can query it; qualified lead defined without rep instinct; consistent structured signal capture; funnel stages defined so two reps move a deal the same way; named person accountable for agent output quality). Step 2 What agents take on (4: enrichments keep data current; missing/stale/wrong data surfaced; account/persona research pulled in by agents; post-call CRM updates drafted from the call). Step 3 How you'll know it's working (4: agents used consistently; output not edited/discarded at a rate suggesting low trust; automated job measurably improved a funnel metric; team moved to higher-judgment work). Scoring / routing logic NOT VERIFIED.
- Result pages: `/agentic-readiness/foundations` (H2 `Start with your data foundation`, 4 next steps), `/agentic-readiness/orchestration` (H2 `You're past readiness`, 4 next steps), `/agentic-readiness/scaling` (H2 `You're ready to build more robust agentic workflows`, 3 next steps). All 3 share: CTAs Explore Attio / Retake the audit; report form `Get the full report. The State of Agentic Sales, 2026, sent straight to your inbox.` Submit button, privacy note `By submitting, you agree to receive marketing emails from Attio...` Email input label/placeholder and form action NOT VERIFIED.
- `/agentic-sales-report`, In sitemap; content NOT inspected. Likely the report landing page; check live.

### 2.7 Customers

- `/customers`, Title `Customers | Attio`. H1 NOT VERIFIED in text extract (hero is likely image-led). Logo strip, 12 logos in order: Granola, Wispr Flow, Listen Labs, Obvious, Modal, Union Square Ventures, Replicate, Railway, Taskrabbit, Public, Wordsmith, Passionfroot (Storyblok SVGs, e.g. `granola.svg` 172x60). Featured grid, 7 cards: Granola (SaaS / $125M Series C, `83% faster lead triage...`), Modal (AI / $750M Series D, `Workflow automation at scale...`), Railway (SaaS / $100M Series B, `Cloud infrastructure magic...`), Passionfroot (Creator Economy / $15M Series A, `25% revenue boost...`), Union Square Ventures (Venture Capital, `From fragmented to unified...`), Snackpass (SaaS / $70M Series B, `Complete GTM visibility. Why Snackpass switched from Salesforce...`), Taskrabbit (SaaS, `4X faster deals...`). Each card: logo, industry/funding tag, metric headline, one-line summary, `Read case study`. Quote: Jamie Marshall, Co-founder and COO, Snackpass, `Attio is the best CRM you will ever use, and likely the last CRM you will ever need.` (appears twice in extract, carousel NOT VERIFIED). Closing CTA: `The CRM behind thousands of companies.` Buttons Start for free / Talk to sales.
- Story pages (12 slugs in sitemap; 3 inspected): Granola and Taskrabbit use Challenge/Solution template; Modal is long-form, no Challenge/Solution block. Common: intro paragraph, H2/H3 narrative sections, bold pull quotes, impact bullets, closing quote. Granola impact: Zero missed leads, 10X faster context, lead review 2h → 20 min/day, 100% GTM adoption. Taskrabbit impact: 90% adoption overnight (from sub-50%), 4x faster deal updates. Granola funding conflict: card says $125M Series C, story says $192M raised, flag for content decision, do not reconcile silently. Hero image, byline, closing CTA on story pages NOT VERIFIED.
- Sitemap stories not individually inspected: baton, conduit, june, marker-io, passionfroot, railway, seedcamp, snackpass, union-square-ventures. Build from the story template after fetching each.

### 2.8 Blog, Apps, Changelog, Help, Engineering

- `/blog`, Title `Insights & updates | Attio Blog`. H1 NOT VERIFIED (title suggests `Insights & updates`). Template listing, chronological newest first. 6 category tags observed: Company, How Attio uses Attio, Inside Attio, Product, Perspectives, Guides. Featured hero card + secondary featured cards + chronological list (exact hero count NOT VERIFIED; extract duplicates posts). Card pattern: title, date (e.g. Oct '26), category tag, one-sentence excerpt, author + role where present. First 10 in list order: `Five unsolved problems worth building a career on` (Oct '26, Perspectives); `Introducing the architecture for Agentic Revenue` (Oct '26, Company, Nicolas Sharp); `Inside the system that runs Attio's GTM` (Oct '26, How Attio uses Attio, Kyle Doherty and Zev Lebowitz); `What AI can't change: Behind Attio's latest brand campaign` (Aug '26, Inside Attio, Kate Donnellan); `Why Attio hires graduates` (Aug '26, Inside Attio); `Attio does the math` (Jul '26, Product); `Inconvo is joining Attio` (Jul '26, Company); `Workflows: orchestrate revenue agents, on every account` (Jun '26, Product); `Work anywhere with Attio` (Apr '26, Product); `Permissions in Attio` (Feb '26, Product). Further posts back to Nov '21 (`Announcing our $7.7m seed round`). Total count, filter bar, pagination/infinite scroll NOT VERIFIED. Category is a frontmatter field, not a separate page, unless live check proves category archives exist.
- `/apps`, Title `App Store | Attio`. H1 NOT VERIFIED in text extract. Directory, template-driven. 186 cards in text extract (lower bound, indices 0–185); sitemap lists ~299 detail pages, so live may paginate/lazy-load beyond the extract. Categories explicitly visible: Automations (75 cards in extract), Calling, Data and enrichment. Further categories NOT VERIFIED, do not list more as Attio facts. Card pattern: app name, `By <vendor>`, one-line description, icon SVG. Search/filter/sort NOT VERIFIED in text. Detail template (verified on `/apps/slack`, title `Slack Integration | Attio`): Overview, How it works (Ask Attio in Slack; Automate workflows), Configure (Workspace settings → Apps and integrations → search/connect Slack → choose workspace → Allow), Learn more link (external help docs).
- `/changelog`, 234 URLs in sitemap. Homepage shows 4 cards dated Oct 29, 2026 (see §2.2). Index + entry template not individually inspected; build one listing + one entry layout from the 4 known entries, then verify against live.
- `/help`, 252 URLs: `/help/academy` courses (Introduction, Reports, Sequences, Workflows, Product-led growth) + `/help/reference` (Attio 101, Attio AI, Automations, Email/Calendar, How-to guides, Imports/Exports, Industry guides, Managing your data, Productivity, Support, Tools/extensions, Workspace/Billing). Help pages were counted, not individually inspected. Build one help layout (sidebar nav + article) and generate from data.
- `/engineering/blog`, 14 URLs, 13 posts, titles in sitemap slugs (e.g. `ask-attio-a-technical-look-at-our-new-agent`, `building-the-attio-mcp-server`, `how-v8s-liftoff-compiler-broke-our-wasm-execution`). Content not inspected.

### 2.9 Templates anomaly (important)

- `/templates` does NOT serve a template directory. It serves the Workflows / Agents product page (title `Agents and automations | Attio`, hero `Hire the best rep. Again. Again. And again.`, subcopy `Agents in concert on every deal, beside your team.`). Sections: testimonial Ian Ahuja, Lightdash; `From idea to live agent, before your next meeting.`; `Built for the long run.` (Trigger from any signal / Loops, branches, fallbacks / Build your own blocks; images `workflow-webhook.png`, `workflow-builder.png`, `workflow-run-code.png`); `Trust by default.` (Assigned user permissions / Every run, on the record / Pause anything. Rerun anything.); closing `Ask more from CRM. Ask Attio.`
- Do NOT build a `/templates` gallery from this URL. Confirm HTTP status + final URL live (redirect vs repurposed route), then build `/templates` as that Workflows page. The 22 `/templates/*` workflow-template URLs in the sitemap are a separate collection; their content was not inspected.

### 2.10 Careers, Brand, Contact, Legal, Partners, Tools

- `/careers`, Title `Careers | Attio`. H1 NOT VERIFIED; first heading `Our mission is to build the platform for agentic revenue.` Sections: mission copy; photo collage (3 images: `team-member-4.jpeg` 400x400, `team-member-2.jpg` 160x160, `team-member-3.jpg` 160x160); team stat `+190` (label as headcount NOT VERIFIED); `Our values.` 4 cards: `Build with love.` / `Context over control.` / `Challenge ideas, not people.` / `Break it down, amp it up.`; `Right role, right time.` (`We'll share relevant opportunities when they open up.`, suggests roles load via embedded job board, not static copy); `Keep up to date.` (likely newsletter signup; fields NOT VERIFIED). Job list NOT in text; third-party claims of Ashby + 8–23 roles are NOT Attio-verified. Build the bespoke page + a marked placeholder for the job embed. Do not invent listings. Sitemap role pages (21) exist; fetch each before building detail pages.
- `/brand`, Title `Brand guidelines and press kit | Attio`. H1 NOT VERIFIED; first heading `About Attio`. Sections in order: About Attio (founded London 2019 by Nicolas Sharp and Alexander Christie, both link to LinkedIn); Naming (one word, uppercase A, pronounced `AT-ee-oh`, emphasis first syllable; capitalise product names Call Intelligence, Workflows, Ask Attio; features lowercase); Wordmark (most recognisable form, use by default; do not use wordmark without logo; do not typeset name in place of wordmark); Logo (logomark for avatars/app icons/favicons; keep clear space; do not warp/stretch/recolor/redraw); Logo pack (SVG + PNG, light/dark; download links NOT VERIFIED); Photos (team + out-of-home campaigns; founders photo `attio-founders.jpg` 7046x4699, captioned `Nicolas Sharp and Alexander Christie`); Press contact (details NOT VERIFIED).
- `/contact/sales`, Title `Talk to Sales | Attio`. H1 NOT VERIFIED. Only verified content in text: testimonial Miguel Pinho, Head of Technology, Seedcamp (`Right from the outset we've felt so supported...`). Form fields, headings, submit button, confirmation copy NOT VERIFIED (JS-rendered form). Build shell (testimonial sidebar + form area) only after live field verification. Do not invent fields.
- `/legal/*` (12), `/legal/privacy` exists, numbered policy (sections 1. Introduction, 2. Our role..., 3. What personal data... 3.1–3.4, 4. Google Services data, 5. Lawful bases, 6. How do we use...). Legal text intentionally NOT reproduced here. Build one legal layout shared across `/legal/*`; port structure only, do not paste legal copy without a legal review decision.
- `/partners`, `/partners/app-partners`, `/partners/creator-partners`, `/partners/expert-partners`, In sitemap + nav; content NOT individually inspected. Build after fetching each.
- `/tools/automation-calculator`, `/tools/crm-roi-calculator`, In sitemap; interactive calculators, content NOT inspected. Treat as Phase 3; do not fake calculator logic.
- `/architecture`, `/agentic-sales-report`, `/download`, `/experts`, `/london`, `/next-gen`, `/redefine`, `/refer`, `/setup`, `/startups`, `/startups` verified: title `Startup Program | Attio`, H1 `Startup program`. The rest: titles/H1s NOT inspected; fetch live before building. `/setup` and `/download` may be app-adjacent; check whether they are marketing pages or redirects.

## 3. Design system

### 3.1 Source stack (OBSERVED)

Next.js (App Router, `/_next/static/immutable/`, `/_next/image?url=...&w=3840&q=75`), Storyblok CMS assets (`a.storyblok.com/f/234930/`), Tailwind-style utility classes in HTML (`font-sans text-base text-primary-foreground antialiased`), CSS custom properties (see §3.3). Rebuild in Astro + Tailwind; do not port Next.js.

### 3.2 Typography (OBSERVED)

Preloaded woff2 (10 files, `/_next/static/immutable/media/`): Inter regular, medium, semibold, bold; Inter Display medium, semibold, bold; Tiempos Text regular + regular italic; 1 unlabelled file (`3fe682a82f50d426-s.p.200l3jur7oygh.woff2`). CSS also references `--font-mono: var(--font-jetbrains-mono)`.
Weights: normal 400, medium 500, semibold 600, bold 700.
Three-voice system (SEARCH-CLAIM, Refero, partly corroborated): Inter for UI, Inter Display for marketing headlines at weight 600 with tight tracking, Tiempos Text serif for testimonial / pull-quote moments. Do not hardcode which file maps to which heading until a live computed-style check is done.
Text scale tokens (OBSERVED in CSS):
- heading-lg: 3.5rem (56px), line-height 3.75rem, weight 600, letter-spacing -0.015em.
- heading-md: 2.5rem (40px), lh 2.75rem, w600, ls -0.01em.
- heading-sm: 2rem (32px), lh 2.25rem, w600, ls -0.01em.
- heading-xs: 1.75rem (28px), lh 2.125rem, w600, ls -0.01em.
- 2xl: 1.5rem, lh 1.875rem, w500, ls -0.01em. xl: 1.25rem, lh 1.625rem, w500. lg: 1.125rem, lh 1.5rem, w500. base: 1rem, lh 1.375rem, w500, ls -0.01em. sm: 0.875rem, lh 1.25rem, w500, ls -0.005em. xs: 0.75rem, lh 1.125rem, w500, ls 0.
- Other font sizes seen in CSS (context not resolved): 72px, 96px, 56px, 48px, 44px, 40px, 32px, 18px, 16px, 15px, 14px, 13px, 12px, 11px, 10px. Hero size is CONFLICTED: Refero claims 56px Inter Display; an unrelated GitHub DESIGN.md claims 80px hero, 40px H2. Neither is verified as the live hero size. Measure computed styles live (see §10, QA-1).
- Inter default weight 500 for much UI text (SEARCH-CLAIM). `ss03` alternates (SEARCH-CLAIM).

### 3.3 Colour (OBSERVED hex in CSS)

Primary accent: blue-500 `#266df0` (25x in main CSS chunk; links, focus rings, active states). blue-600 `#245bc2`, blue-400 `#709ff5`, blue-450 `#538bf3`, blue-300 `#94b9ff`, blue-200 `#c8dcff`, blue-100 `#e8f0ff`.
Ink / neutrals: black-100 `#1c1d1f` (primary ink, 22x), black-50 `#101010`, black-200 `#202124`, black-300 `#232529`, black-400 `#2e3238`, black-500 `#383e47`, black-600 `#505967`, black-700 `#6f7988`, black-800 `#8f99a8`, black-900 `#a4adba`.
Surfaces: white-100 `#ffffff`, white-200 `#fafafb`, white-300 `#f3f4f6`, white-400 `#edeff3`, white-500 `#e4e7ec`, white-600 `#dee2e7`, white-700 `#d3d8df`.
Semantic: green-500 `#0fc27b`, green-600 `#0db472`, red-500 `#ff5b59`, red-600 `#f65351`. Changelog tags: design `#00b5e6`, enhancement `#ab92f1`, feature `#72a4fb`.
Theme tokens (indirect via `--internal-color-*`): light, primary background white-100 `#fff`, primary foreground black-100 `#1c1d1f`; dark, primary background black-50 `#101010`, primary foreground white-100. Secondary background light white-200 / dark black-100. Surface light white-400 / dark black-400. Strokes: weak/subtle/strong/default map to white-400/500/800 and black-300/400/800 by theme. Focus ring: `#266df04d` (light), `#709ff599` (dark). Link foreground: blue-500 light / blue-400 dark. Caption foreground: black-600 light / black-900 dark.
CONFLICTING, do NOT use: GrainHQ claims Paper `#FFFFFF`, Ink `#1A1A1A`, Surface Cream `#F5F1EB`, Muted Warm Slate `#5A4A36`, Accent Violet `#7B5CFF`, Object Sans headlines, conflicts with current Inter/Tiempos evidence; treat as an outdated brand snapshot. Refero claims Obsidian `#101113` and Cobalt Soft `#538bf3`, `#538bf3` is corroborated as blue-450; `#101113` is NOT verified (dark bg observed is `#101010`).
Backgrounds / section banding: NOT fully resolved from CSS alone. Light canvas is white; dark sections use black-50/black-100. Confirm each section's background live before finalising tokens.

### 3.4 Layout, spacing, radius, shadow (PARTIAL)

- Breakpoints (OBSERVED media queries): 375, 576, 768, 992, 1024 (64rem), 1200, 1400, 1536px. Tailwind default breakpoints do not match 1:1; define custom breakpoints in Tailwind config from these values.
- Container widths: NOT VERIFIED from CSS alone. `max-width` values seen (unresolved context): 992, 768, 576, 375, 1536, 1200, 960, 926, 920, 896, 826, 720, 696, 570, 558, 520, 478, 464, 448, 396, 392, 381, 376, 360, 352, 345, 340px. Do not pick a container width from this list; measure the live header/hero/footer containers at 1440px (QA-2).
- Radius (OBSERVED): xs 0.125rem (2px), sm 0.25rem (4px), md 0.375rem (6px), lg 0.5rem (8px), xl 0.75rem (12px). Button radius NOT VERIFIED (GitHub claim 10px, do not use).
- Shadows, section padding, grid gaps: NOT VERIFIED. Measure live.
- Aspect ratios: `--aspect-golden: 1.618`, `--aspect-video: 16/9` (OBSERVED tokens).

### 3.5 Buttons, links, forms (OBSERVED / NOT VERIFIED)

- Primary CTA copy is consistently `Start for free` (15x on homepage) / `Try for free` (solutions heroes) / `Talk to sales` / `Send me a demo`. Destinations: Start/Try for free → `/go/welcome/sign-in`; Talk to sales → `/contact/sales`; Send me a demo → NOT VERIFIED.
- Button visual style (bg, text colour, radius, padding, hover): NOT VERIFIED from text. A GitHub reconstruction claims near-black `#202124` bg, `#F3F4F6` text, 10px radius, `#505967` border, treat as SEARCH-CLAIM, weak (`#202124` appears once in main CSS vs `#1c1d1f` 22x). Measure live (QA-3).
- Newsletter form (homepage): Subscribe button observed; input label/placeholder NOT VERIFIED. Agentic Readiness report form: email input + Submit, privacy note observed; input details NOT VERIFIED. Contact sales form: entirely NOT VERIFIED.

## 4. Component inventory

Build these as Astro components (names are for the rebuild, not Attio's):

- `BaseLayout.astro`, html lang en, meta/OG/Twitter, fonts, announcement bar, header, footer, final CTA slot.
- `AnnouncementBar`, `SiteHeader` (mega menu, mobile nav), `SiteFooter` (6 columns + bottom bar + socials), `FinalCta` (variant: default `Agentic revenue runs on Attio.` / workflows `Ask more from CRM. Ask Attio.`).
- `Hero` (eyebrow link, H1, subcopy, 3 CTAs), `HeroProductDemo` (homepage GreenLeaf/Slack/agent-run mock, build as HTML/CSS mock UI, not a static image, per §2.2 evidence).
- `PlatformTabs` (5 tabs, anchor ids `#home-2026-generate-pipeline` etc., with per-tab product-table / workflow / deal-table / forecast / retention mocks), `TabPanel`.
- `ProductTable` (Companies to work, ICP scores), `DealTable`, `WorkflowMock` (trigger → agents → route), `AskAttioMock` (query + tools used + SQL rows), `ChartMock` (Book Size ARR, Account Health, rebuild as styled divs/SVG, not screenshots, unless live check proves they are images).
- `FeatureGrid` (numbered 01–04 cards), `FeatureCard`, `AgentCard` (15 agents, Workflows), `ObjectCard` (Data marquee), `MetricStat` (30M / 2.6B / 76k / 15M), `QuoteBlock` (Margaret Shen, Eric Weiss, Ian Ahuja, Taimur Abdaal, June Angelides, Jamie Marshall, Miguel Pinho, Tiempos Text per §3.2 hypothesis).
- `LogoStrip` (12 customer logos), `CaseStudyCard` (7 featured), `CustomerStoryLayout` (optional Challenge/Solution header block).
- `PricingCard` (4 tiers + Most popular badge), `BillingToggle` (Monthly/Annual, Save 20%), `CompareTable` (8 groups, tick/cross, verify matrix live), `FaqAccordion` (13 Qs, behaviour NOT VERIFIED), `CreditsTable`.
- `BlogCard`, `BlogListing`, `AppCard`, `AppDetailLayout` (Overview / How it works / Configure), `ChangelogCard`, `HelpLayout` (sidebar + article), `LegalLayout` (numbered sections).
- `AuditChecklist` (Agentic Readiness, 3 steps, 13 checkboxes total) + `AuditResultLayout` (next-steps list + report form + Explore/Retake CTAs).
- `NewsletterForm`, `ValueCard` (careers, 4 values), `IntegrationPills` (Attio plugs in, 8 pills), `SignalTabs` (Context / Agents + automations / Ecosystem).

## 5. Animation and interaction inventory

### OBSERVED in CSS / HTML (tokens and structure)

- Animation tokens: `--animate-caret-blink` (0.5s), `--animate-collapsibleSlideDown/Up` (0.3s, cubic-bezier .65,0,.35,1), `--animate-dialog-scale-in/out` (0.2s), `--animate-navigation-enter-from-left/right` + `navigation-fade-in/out` (0.2s/0.1s, ease-in-out-cubic), `--animate-ping` (1s), `--animate-pulse` (2s), `--animate-search-shine` (1s), `--animate-slideDown/Up` (0.3s), `--animate-spin` (1s), `--animate-productivity-intro-height/width` (0.8s linear), `--ask-hero-delay` .1s/.2s/.3s.
- `prefers-reduced-motion: reduce / no-preference` media queries exist. `hover:hover` and `pointer:fine` queries exist. `forced-colors: active` query exists.
- Scroll-aware header: `<html>` carries `data-[scrolled=true]:bg-(--color-overscroll-bottom)` / `data-[scrolled=false]:bg-(--color-overscroll-top)` and `--color-overscroll-bottom/top` tokens. Exact sticky behaviour NOT VERIFIED.
- Homepage tab anchors (`#home-2026-*`) imply tab switching without page reload. Signals tabs (Context / Agents + automations / Ecosystem) and Solutions tabs (Contacts/Pipelines/Activity/Reporting; Pipelines/Relationships/Recruiting) are the same pattern.

### NOT VERIFIED, must check live before building (see §10)

Mega-menu open/close (hover vs click, delay, animation), mobile hamburger menu, sticky header on scroll, hero demo autoplay/video vs static, platform tab auto-advance, marquee motion (Workflows agent cards, Data object cards, direction, speed, pause on hover), scroll-reveal animations, carousel behaviour (customer stories, quotes, Snackpass quote appears twice, suggesting a carousel), changelog/newsletter interactions, pricing toggle animation, FAQ accordion animation, Agentic Readiness scoring/routing, calculators (`/tools/*`), app directory search/filter, blog filter/pagination, careers job-board embed.

## 6. Responsive behaviour

### OBSERVED (CSS breakpoints)

Breakpoints: 375, 576, 768, 992, 1024, 1200, 1400, 1536px. The 375px breakpoint and `max-width:375px` rules indicate a small-phone layout is explicitly designed, not just fluid scaling. `min-width:64rem` (1024px) is used as a major switch (likely desktop nav vs mobile nav).

### NOT VERIFIED, must check live (see §10)

- Desktop (1440px): container widths, nav vs hamburger switch point, hero layout (text + demo side-by-side vs stacked), tab layout, grid columns per section.
- Tablet (768–1024px): nav collapse point, grid collapses (4→2, 3→2, etc.), pricing cards stacking, tables → cards.
- Mobile (390px and 375px): hamburger menu contents, hero H1 size, CTA stacking, product mocks (hidden, simplified, or horizontally scrollable), footer columns → stacked, compare table → stacked/accordion.
- No responsive screenshots were taken in this research pass. Do not write responsive rules from the breakpoint list alone.

## 7. Asset inventory

### 7.1 How assets are served (OBSERVED)

- CMS: `https://a.storyblok.com/f/234930/<size>/<hash>/<file>` (images, SVGs, photos). OG image: `.../1200x630/df480627b4/attio-og-image.jpg`. Logo mark: `.../18x18/cfb7753a31/attio.svg`.
- Next.js image proxy: `/_next/image?url=%2F_next%2Fstatic%2Fimmutable%2Fmedia%2F<file>&w=...&q=75`. Originals live under `/_next/static/immutable/media/`.
- Fonts: `/_next/static/immutable/media/*.woff2` (10 files, §3.2). CSS: `/_next/static/immutable/chunks/*.css` (9 chunks on homepage; largest `2cvjm1x0_9-ke.css`, 502,742 bytes).

### 7.2 Named image assets verified in page text (download from these paths, rehost locally)

- Homepage hero/demo: no image filenames verified (mock UI, §2.2). Customer covers: `granola-cover.webp`, `modal-cover.webp` (plus `granola.png`, `avatar-1.jpg`, `avatar.png`, `header-avatar.png`, `hero-slack-avatar.jpg`, `owner-amelia.png`, `owner-daniel.png`, `owner-george.png`, `owner-isla.png`, `browserbase.png`, `dropbox.png`, `elevenlabs.png`, `harvey.png`, `linear.png`, `notion-square.png`, `openai.png`, `cursor.png`, `logo.png`, all under `/_next/static/immutable/media/`).
- Platform AI: `prepare-me-for-the-day-v5.png`, `write-a-follow-up-email-pls-v5.png`, `what-do-you-know-about-the-founders-of-greenleaf-v6.png`, `what-was-their-product-feedback-v5.png` (2096x1252–1256), `trip-1.png` 1348x1348, `customer-signals.png` 1760x1760, `trilogy-2.png` 2204x2204, icons `context.svg`, `agent.svg`, `safe.svg` 20x20.
- Workflows: `workflow-webhook.png`, `workflow-builder.png`, `workflow-run-code.png` 1280x1280; `promo-locked-icon.svg` 18x18, `gallery.svg` 20x20, `promo-workflows-icon.svg` 14x14.
- Data: `data-model.png` 2416x1008, `filter-transparent.png` 2000x2000, `attribute-transparent.png` 2200x1320, `associations.png` 2416x1008, `enrichment-new.png` 1472x1008, `relationship-intelligence-transparent.png` 1820x1820, `data-powered-workflows-transparent.png` 2000x2000; icons `apps.svg`, `activity.svg`, `status.svg` 18x18.
- Reporting: `reporting-refresh-map-soft-3x.png` 2418x948, `reporting-refresh-reports-transparent-3x.png` 1374x948, `reporting-refresh-collaboration-transparent-3x.png` 1374x948, `reporting-refresh-dashboard-transparent-3x.png` 2418x948, `reporting-refresh-history-transparent-3x.png` 2088x2562, `reporting-example-N-rounded.png` 1478x800 (N=1–6).
- Solutions: `startup-hero.png` 4528x4160, `vc-hero.png` 4528x4160, `startups-pipeline-cropped.png` 4528x4160, `relationships.png` 4528x4160, `startup-src-permission-control.png` 1158x400, `startup-src-collaboration.png` 1158x400, `startup-src-secure.png` 1159x400, `taimur-abdaal.jpeg` 800x800, `june-angelides.png` 216x216.
- Customers: `jamie-snackpass.png` 880x972, 12 logo SVGs (Storyblok 172x60 pattern), `granola-cover.webp`, `modal-cover.webp`.
- Careers: `team-member-4.jpeg` 400x400, `team-member-2.jpg` 160x160, `team-member-3.jpg` 160x160. Brand: `attio-founders.jpg` 7046x4699.
- Integration logos (homepage HTML preloads): Claude, Notion, Apollo, Linear, Slack, Lemlist, Fin, ChatGPT, Stripe, Granola, Pylon (SVG/PNG under `/_next/static/immutable/media/`).
- Full asset list per page was NOT extracted (only the filenames above, from text fetches). OpenCode must crawl each page's live DOM for `img`, `picture`, `source`, and CSS `background-image` URLs and record them in `src/data/assets.json` before downloading. Do not hotlink Attio/Storyblok in production.

### 7.3 Asset organisation (recommended)

`public/images/<page>/` for page art, `public/images/customers/` for logos/covers, `public/images/apps/` for app icons, `public/fonts/` for licensed fonts only, `src/data/*.json` for apps, blog, customers, changelog, pricing, help. Keep original filenames + a `source-url` field per asset for traceability.

### 7.4 Licensing / IP warning

Attio wordmark/logomark, customer logos, founder/team photos, and Tiempos Text are third-party IP. This spec is a research document for a visual recreation; it does not grant a licence. Before launch, the user must decide: use Attio's brand assets as reference only (replace with placeholders), seek permission, or limit the rebuild to an internal demo. Do not ship Attio's hashed woff2 files without checking their licence; Inter is open (Google Fonts / OFL), Inter Display and Tiempos Text are not freely redistributable by default, plan substitutes (Inter Tight for Inter Display, Source Serif 4 / Spectral for Tiempos) and flag the decision.

## 8. Recommended Astro component architecture

```
src/
  layouts/BaseLayout.astro      # html, head/SEO, fonts, header, footer, final CTA
  components/
    AnnouncementBar.astro  SiteHeader.astro  MegaMenu.astro  MobileNav.astro
    SiteFooter.astro  FinalCta.astro
    Hero.astro  HeroProductDemo.astro  PlatformTabs.astro  ProductTable.astro
    DealTable.astro  WorkflowMock.astro  AskAttioMock.astro  ChartMock.astro
    FeatureGrid.astro  AgentCard.astro  ObjectCard.astro  MetricStat.astro
    QuoteBlock.astro  LogoStrip.astro  CaseStudyCard.astro
    PricingCard.astro  BillingToggle.astro  CompareTable.astro  FaqAccordion.astro
    BlogCard.astro  AppCard.astro  ChangelogCard.astro  AuditChecklist.astro
    NewsletterForm.astro  IntegrationPills.astro  SignalTabs.astro
  pages/
    index.astro  pricing.astro  architecture.astro  brand.astro  startups.astro
    contact/sales.astro  agentic-readiness/index.astro
    agentic-readiness/foundations.astro  agentic-readiness/orchestration.astro
    agentic-readiness/scaling.astro
    platform/ai.astro  platform/ask.astro  platform/call-intelligence.astro
    platform/custom-agents.astro  platform/data.astro  platform/developers.astro
    platform/enrichment.astro  platform/mcp.astro  platform/productivity.astro
    platform/reporting.astro  platform/sequences.astro
    platform/universal-context.astro  platform/web-agent.astro  platform/workflows.astro
    solutions/startup-crm.astro  solutions/deal-flow-management-software.astro
    customers.astro  customers/[slug].astro
    blog.astro  blog/[slug].astro
    apps.astro  apps/[slug].astro
    templates.astro  templates/[slug].astro   # see §2.9 anomaly
    changelog.astro  changelog/[slug].astro
    help/[...slug].astro  careers.astro  careers/[slug].astro
    legal/[slug].astro  partners/[slug].astro
    tools/automation-calculator.astro  tools/crm-roi-calculator.astro
  content/  (or src/data/*.json)
    apps.json  blog.json  customers.json  changelog.json  help.json
    pricing.json  templates.json  careers.json
  styles/global.css   # Tailwind 4 @theme: fonts, colours (§3.3), breakpoints (§3.4), radius
public/
  images/...  fonts/...  favicon.ico
```

- Routing: Astro file-based routing matches Attio slugs 1:1 (no trailing slash). Use `getStaticPaths` for `[slug]` collections.
- SEO: one H1 per page (fix the homepage duplicate H1, §2.2), title/description/OG per page from §2, `lang="en"`, viewport `width=device-width, initial-scale=1`.
- Interactivity: Astro islands only where needed (tabs, billing toggle, FAQ accordion, audit checklist, mobile nav, mega menu). Everything else static HTML/CSS.
- Tailwind: define `@theme` tokens from §3 (colours, text scale, breakpoints 375/576/768/992/1024/1200/1400/1536, radius). Do not use Tailwind defaults for these values.
- No-index the rebuild until the user approves launch (Attio is a third-party brand; do not let a clone get indexed as Attio).

## 9. Development priorities

1. **P0, Shell + homepage:** BaseLayout, header/mega menu (desktop), footer, final CTA, announcement bar, homepage hero + 5 platform tabs + self-building + Universal Context + signals + connectivity + scale stats + customer stories + changelog + newsletter. This is the page the user will judge first.
2. **P0, Design tokens:** fonts, colours, text scale, breakpoints, radius in `global.css`, verified live (QA-1–3). Do not build pages before tokens are measured.
3. **P1, Pricing:** 4 tiers, billing toggle, compare table, FAQ, startup discount. Prices are fully verified (§2.3); the tick/cross matrix needs one live check.
4. **P1, Platform pages:** AI, Workflows, Data, Reporting first (full copy verified), then Ask + the 9 remaining platform pages after fetching each.
5. **P1, Solutions + Agentic Readiness:** startup-crm, deal-flow, audit + 3 result pages (scoring logic needs live verification).
6. **P2, Customers + Blog + Apps:** listing + detail templates, data files from sitemap (7 stories, 36 blog posts, 186+ apps). Generate, do not hand-build.
7. **P2, Templates anomaly, Careers, Brand, Contact:** resolve `/templates` redirect live; careers job embed placeholder; brand logo-pack downloads; contact form fields.
8. **P3, Help (252), Changelog (234), Legal (12), Partners, Tools, Engineering blog, localised `/p/*` (226):** collection templates + data import. Calculators and localised pages last; confirm scope with the user first.
9. **P3, Responsive + motion pass:** mobile nav, tab/marquee motion, scroll reveals, reduced-motion support, then the visual QA loop in §10.

## 10. Visual QA checklist

Do each check at 1440px desktop, 768px tablet, and 390px mobile, original vs local side by side (Playwright screenshots + pixel diff, §0 item 8). Mark Pass / Partial / Open per page; never mark Done on server checks alone.

- QA-1 Typography: measure live computed H1/H2/H3 sizes, weights, line-heights, letter-spacing on homepage, pricing, and one platform page. Resolve the 56px vs 80px hero conflict. Check Inter vs Inter Display vs Tiempos usage per element.
- QA-2 Layout: measure live container widths (header, hero, sections, footer) at 1440px; section padding; grid columns; compare-table layout.
- QA-3 Buttons/CTAs: measure primary/secondary button bg, text, radius, padding, hover state; confirm `Send me a demo` destination; confirm newsletter input placeholder.
- QA-4 Homepage: H1 is single (not duplicated); hero demo content matches §2.2 (GreenLeaf, Slack, agent run); all 5 tabs switch and show the correct mock data (companies, ICP scores, deal rows, forecast values, at-risk accounts); scale stats are 30M / 2.6B / 76k / 15M; changelog cards are the 4 dated Oct 29, 2026.
- QA-5 Pricing: toggle Monthly/Annual changes all prices (Plus $50/$40, Pro $125/$100, Ultra $1,000 annual); compare table tick/cross matches live per row; FAQ has all 13 questions; startup discount copy present.
- QA-6 Platform: AI 4 numbered cards + images; Workflows 15 distinct agent cards; Data object marquee; Reporting dashboard values ($4.28M etc.) and 6 math cards.
- QA-7 Solutions: startup-crm vs deal-flow heroes, tabs, quotes (Taimur Abdaal / June Angelides), and the shared 3-card `And that's not all` block.
- QA-8 Agentic Readiness: all 13 checkboxes present; scoring routes to the correct result page (verify logic live); report form present on all 3 result pages.
- QA-9 Customers/Blog/Apps: customers shows 12 logos + 7 cards in order with correct funding/metric headlines; Granola/Taskrabbit show Challenge then Solution, Modal does not; blog first 10 titles match §2.8 in order; apps groups under Automations, Calling, Data and enrichment at minimum.
- QA-10 Templates: `/templates` renders the Agents page, not a gallery (or the redirect is documented and matched).
- QA-11 Careers/Brand/Contact: careers shows all 4 values + `+190`; brand shows Naming, Wordmark, Logo, Logo pack, Photos, Press contact in order; contact shows the Seedcamp quote and the form only after field verification.
- QA-12 Interactions: mega menu (hover/click, all items in §2.1), mobile nav at 390px, sticky header, tab switching, marquee pause on hover, FAQ accordion, reduced-motion behaviour.
- QA-13 Responsive: no horizontal scrollbar at 390/768/1440px; no clipped text; footer stacks cleanly; pricing/compare table is usable on mobile.
- QA-14 SEO/Access: one H1 per page; title/description/OG per page; alt text on images (original has mostly empty alt, record the decision, do not silently copy empty alt); keyboard focus visible; contrast checked.
- QA-15 Links: every internal link resolves (no 404s across the 1,173-URL scope built); every external link in §1.4 points to the correct domain; sign-in links go to `/go/welcome/sign-in` (302 to app.attio.com), not to a fake login.

## Appendix A, What was NOT accessed / NOT verified

- No login, signup, or app workspace areas. No form submissions. No authenticated pages.
- Not verified in this pass: live computed styles (sizes, spacing, container widths, button visuals), mega-menu/mobile-nav behaviour, hero video vs mock, tab auto-advance, marquee motion, carousel behaviour, pricing compare tick/cross matrix, Free-plan hero price figure, Agentic Readiness scoring logic, contact/newsletter/audit form fields and actions, `/templates` HTTP status, full `/apps` category list and total, blog filter/pagination, careers job embed, brand logo-pack downloads, press contact details, and the individual copy of 9 platform pages, 9 customer stories, 22 templates, 234 changelog entries, 252 help pages, 21 careers pages, and all localised `/p/*` pages.
- Live browser inspection ran 2026-11-13 16:03 to 16:08 IST and its findings are in Appendix C. They supersede the text-fetch notes on header nav, hero variants, logo strip, sticky behaviour, scrollspy, accordion, tabs, and animations. Mobile 390px behaviour is still outstanding.

## Appendix C, Live browser addendum, 2026-11-13 16:03 to 16:08 IST (supersedes conflicting text-fetch notes)

Method: live desktop browser inspection of https://attio.com/, 10:33 to 10:37 UTC. No sign in, no Start for free click, no form submission. Full page PDF saved in the browser outputs folder (3.7 MB, browser-page-20261113T103744Z-0.pdf). Viewport note: the assignment asked for 1440x900, the window actually rendered at 1919x998, so layout below is as seen at about 1919px wide, NOT VERIFIED at exactly 1440x900.

### C.1 Header, corrected (supersedes section 2.1 nav list)

Top announcement banner is black, full width, dismissible with an X at right. Banner text varies per load, do not treat either as fixed: first load read "Attio is now Context Engine, Read more", after reload it read "The new architecture for agentic revenue is here, Read more". Dismissing it removes the banner and the header becomes white with black text and a thin bottom border. On the initial black header the logo is white.

Nav labels left to right: "Platform" (button), "Resources" (button), "Developers" (link), "Customers" (link), "Pricing" (link). Right side: "Sign in" (pill, outlined) and "Start for free" (pill, solid). There is NO top level "Company" nav item. Company appears only as a group inside the Resources menu.

Mega menus open on HOVER, no click needed, as large white rounded panels with a drop shadow, centred under the nav. The chevron rotates. Moving the pointer away or pressing Escape closes them. A blue keyboard focus outline was visible on Resources after Escape.

Platform menu: 3 white columns (Context, Orchestration, Ecosystem, same items as section 2.1) plus a grey right hand panel: "Attio for" (Slack, ChatGPT, Claude) and "Get started" (Attio 101, Startup program, Talk to sales).

Resources menu: narrower, 2 columns. "Support": Help Center, Academy, Developer Docs, Attio Community. "Partners": Technology partners, Expert partners, Creator partners. "Company": Changelog, Announcements, Engineering blog, Careers.

Sticky behaviour VERIFIED: both the black banner and the header stayed pinned at the top while scrolling through the whole page. After the banner is dismissed the white header still stays pinned. No shrink or change in header height was observed on scroll.

Cookie banner: bottom left, with "Continue" and "Reject" buttons (Reject was clicked in this check). Exact cookie wording NOT VERIFIED, it was not transcribed. Intercom chat button: black circular button, bottom right on every view ("Open Intercom Messenger").

### C.2 Hero, two variants observed (supersedes section 2.2 hero)

Variant A (first load): eyebrow pill "Attio is now Context Engine, Read more", H1 "The CRM for agentic revenue.", subcopy "Attio is the first CRM built for a world where humans and AI agents work together. It sees everything, remembers every interaction, and turns it into outcomes, so your team closes more and admins less."

Variant B (after reload and on all later loads): eyebrow pill "GTM lessons from Elena Verna and more", H1 "Welcome to agentic revenue.", subcopy "Attio is the platform that builds pipeline, advances deals, and grows accounts around the clock."

Page title on both loads: "Attio: The CRM for agentic revenue". Whether the two variants are an A/B test or time based is NOT VERIFIED. The rebuild must allow for both, default to Variant B (the one seen in the HTML fetch and on reload).

CTAs, both variants, centred under the subcopy: "Talk to sales" (white/light pill, black text, thin black border) and "Start for free" (solid black pill, white text). No small print line was visible under the hero CTAs. H1 is very large, centred, black. Subcopy is smaller blue-grey, centred, about 2 lines max.

Hero product visual: a large, light grey framed browser/app mock up (macOS traffic light dots top left, workspace "Basepoint" in the sidebar). It is NOT a video player, no video controls, play button or progress bar were observed, and it is NOT a static image. It is animated, auto playing HTML product UI that cycles through frames with no visible tabs or controls. Frames observed in sequence: (i) "Ask Attio", prompt "How do I win my deal with GreenLeaf?" typing character by character, then an AI answer with deal context (GreenLeaf Intro, Dec 12, 10:40 to 11:32 AM, 48m) and key opportunity signals (large startup deal, 52 seats, active migration intent, strong ICP fit, recent engagement), plus "Good morning, Alex", "Meetings Today, Oct 14", a "Product Demo w/ GreenLeaf" transcript (10:00 / 28:14, Ashley/Sam), a Slack "#pipeline" card, and a black agent card ("Find yesterday's demo call and create the right follow-up task", "Ran 3 commands, search-call-recordings-by-metadata, get-call-recording, create-task", "Opus 4.8, 1M context"); (ii) a "Business Metrics" dashboard, bar chart "Revenue growth by paid plan", donut "Closed-won deals", "Sales pipeline" funnel, "New signups" cohort chart; (iii) a Workflows screen ("Run #7 / Run history"). The frame had changed completely after a 10 second wait with no scrolling, so it auto advances and loops. Exact cycle order and timing NOT VERIFIED. On first load the hero text was initially blurred/faded and sharpened in (entrance fade/blur animation).

### C.3 Sections, live corrections

Logo strip VERIFIED (section 2.2 said NOT VERIFIED): heading "Trusted by 30,000+ companies", wordmark row Granola, Turbopuffer, Parallel, Modal, Wispr Flow, Railway, Listen, Taskrabbit, AIUC, Wordsmith. Granola, Modal, Railway and Taskrabbit are links to customer stories.

Platform: left sticky nav (5 items) plus 5 right hand articles, scrollspy VERIFIED. Clicking "Convert leads" scrolled smoothly to that article and made it active (black text plus blue left border, others grey). The active item also updated while scrolling (Build pipeline, Run sales motions, Retain and expand observed).

Self-building H2 as rendered live: "Live from day one. Connect your inbox and calendar. Attio learns your business and builds itself around you." plus a "Start for free" link. This differs slightly from the HTML text ("around it, before your first agent even gets to work"), use the live wording as primary and keep the HTML wording as a cross check.

Universal Context: full bleed BLACK section, fine vertical pinstripes, glowing orange to teal arc at the bottom. H2 "The only CRM with Universal Context", with a separate line above reading "The only CRM with". Five feature lines as in section 2.2.

Signals (black): H2 "All of the signals, none of the noise. Ready to act on." plus "See more" link. Accordion list "Context / Agents + automations / Ecosystem" beside an animated funnel/radar wireframe with moving dots. Clicking "Agents + automations" collapsed Context and showed "An always-on revenue engine. Prospect, follow-up, research, running 24/7." with a thin progress underline. Context is expanded by default.

Connectivity: H3 "Your whole stack, connected." plus "Explore the ecosystem". H2 "SDK. API. MCP. Build anything on Attio." plus "View docs". Connectivity row is a horizontal strip of app icons (Notion, Clay, Linear, Slack, Typeform, etc.) in dark rounded squares, cut off at both edges, consistent with an auto scrolling marquee, but motion and timing NOT VERIFIED.

Testimonial: same Margaret Shen quote as section 2.2, with the scroll word fill effect (see C.4).

Build to scale: stats count up on entering view, numbers were seen mid count at lower values (for example 28M / 2.2B / 58k / 10M) before settling at 30M / 2.6B / 76k / 15M.

Customer stories: tabs Granola / Railway / Modal / Taskrabbit. Default is Granola ("ARTIFICIAL INTELLIGENCE / 83% faster lead triage. How Granola turns product signals into revenue at scale."). Clicking Railway switched to "CLOUD COMPUTING / 100% adoption, no admin. How Railway built a lead engine the whole team runs on." with a team photo. Pressed tab state is a black pill with white text.

Changelog: 4 cards dated October 29, 2026, as in section 2.2, each card has a small product screenshot.

Newsletter: heading "Stay ahead of GTM.", subcopy "Product updates in your inbox.", email textbox placeholder "Your email address", button "Subscribe". Form was NOT submitted.

Final CTA: black, pinstriped like Universal Context. H2 "Agentic revenue runs on Attio." plus "Talk to sales" (dark grey pill) and "Start for free" (light grey pill), colours inverted versus the hero, on black.

Footer: black, full width. Large Attio logo top left, social icons bottom left (LinkedIn, X, Dribbble, YouTube), then 4 visual columns with headings grouped two per column (Platform plus Company, Import from plus Attio for, Apps, Resources, same link lists as section 2.1). Headings are small grey-blue, links light grey. "New" is a small blue pill. External links show an arrow. Bottom bar: "Copyright 2026 Attio Ltd. All rights reserved." plus "Services Agreement", "Privacy Policy", "LLMs", and a "Cookie settings" button.

### C.4 Animations and interactions, live verified (supersedes section 5 unknowns)

Hero fade/blur in on load. Scroll linked word fill on the Platform, Self-building, Universal Context and testimonial headings (grey-blue to black as you scroll). Stats count up on entering view. Sections and images lazy load with a blank/skeleton state before fading in. The hero Ask Attio prompt types itself and the hero frames auto advance. Platform left nav is a sticky scrollspy (see C.3). Signals is a click to expand accordion, not tabs. Customer stories are tabs, not a carousel. No video player anywhere. No other carousel arrows, dots or autoplay sliders were observed. Hover only colour changes on cards and buttons were not systematically measured, treat as NOT VERIFIED beyond the menu and nav states above.

### C.5 Still NOT VERIFIED after the live check

Exact 1440x900 rendering (live check was 1919x998). Mobile menu at 390px: the browser tools in that check had no viewport resize control, no hamburger menu was observed at desktop width, and no 390px behaviour could be tested. This needs a separate 390px wide Playwright check. Also NOT VERIFIED: exact cookie banner wording, hero cycle order and timing and whether the two H1 variants are A/B or time based, Connectivity marquee motion, detailed hover colours on changelog and newsletter cards.

## Appendix B, Research sources (2026-11-13)

- Sitemap: `https://attio.com/sitemap.xml` (1,173 URLs), `robots.txt` (Allow: /).
- Homepage HTML: `curl https://attio.com/` (200, 1,083,309 bytes); CSS chunk `2cvjm1x0_9-ke.css` (502,742 bytes).
- Key pages fetched by curl: `/pricing`, `/platform/ai`, `/platform/workflows`, `/platform/data`, `/platform/reporting`, `/platform/ask`, `/solutions/startup-crm`, `/solutions/deal-flow-management-software`, `/customers`, `/blog`, `/apps`, `/agentic-readiness`, `/startups`, `/brand`.
- Text fetches (`browser.open`) for homepage, pricing, platform pages, solutions, agentic-readiness + results, customers + 3 stories, blog, apps (+ `/apps/slack`), templates, careers, brand, contact/sales, legal/privacy.
- Third-party design reconstructions (SEARCH-CLAIM only): Refero `styles.refero.design/style/08c8700c-f278-42bc-812e-f60dc6ce996e`; GitHub `DESIGN.md` (attio reconstruction); GrainHQ `grainhq.app/brand/attio` (conflicting/outdated, do not use).
