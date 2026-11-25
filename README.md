# Web Crawler API Costs and Limits: What Does a Crawl Really Cost, How Deep Will It Go by Default, and Which ScraperAPI Plan Fits? (Full Plan Table + Credit Multipliers)

ScraperAPI charges **1 credit for a standard page**, 5 for Amazon, 25 for Google and Bing, 30 for LinkedIn, and adds **10 credits** when a request needs Cloudflare, DataDome or PerimeterX bypassed. Its free plan gives 1,000 credits a month with 5 concurrent connections. Those figures decide your monthly bill far more than the name of the plan you pick, so this comparison is built around them: what each credit buys, how many pages a crawler discovers before you notice, and what happens at the moment your allowance runs out.

Most people comparing a web crawler API get stuck on the same three questions. How much does a page cost on the sites I actually need? Will the crawler find the pages I want without me listing them? And what does the bill look like in month three, after the introductory pricing is gone? The sections below answer each one with published numbers, including the defaults that competitor crawlers apply when you don't set anything.

> **Quick pick:** start on ScraperAPI's free plan to test coverage — 1,000 credits a month, 5 concurrent connections, 1 credit per standard page. The limits that push you to a paid tier are crawler depth capped at 1 and no scheduling, so any recurring or site-wide crawl needs a paid plan.
> 👉 [See the current ScraperAPI plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans at a glance

ScraperAPI sells each tier by the credit volume it includes, billed monthly or annually. The pricing page carries the current dollar amount for each tier; what the table below fixes is what each tier unlocks and how it behaves when the credits are gone.

The free plan is the only tier with published hard caps: **1,000 credits per month and 5 concurrent connections**, plus a crawler restricted to one-time runs at `max_depth: 1`.

| Plan | Crawler access | When credits run out | Billing | Plan page |
| --- | --- | --- | --- | --- |
| Free | One-time crawls only, depth capped at 1 | Requests rejected with 403 | — | [Free plan details](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | Full depth, hourly/daily/weekly/monthly schedules | Auto-upgrade to next tier (opt-in) | Monthly or annual | [Hobby pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Full depth, all schedule intervals | Auto-upgrade to next tier (opt-in) | Monthly or annual | [Startup pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | Full depth, all schedule intervals | Auto-upgrade to next tier (opt-in) | Monthly or annual | [Business pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Full depth, all schedule intervals | Auto-upgrade or pay-as-you-go | Monthly or annual | [Scaling pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Full depth, all schedule intervals | Auto-upgrade or pay-as-you-go | Monthly or annual | [Professional pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | Full depth, all schedule intervals | Pay-as-you-go up to your monthly cap | Monthly or annual | [Advanced pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Full depth, all schedule intervals | Pay-as-you-go up to your monthly cap | Monthly or annual | [Enterprise pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two of those columns carry more weight than they look. Auto-upgrade is opt-in, meaning your account silently moves to a bigger plan the moment credits hit zero unless you leave it off. Pay-as-you-go keeps you on the same tier at a fixed per-credit rate, with a spending cap you set yourself.

## What the credit multipliers do to your monthly page count

Your allowance converts into wildly different page counts depending on the target. Run the numbers before you compare tier prices, because the same 1,000 credits produce:

- **1,000 pages** on ordinary sites
- **200 pages** on Amazon (5 credits each)
- **40 pages** on Google or Bing (25 credits each)
- **33 pages** on LinkedIn (30 credits each)
- **about 90 pages** behind Cloudflare, DataDome or PerimeterX (1 credit plus 10 for the bypass)

A price-comparison pipeline that scrapes Google results burns through a free plan in 40 requests. A content pipeline hitting ordinary blogs gets a thousand. Same plan, same bill, 25x difference in output.

Two controls make this manageable. `max_cost` on a single scrape caps the credits one request can spend. The Domain Cost Estimator in the dashboard returns the exact cost for a specific URL before you commit a crawl to it, which matters most for mixed lists where you can't tell a 1-credit page from a 25-credit one by looking.

👉 [Check what your target domain costs per request](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Crawl defaults decide how many pages you pay for

A crawler's cost is set by how many pages it decides to visit, and most APIs make that decision with defaults you never see. Firecrawl's crawl endpoint is a useful reference point because its defaults are documented in detail:

- `limit` defaults to **10,000 pages** per crawl, so an unset crawl on a large site can consume a ten-thousand-page budget before you look at it.
- `crawlEntireDomain` defaults to `false`, which means the crawler only descends into child paths. Start at `/features/feature-1` and it will follow `/features/feature-1/tips` but skip `/pricing` and the homepage entirely.
- `sitemap` defaults to `include`, with `skip` (follow links only) and `only` (sitemap plus the start URL, no HTML link discovery) as alternatives. The mode alone can change your page count by an order of magnitude.
- `allowExternalLinks` defaults to `false`. Switch it on and external links are followed exactly one hop, while links pointing at an external site's homepage are dropped and logged as `EXTERNAL_LINK` errors.
- `ignoreRobotsTxt` and `robotsUserAgent` are Enterprise-only, so robots handling isn't adjustable on lower tiers.
- Path filters use RE2-style regex with no look-around or backreferences, and a pattern that fails to compile returns a 400. The starting URL is also checked against `includePaths`, so a mismatched pattern can return zero pages.

The same class of trap shows up elsewhere. crawlcrawl measures a run against a hard cap of 100,000 pages and a depth range of 1 to 50, with defaults of 1,000 pages, depth 5, a 250ms per-host delay and robots respected; its `store_html` flag decides whether you can re-render a page as markdown later, and turning it off permanently forfeits that. Its bulk endpoint accepts 1 to 100 URLs, clamps concurrency at 64, and counts every URL as one page **whether or not it was served from cache**. Crawl4AI's hosted tier publishes a free allowance of 1,000 credits a month at 10 requests a minute and 2 concurrent, moving to 120 a minute and 10 concurrent from $10 a month.

The lesson carries over to any crawler API: set the depth or budget explicitly, decide whether the sitemap or link-following drives discovery, and check whether filtered-out or failed pages still bill.

## How ScraperAPI's crawler is configured

ScraperAPI's Crawler takes a seed URL and follows links matching a regex you supply. Three fields are mandatory at setup: **link depth**, an **include path** pattern (`.*` for the whole site), and a **maximum cost in credits**. That last one is the guardrail worth understanding, because the job stops once the credit ceiling is reached rather than running until the depth limit.

Depth and budget solve different problems. Use `max_depth` when you know the site's shape and want a bounded section, three hops from a landing page for instance. Use `crawl_budget` when the site's size is unknown and you'd rather cap spend than page count. Setting both means the job stops at whichever limit arrives first, `max_depth: 3, crawl_budget: 500` being the safe-exploration pattern.

Output format is a required choice, and the options aren't interchangeable. Raw HTML is the default. JSON and CSV are available only for Amazon, Google, eBay, Redfin and Walmart. Markdown and plain text are there for LLM pipelines. If your extraction target is one of the five supported domains, structured output saves you a parsing layer; if not, you're parsing HTML yourself or asking for markdown.

Scheduling runs on `once`, `hourly`, `daily`, `weekly` or `monthly`, and paid plans are required for anything beyond a single run. A callback URL delivers results to your endpoint when the job finishes.

Error codes are specific enough to debug without support. A 400 means malformed input or bad regex syntax. 401 is the API key. **403 means credits are exhausted, or a free account asked for depth above 1.** 429 means too many concurrent jobs. 500 is transient and worth a retry with backoff.

👉 [Compare the crawler tiers side by side](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Total cost: what happens at the limit, and how annual billing behaves

Hitting your concurrency ceiling returns a 429 and the request is turned away. It is not queued and there's no surcharge for the attempt. Hitting your credit ceiling pauses scraping until the allowance renews, unless you've configured one of the escape hatches.

Auto-upgrade moves you to the next plan automatically when credits run out. It's opt-in and available on Hobby, Startup, Business, Scaling and Professional. It protects a pipeline from stalling but also means your monthly spend can climb a tier without you approving each step.

Pay-as-you-go keeps you on the current plan at a fixed per-credit rate, bounded by a monthly cap you set. It's available on Scaling, Professional, Advanced and Enterprise. For workloads with spiky months, this is the cheaper of the two, because the marginal credits are priced per credit rather than as a whole tier.

Annual plans carry one detail that doesn't appear in most comparisons: they can renew early to pull in a fresh batch of credits. If you're on annual billing and burn through your allowance in month eight, you don't have to wait for the anniversary or switch tiers to keep working.

The combinations that decide your first-year spend are therefore: monthly versus annual, auto-upgrade on or off, and pay-as-you-go with or without a cap. Set the cap deliberately. A pay-as-you-go plan without a spending limit behaves like auto-upgrade with less warning.

## Which setup fits your workload

**One page at a time.** Use the Standard API rather than the Crawler. Crawling a site to fetch a single URL spends credits on discovery you didn't need.

**A known list of URLs.** Use the Async API. ScraperAPI's own guidance is that it's cheaper and more predictable than the Crawler when you already have the URLs, which is the common case for product feeds and sitemap exports.

**An unknown site structure.** Use the Crawler with `crawl_budget` set. You're paying for discovery, so cap it.

**Recurring monitoring.** Use Crawler scheduling on a paid plan. Weekly or monthly intervals cover most price and content monitoring, and the callback URL saves you polling.

**Sites behind bot protection.** Budget 11 credits per page rather than 1. A 1,000-credit allowance covers roughly 90 such pages, which is the figure that should drive your tier choice if your targets sit behind Cloudflare or similar.

**Free plan with depth beyond 1.** Not possible. One-time crawls only.

## Self-hosted crawler or managed API

Crawl4AI is the strongest open-source option for LLM-oriented pipelines: Apache 2.0 licensed, installable via pip or a Docker image on port 11235, protected by a `CRAWL4AI_API_TOKEN` you generate, with a dashboard and playground for testing. It ships markdown filters for stripping boilerplate, CSS/XPath/regex extraction without an LLM, BFS/DFS/best-first deep crawling with crash recovery, and browser support across Chromium, Firefox and WebKit. Its compliance status is stated as SOC 2 Type I complete with Type II in progress.

Self-hosting wins on volume and on data that can't leave your infrastructure. It costs you a browser fleet, proxy management and the bot-wall fight, which is exactly what a managed API prices into its credits.

Browserless's Crawl API is another managed option worth knowing about, with two caveats stated in its own documentation: it is in **beta**, with parameters and response shapes subject to change, and it's available on Cloud plans only. It authenticates with a `token` query parameter rather than a bearer header, submits a job and returns a crawl ID to poll.

## FAQ

**Is the free plan enough to evaluate a crawler API?**
For a one-off crawl of a small site, yes: 1,000 credits, 5 concurrent connections, depth 1. It won't tell you how the crawler behaves three hops deep or on a schedule, so budget a paid month if those matter to your decision.

**Do I need to supply my own proxies?**
No. Proxy rotation and bot-protection bypass are handled on ScraperAPI's side, and the bot-protection bypass is what the extra 10 credits pay for.

**Can I get structured JSON instead of HTML?**
Only on the five supported domains (Amazon, Google, eBay, Redfin, Walmart) through the Crawler's JSON/CSV output. Everywhere else you take raw HTML or markdown and parse it yourself.

**What if a crawl overshoots my budget?**
It can't. Maximum cost in credits is a required field, and the job halts when the ceiling is reached.

**Can I schedule crawls on the free plan?**
No. Scheduling requires a paid plan, and free accounts are limited to one-time crawls at depth 1.

## Before you pick a tier

Match the tier to the credit cost of your targets, not to the page count you hope to hit. A pipeline on Google results needs 25 credits per page; the same budget on ordinary sites buys 25 times more. Work out your per-page cost with the Domain Cost Estimator first, multiply by your monthly page target, and only then compare tiers.

Second, decide what happens at the limit before you hit it. Auto-upgrade keeps a production pipeline alive and raises your bill without a prompt. Pay-as-you-go with a cap gives you a ceiling you chose. Pick one deliberately.

If your crawl is a one-off on a small site, the free plan covers it. If you're running a scheduled crawl across an unknown site structure, set `crawl_budget` and start on a tier where the credits cover the bypass multiplier.

👉 [Pick a ScraperAPI plan that matches your crawl budget](https://www.scraperapi.com/pricing?fp_ref=coupons)
