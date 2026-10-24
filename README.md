# google reviews scraper api: Real Credit Costs, Plan Comparison, and a Working Python Setup for Pulling Google Maps Reviews

Most people searching for a google reviews scraper api want one of two things: a dedicated endpoint that returns review JSON with one request, or a scraping API that can get past Google's anti-bot walls while they extract reviews themselves. ScraperAPI fits the second category. It doesn't sell a standalone "Google Reviews endpoint" — instead it gives you rotating proxies, JavaScript rendering, CAPTCHA handling, and a structured Google SERP endpoint, and its official docs walk through scraping Google Maps business data with Python. That distinction matters a lot for pricing, because Google requests burn credits at a much higher rate than normal pages. This guide covers the real credit math, every current plan, how the extraction actually works, and where a dedicated-reviews competitor like SerpApi makes more sense.

> **Quick take:** If you're testing the waters, the 7-day free trial (5,000 credits, no credit card) gets you roughly 200 Google requests before you pay anything. For ongoing light use, Hobby at $49/month covers 4,000 Google requests. If you need structured review JSON with sorting and owner replies out of the box, SerpApi's dedicated endpoint does that — but it's a different pricing model. 👉 [Start with the free 5,000-credit trial and test Google Maps scraping yourself](https://www.scraperapi.com/signup?fp_ref=coupons)

## Two ways to get Google reviews data — and why the difference matters

There are two fundamentally different products hiding behind the phrase "Google reviews scraper API," and picking the wrong one wastes money.

**Route 1: A dedicated reviews API.** You send a place ID, the service returns an array of reviews as JSON — rating, text, date, reviewer info. SerpApi works this way with its Google Maps Reviews engine: you pass a `data_id` or `place_id`, and optionally sort with `newestFirst`, `ratingHigh`, or `ratingLow`, filter by topic or keyword, and get back structured review objects. One request, no parsing.

**Route 2: A general scraping API.** You point it at Google (Maps or Search), it handles proxies, rendering, and CAPTCHAs, and you parse the HTML — or use its structured endpoints where available. ScraperAPI is this type. Its documented structured endpoint covers Google search results (returning `organic_results`, `search_information`, and related searches as JSON or CSV), and for Maps reviews specifically, the official tutorial uses Selenium in proxy mode to render Maps pages and extract review counts and ratings from the page.

The practical consequence: Route 1 costs more per request typically but returns clean data immediately. Route 2 gives you flexibility across any site, but Google's anti-bot measures mean each request is expensive and, for reviews, you're writing the parser yourself. Neither is wrong — they just solve different problems.

## ScraperAPI plans: the complete current lineup

Here's every plan currently listed on the official pricing page, with monthly pricing and the yearly-billing discount (10% off, shown as the effective monthly rate):

