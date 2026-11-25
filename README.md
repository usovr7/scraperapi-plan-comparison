# Web data extraction: static, JS-rendered or bot-protected? How to tell what you're stuck on, what self-hosting really costs, and what ScraperAPI's credits buy (full plan table)

Most web data extraction projects fail at classification, not at code. A page that ships its data inside the HTML source comes out of an HTTP request plus a parser in an afternoon. The same page behind client-side rendering needs a browser that executes JavaScript. The same page behind Cloudflare needs that browser, residential IPs, retries and challenge handling. Running costs across those three cases differ by roughly an order of magnitude, and picking the tool before you know which case you're in is how teams end up babysitting a headless Chrome fleet for a site that never needed one.

ScraperAPI takes the managed route: one call with parameters for rendering, proxies, geotargeting and parsing, billed in credits rather than server hours. The free tier is **1,000 credits a month with 5 concurrent connections**, and the 5,000,000-credit Scaling plan is **$475/month, or $427.50/month billed annually ($5,130 a year)**. A standard page costs 1 credit while a JavaScript-rendered page costs 10, so the plan that fits follows from your targets more than from your page count.

> **Quick pick:** use the free tier to find out whether your real URLs come back at all, then size the paid plan in credits. Testing a few dozen of your own targets with rendering switched on costs nothing. 👉 [Compare every ScraperAPI plan and its current price](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Every ScraperAPI plan on one page

ScraperAPI sells seven paid tiers plus a free plan. They run on the same endpoint and differ in credit allowance, concurrency and geotargeting.

| Plan | Monthly API credits | Concurrent threads | Price and billing | Get started |
| --- | --- | --- | --- | --- |
| Free | 1,000 (5,000 for the first 7 days) | 5 | $0 | [Start on the free plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | 100,000 | 20 | Listed on the pricing page | [Hobby plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | Listed on the pricing page | [Startup plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Listed on the pricing page | [Business plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | **$475/month**, or $427.50/month billed annually ($5,130/year) | [Scaling plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Listed on the pricing page | [Professional plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Listed on the pricing page | [Advanced plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom quote | [Talk to sales about Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

Annual billing is exactly 10% off the monthly rate on every tier, which is where Scaling's $427.50 comes from. Credits reset at each renewal and unused ones don't roll over. Enterprise pricing is custom and applies across your usage, except requests to Ultra Premium, which carry their own minimum rate.

Three other parameters decide whether a plan is usable for your workload, and none of them sit on the headline pricing card:

| Plan | Geotargeting | Analytics history | What happens when credits run out |
| --- | --- | --- | --- |
| Hobby | US & EU | Last 30 days | Auto-upgrade to the next tier, or pause until renewal |
| Startup | US & EU | Last 30 days | Auto-upgrade, or pause until renewal |
| Business | Country-level | Unlimited | Auto-upgrade, or pause until renewal |
| Scaling | Country-level | Unlimited | Auto-upgrade, or pay-as-you-go at a fixed per-credit rate up to a cap you set |
| Professional | Country-level | Unlimited | Auto-upgrade, or pay-as-you-go up to your cap |
| Advanced | Country-level | Unlimited | Pay-as-you-go up to your cap; plan changes go through sales |
| Enterprise | Country-level | Unlimited | Pay-as-you-go up to your cap |

## Country-level geotargeting starts at Business, and that moves more buyers than the credit ladder

Hobby and Startup are restricted to US and EU geotargeting. If you need prices, stock or search results as a shopper in Tokyo, São Paulo or Sydney sees them, the cheapest tier that offers country-level targeting is Business, at 3,000,000 credits and 100 threads. That constraint alone pushes a lot of teams from a 100,000-credit plan to a 3,000,000-credit one, which is worth knowing before you budget around the entry price.

The same logic applies to concurrency. Twenty threads on Hobby means twenty requests in flight; if your job is a nightly crawl of 50,000 product pages, you can do the arithmetic on how long that takes. Scaling's 200 threads exist for exactly that reason.

## Which of the three page types are you actually stuck on?

You can usually tell from three observations, and each one points at a different tool. The table below is the one I'd tape to the wall before choosing anything.

| What you observe | What it means | Where to go |
| --- | --- | --- |
| A 200 response, but the parsed body is empty or a bare SPA shell with a JS bundle under it | The page builds its content client-side | A rendering layer: a managed scrape API with `render=true`, or a headless browser |
| The fields you want are missing from View Source but visible when you load the page in Chrome | Same cause, confirmed | Same as above |
| The response redirects to an anti-bot interstitial, or data sits behind a login you can't reach with a plain request | Session-gated or protected | Rendering plus cookies/sessions, then proxies if it persists |
| Data only appears after you click a tab, scroll an element into view or fill a form | Multi-step interaction | Full programmatic browser control (Puppeteer, Playwright), not a fetch |
| The page renders a different DOM under different conditions, so selectors break no matter how you write them | Dynamic layout you don't control | Browser control, or a vendor endpoint that keeps the selectors for you |
| Your result array comes back empty even though the page works fine in Chrome | Bot detection, not a parsing bug | Stealth and residential proxies before you rewrite anything |

The last row is the one people misdiagnose most often. An empty result set looks identical to a selector problem, but if the page renders normally in a browser and returns nothing through your script, you've been fingerprinted or blocked. Rewriting the parser will not fix it.

Two shortcuts can skip the ladder entirely when a site cooperates. Many publishers already emit structured data for SEO in JSON-LD, microdata and OpenGraph, and that data is free to harvest without rendering anything. The other shortcut is AI-assisted extraction for pages where maintaining selectors costs more than the page is worth.

## What running your own headless browser actually costs

The library is free. Everything around it isn't. Each Chrome session consumes hundreds of megabytes to a gigabyte under load, so a laptop or a single container falls over at a handful of parallel jobs. CI containers and Lambda need Chromium plus system libraries such as `libnss3`, `libatk-bridge` and `libgbm`, and that install diff grows into a forty-line Dockerfile that breaks on Alpine. Cookies and login state evaporate between runs because there's nowhere durable to keep a session. Cold starts add seconds to every job, which compounds fast on a cron schedule.

That's the cost structure a managed API is pricing against, and it explains why per-credit pricing looks expensive next to a free npm package until you count the engineering hours. ScraperAPI's answer to the browser problem is `render=true`, billed at 10 credits per request and available on every plan including Hobby, with concurrency measured in threads instead of processes.

## Credits are not requests, and that's where budgets go wrong

ScraperAPI bills by credit, and the credit cost of a request depends on the target. A standard page is 1 credit. Amazon, Walmart and eBay are 5. Google and Bing, including subdomains, are 25. LinkedIn is 30. On top of that, bot-protection bypass adds 10 credits when ScraperAPI activates bypass logic for Cloudflare, Cloudflare Turnstile, DataDome, PerimeterX/HUMAN or Akamai, and it's applied automatically rather than when you ask for it.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| E-commerce (Amazon, Walmart, eBay) | 5 |
| SERP (Google, Bing, incl. subdomains) | 25 |
| Social (LinkedIn) | 30 |
| Bot-protection bypass surcharge | +10 on the base cost |
| JS rendering (`render=true`) | 10 |
| Premium proxies (`premium=true`) | 10 |
| Screenshot (`screenshot=true`) | 10 |
| Premium proxies + JS rendering | 25 |
| Ultra Premium (`ultra_premium=true`) | 30 (paid plans only) |
| Ultra Premium + JS rendering | 75 (paid plans only) |
| Geotargeting, sessions, device type, output format, autoparse | Free |

Run the division on a 100,000-credit plan and the spread is stark: 100,000 standard pages, 20,000 Amazon product pages, 10,000 JS-rendered pages, 4,000 Google results pages, or about 1,333 Ultra Premium pages with rendering. Same credits, four very different jobs.

At the Scaling tier the per-unit maths is **$0.095 per 1,000 standard pages**, but $2.375 per 1,000 pages that need a premium proxy plus rendering. If you're evaluating a competitor's headline rate, check the unit. Oxylabs, for example, advertises structured output starting at $0.25 per 1,000 results, a per-result basis rather than a per-request one, so the two numbers can't be compared without knowing how many results a page yields. Oxylabs also publishes a 99.9% uptime figure, 177M+ proxies across 195 countries, and ISO/IEC 27001:2022 and SOC 2 Type II certification, which is the sort of thing to weigh if compliance review is part of your buying process. ScraperAPI's own plans list a 99.9% uptime guarantee, premium residential and mobile IPs, CAPTCHA handling and automatic retries on every tier.

Two habits keep this predictable. Every ScraperAPI response carries an `sa-credit-cost` header showing what that request actually cost, and the Domain Cost Estimator in the dashboard quotes any URL before you scrape it. You can also set `max_cost` per scrape so a single request can't run away with your allowance. 👉 [Start on the free tier and check a few target URLs against the estimator](https://www.scraperapi.com/?fp_ref=coupons)

## After the HTML arrives, half the job is still ahead

Fetching is the easy half. Real pages interleave the content you want with navigation menus, ads, recommendation modules, comments, embedded media, tables and captions, and stripping the boilerplate is only part of it. What counts as the right output depends on what consumes it: a retrieval or RAG pipeline wants clean article text, a multimodal grounding pipeline wants markdown with images and captions kept in place, and a tool-calling agent wants structured JSON with metadata, tables and links. That makes extraction a requirement-matching problem rather than a cleaning problem.

It also changes which tool makes sense. General-purpose extractors built on text density and link density deploy everywhere and break on publisher-specific templates, interactive pages, and targets outside their schema, such as metadata, in-page images and tables. Calling an LLM on every page handles weird layouts, at real cost, latency and non-determinism. The middle path is to use an LLM once to learn a publisher's recurring structure, then run a deterministic extractor against that learned configuration, which is what PACE (ProRata.ai, arXiv:2608.27466) does: training-time LLM analysis produces reusable configurations, and inference instantiates a fixed template without further model calls. Their experiments across article body, metadata and multimodal extraction report results close to hand-built publisher-specific parsers.

The practical version for a buying decision: if you only need clean text, a boilerplate remover gets you most of the way. If you need tables, captions, images and metadata as structured output, budget for either a parsing layer or per-page model calls, and know which one you're paying for. ScraperAPI's autoparse returns JSON and costs no extra credits, and output format is a free parameter alongside geotargeting and sessions.

## Matching a plan to a workload

Size the plan from credits after you've measured a sample, not from the page count in your requirements doc.

- **Checking whether targets are reachable at all:** free plan, 1,000 credits a month, 5 threads, 5,000 credits during the first 7 days.
- **Up to ~100,000 standard pages a month:** Hobby. If half your targets need rendering, that halves to roughly 50,000, since `render=true` costs 10 credits.
- **SERP tracking:** each Google or Bing query is 25 credits, so a 1,000,000-credit Startup plan covers about 40,000 queries a month.
- **Need prices or results outside the US and EU:** Business, the first tier with country-level geotargeting, at 3,000,000 credits and 100 threads.
- **Sustained 200 parallel requests:** Scaling at $475/month, which also unlocks pay-as-you-go.
- **300 to 500 threads, or overage headroom past the monthly cap:** Professional or Advanced.
- **Above 22,000,000 credits:** Enterprise, quoted per workload.

Two behaviours to confirm before you commit. When a plan's credits run out, requests are refused rather than billed as overage, and Hobby, Startup and Business accounts see that as a 429 until renewal unless auto-upgrade is switched on. Pay-as-you-go only exists from Scaling upward, so if you expect spiky months, the entry tiers aren't the place to sit. Cancellation is available from the dashboard at any time with no charge, and there's a **7-day no-questions-asked refund**, which is enough time to run a real sample at volume.

## Before you point a crawler at someone else's site

Terms and rate matter more than most tutorials admit. The United States has three main routes for a site owner to object: copyright in a compilation, the Computer Fraud and Abuse Act, and trespass to chattels, and outcomes depend on the facts of each case. Feist Publications v. Rural Telephone Service established that copying facts themselves is permitted, which is why scraped data often falls outside copyright even when the surrounding page doesn't.

Courts have gone both ways on access. In Craigslist v. 3Taps, a cease-and-desist letter followed by IP blocking was enough for Craigslist to pursue a CFAA claim after 3Taps scraped classified housing ads. Cvent Inc. v. Eventbrite Inc. (E.D. Va., 2010) held that browsewrap terms must be brought to a user's attention before they're enforceable. QVC's 2014 suit against Resultly in the Eastern District of Pennsylvania alleged 200 to 300 requests per minute, sometimes up to 36,000, which QVC said took its retail site down for two days. Southwest Airlines' claims against FareChase and Outtask, covering CFAA, trespass, misappropriation and breach of the user agreement, never reached the Supreme Court; FareChase was shut down by Yahoo! and Outtask was acquired by Concur.

Outside the US the picture is patchier. Indian courts haven't ruled clearly on scraping outside a few IP cases, but electronic contracts are enforceable there, so breaching terms that forbid scraping is a contractual matter, and the Information Technology Act 2000 penalises unauthorised access to or extraction of data from computer resources. The consistent advice across jurisdictions is unglamorous: read the site's terms of use and robots.txt before you scale, keep request rates within what a normal visitor would generate, and treat a cease-and-desist as a stop signal.

## Questions that come up before signing up

**Does ScraperAPI handle JavaScript-heavy pages?** Yes, through `render=true`, which costs 10 credits per request and is available on every plan including Hobby.

**What happens if I exceed my credits mid-month?** Requests stop rather than bill you extra. Hobby, Startup and Business return a 429 until renewal unless auto-upgrade is on; Scaling and above can use pay-as-you-go at a fixed per-credit rate against a monthly cap you set.

**Can I try it before paying?** The free plan gives 1,000 credits a month with 5 concurrent connections, plus 5,000 credits during the first 7 days after signup. Paid plans carry a 7-day refund window and can be cancelled at any time.

## What to pick

If you're still identifying your targets, stay on the free tier until you've run a real sample and read the `sa-credit-cost` header on a few responses. Once you know your credits per page, pick the tier whose allowance covers a typical month rather than a peak one, and use auto-upgrade or pay-as-you-go for the spikes. Anyone who needs country-level geotargeting should start the comparison at Business, because nothing below it will do the job regardless of credit headroom.

Last two checks before you enter card details: confirm your targets don't need clicking or form-filling, which is a browser-automation problem no fetch API solves, and confirm how many credits your worst-case page costs, since a Cloudflare-protected, JS-rendered page is not a 1-credit request. 👉 [See the current ScraperAPI plans and start with the free tier](https://www.scraperapi.com/pricing/?fp_ref=coupons)
