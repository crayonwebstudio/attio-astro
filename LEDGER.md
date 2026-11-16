# Page ledger, Attio rebuild (per local_page_by_page_replication skill)

One row per page in scope order. A page is completed only after my own browser check of the built page passes its sections. Never claim the whole site complete until every row is completed. Shared header/footer differences stay open on every page they touch.

Scope: 1,173 URLs in urls.txt. P0 is the homepage only. P1/P2/P3 follow spec section 9.

| Page | Status | Sections checked | Open items |
|---|---|---|---|
| `/` (homepage) | live at https://attioastro.woodwolfdigital.com, browser verified 2026-11-16 | Hero (Variant B, one H1), logo strip, Platform scrollspy 5 articles, Self-building, Universal Context, Signals accordion, Connectivity, testimonial, Build to scale count-up, Customer stories tabs, Changelog, Newsletter, Final CTA, header mega menus, footer. Build passes. Playwright at 1440/768/390: 0px horizontal overflow, 1 H1, Railway tab switches, banner dismiss turns header white, mobile drawer opens, count-up reaches 30M/2.6B/76k/15M on scroll. Live check 2026-11-16: valid certificate, no warning, title, H1, header, hero buttons and Ask Attio panel all render. Deploy hntrigv0shoqln9opjka9bdl, repo crayonwebstudio/attio-astro commit 3dde9c8, Coolify app rpfy4mcbgyf2394opkz9itdd. | No live attio.com side-by-side pixel comparison done, comparison is against spec memory only, not pixel perfect. Hero cycle timing is about 10s per spec, exact live timing unverified. Computed sizes unverified against live (hero 56px mobile / 72px desktop is a spec-token choice). Logo strip and app icons are text wordmarks, not SVGs. Footer/header links to P1+ pages 404 until those pages are built. Google Fonts import fails in this sandbox browser with a tunnel error (curl to fonts.googleapis.com returns 200), so QA screenshots may show a fallback font; consider self-hosting Inter. Announcement bar wraps to two lines at 390px. Newsletter button changes to Subscribed with no backend, confirmation copy is not Attio copy. Deal table next step and age columns omitted, those values were not in the spec copy. |
| `/pricing` | pending | | |
| `/platform/ai` | pending | | |
| `/platform/workflows` | pending | | |
| `/platform/data` | pending | | |
| `/platform/reporting` | pending | | |
| `/platform/ask` | pending | | |
| `/solutions/startup-crm` | pending | | |
| `/solutions/deal-flow-management-software` | pending | | |
| `/agentic-readiness` (+ 3 results) | pending | | |
| `/customers` + 12 stories | pending | | |
| `/blog` + posts | pending | | |
| `/apps` + details | pending | | |
| `/templates` (Workflows page, see spec 2.9) | pending | | |
| `/careers`, `/brand`, `/contact/sales` | pending | | |
| `/help` (252), `/changelog` (234), `/legal` (12), `/partners`, `/tools`, `/engineering/blog`, localised `/p/*` | pending | | |

Build: `npm run build` in ~/workspace/attio-astro. Verify: Playwright screenshots at 1440, 768, 390 vs attio.com.