| Plan | Monthly price | Yearly (per month) | Credits | Concurrency | Link |
| --- | --- | --- | --- | --- | --- |
| Free trial | $0 for 7 days | — | 5,000 | — | [Claim the 7-day trial](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Free plan | $0 | — | 1,000 | 5 threads | [Get 1,000 free credits](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Start Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Start Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Start Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Start Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Start Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Start Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom quote | — | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/?fp_ref=coupons) |

Every paid plan includes JavaScript rendering, premium residential and mobile IPs, JSON auto-parsing, CAPTCHA and anti-bot bypass, custom sessions and headers, automatic retries, unlimited bandwidth, a 99.9% uptime guarantee, structured data APIs, the DataPipeline, and Slack-based priority support. The tiers differ mainly in credits, concurrency, and two restrictions covered below.

## The credit math nobody leading the search results spells out

Here's the part most comparison posts gloss over: **a Google request costs 25 credits, not 1.** ScraperAPI prices pages by difficulty — a standard page is 1 credit, Amazon is 5, Google and Bing (including subdomains) are 25, LinkedIn is 30. Sites behind Cloudflare, Datadome, or PerimeterX add another 10 credits per request when bypassed.

Do the arithmetic before choosing a plan:

- **Free trial (5,000 credits):** about **200 Google requests** total. Enough to validate your parser, not enough for a real project.
- **Free plan (1,000 credits):** roughly **40 Google requests**, ever. Purely for plumbing tests.
- **Hobby ($49/month, 100,000 credits):** about **4,000 Google requests per month**.
- **Startup ($149/month, 1,000,000 credits):** about **40,000 Google requests per month**.

And it can get worse. Turning on JavaScript rendering adds 10 credits per request; premium proxies add another 10. A Google Maps page fetched with both enabled can burn around 45 credits per request, which drops Hobby's effective capacity to roughly 2,200 rendered Google requests a month. Since Maps pages are JavaScript-heavy, rendering is often unavoidable — which is exactly why the official tutorial uses the render flag in its proxy string.

Two tools keep this from running away from you. The **Domain Cost Estimator** in the dashboard shows the exact credit cost for any URL before you scrape it. And the `max_cost` parameter caps what any single request can consume, so one misconfigured scrape can't silently drain your balance.

This per-credit reality is the single biggest gap in most articles about scraping Google reviews: they compare sticker prices ($49 vs. competitors) without translating credits into actual review requests. 👉 [Check current plan prices and credit allowances on the official pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## How to actually pull reviews with ScraperAPI: the documented workflow

ScraperAPI's own tutorial for Google Maps reviews uses Python with Selenium in proxy mode. Here's the shape of it:

1. **Install the stack:** `pip install selenium selenium-wire webdriver-manager beautifulsoup4 lxml`. You'll need recent Python and Chrome. Selenium drives the browser, Selenium Wire handles the proxy integration, Webdriver Manager manages the Chrome driver, and BeautifulSoup with LXML parses the HTML.

2. **Route Selenium through ScraperAPI.** The proxy string looks like this, with your API key from the dashboard in place of the placeholder:

   
   http://scraperapi.render=true.country_code=us:{API_KEY}@proxy-server.scraperapi.com:8001
   

   Note the `render=true` flag — Maps needs it, and remember it's 10 extra credits per request.

3. **Run headless Chrome against Google Maps search.** The tutorial's example searches "pizza in new york." The script uses headless mode, waits up to 30 seconds for JavaScript, dismisses the cookies consent popup if it appears, and simulates pressing Page Down five times with 1-second pauses to load more results in the left-hand panel.

4. **Parse the results by CSS class.** The documented selectors are `hfpxzc` for business names (pulled from the `aria-label` attribute), `MW4etd` for ratings, `UY7F9` for review counts, and `Ahnjwc` for service options. Missing fields fall back to `N/A`.

5. **Export to CSV.** The example writes everything to `pizza_maps_places.csv` with columns for Place, Rating, Reviews, and Service options.

One honest caveat: these CSS class names are Google's current frontend markup, and Google changes its frontend. When a class changes, your parser breaks and needs updating. That's the trade-off of the general-API route versus a dedicated endpoint that shields you from markup changes.

If your target is Google **search** results rather than Maps reviews, ScraperAPI also has a structured Google SERP endpoint that returns JSON or CSV directly — no parsing needed. The sync version takes `api_key`, `query`, and optional `tld` (20 country domains from google.com to google.co.jp), `country_code`, `uule` for region targeting, `num`, `hl`, `gl`, and `start` for pagination. There's an async version too, at the async subdomain, that accepts a `queries` array for batch jobs, returns a task `id` and `statusUrl` per query, and supports webhook callbacks — the results land in the `response` key with the same structure as the sync endpoint.

## Hidden limits that only show up in the fine print

Several restrictions sit below the pricing-table surface but directly affect which plan makes sense:

- **Geo-targeting is locked on cheaper tiers.** Hobby and Startup only support US and EU geotargeting, and their analytics dashboards only show the last 30 days. Country-level geotargeting requires Business ($299) or above. If you need reviews from Google Maps as seen in Japan or Brazil, Hobby won't do it.
- **Concurrency overage means rejection, not queuing.** Exceeding your thread limit returns a 429 and the request is refused — no extra charge, but no automatic retry either. Budget your parallel workers accordingly.
- **Running out of credits pauses scraping by default.** You have options: opt-in auto-upgrade (Hobby through Professional) bumps you to the next plan automatically when credits run out; pay-as-you-go (Scaling and above) keeps scraping at a fixed per-credit rate up to a monthly cap you set; annual plans can renew early for a fresh batch of credits.
- **Yearly billing saves 10%.** On Hobby that's $44.10/month instead of $49 — about $59 saved per year. It's not dramatic, but it's real, and the pricing page displays both rates side by side.

## Where SerpApi does it better — and when that matters

Credit where it's due: for the narrow job of "give me reviews for this place," SerpApi's dedicated Google Maps Reviews engine is more direct. Its response includes fields ScraperAPI's DIY approach makes you build yourself: reviewer name, local guide status, contributor review and photo counts, star rating, likes, dates in both human and ISO format, review photos, and — notably — the **owner's reply** with its own date and text. It also supports translated snippets, topic filtering via `topic_id`, and text filtering via `query`.

Its pagination is quirky, though: the docs say `num` ranges from 1 to 20 with a default of 10, then also state it always returns 8 results on the initial page when no filters are set. Read the docs carefully before building pagination logic around it.

A quick side-by-side on the dimensions we could verify:

| Dimension | ScraperAPI | SerpApi (Google Maps Reviews) |
| --- | --- | --- |
| Return format | Raw HTML you parse (Maps); JSON/CSV via SERP endpoint (search) | Structured review JSON |
| Sorting reviews | DIY (parse and sort yourself) | `newestFirst`, `ratingHigh`, `ratingLow`, `qualityScore` |
| Owner replies | DIY extraction | Included in response |
| Scope | Any website, plus Google endpoints | Google-focused engines |
| Pricing model | Credits per request (Google = 25) | Per-request API plans |

If you also need to scrape Amazon, LinkedIn, or arbitrary sites behind anti-bot protection, ScraperAPI's general-purpose model covers all of it under one credit balance — that's the scenario where it pulls ahead. If reviews from Google Maps are genuinely all you need, a dedicated engine will get you there with less code.

One more budget option worth knowing: open-source Google Maps review scrapers exist (Bright Data publishes a free one aimed at small-scale projects), which cost nothing but require you to run and maintain the infrastructure — proxies, retries, and markup updates included.

## Which plan should you actually pick?

Match the plan to your request volume, not to the feature list — the features are identical across paid tiers anyway.

- **Just exploring or building a prototype:** the free trial (5,000 credits / 7 days) plus the free plan's 1,000 credits give you ~240 Google requests combined, zero dollars, no card required.
- **Monitoring a handful of businesses monthly:** Hobby at $49 covers ~4,000 Google requests. If rendering is on, closer to 2,000. Fine for tracking dozens of places.
- **Production monitoring, competitor tracking, or sentiment analysis across many locations:** Startup ($149, ~40,000 Google requests) or Business ($299) — the latter if you need country-level geotargeting outside the US and EU.
- **High-volume pipelines:** Scaling at $475 unlocks pay-as-you-go overage, which is the practical reason to jump past Business.

Before you commit to anything, two checks: run your target URL through the Domain Cost Estimator to confirm the real credit cost, and set `max_cost` on your requests so a single bad job can't drain a month's balance. 👉 [Compare every plan and its credit allowance before you commit](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## FAQ

**Does ScraperAPI have a dedicated Google Reviews API endpoint?**
No. Its structured endpoints cover Google search results (JSON/CSV with organic results), and Google Maps reviews are extracted via rendering + parsing, as shown in its official Selenium tutorial. If you want a one-call reviews endpoint with owner replies and sorting, that's the dedicated-engine model SerpApi offers.

**How many reviews can I pull with the free trial?**
The trial gives 5,000 credits over 7 days. At 25 credits per Google request, that's about 200 requests — and each Maps request with rendering enabled costs more (roughly 35–45 credits), so plan for fewer page loads if rendering is on.

**What happens when I exceed my plan's concurrency limit?**
ScraperAPI returns a 429 and rejects the request outright — no queuing, no overage fees. Lower your parallel workers and retry.

**Can I target Google results outside the US and EU?**
Not on Hobby or Startup; they're limited to US/EU geotargeting. Country-level targeting starts at Business ($299/month). The SERP endpoint also supports a `uule` parameter for finer region control on higher tiers.

**Do cheaper plans include JavaScript rendering and CAPTCHA bypass?**
Yes. Rendering, premium residential and mobile proxies, CAPTCHA handling, retries, and the 99.9% uptime guarantee are included on every paid plan — the tiers differ in credits, concurrency, and geo restrictions, not features.

**Bottom line for different situations:** if your job is "reviews for 20 restaurants, monthly," Hobby plus the free trial covers you for months. If it's "structured review JSON with owner replies across hundreds of locations, minimal code," the dedicated-endpoint route from SerpApi will save you engineering time even if it costs more per request. And if Google is just one of five sites you're scraping, one general-purpose credit pool beats juggling three vendors. 👉 [Test the workflow with ScraperAPI's free 5,000-credit trial first](https://www.scraperapi.com/signup?fp_ref=coupons)
