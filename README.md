# Realtor.com Scraper: Where the Data Hides, What a Page Costs in Credits, and Which ScraperAPI Plan Fits Your Volume (Full Plan Table)

Realtor.com hands you the entire listing record as JSON inside the page, which makes it one of the easier portals to parse and one of the harder ones to reach. Three things decide whether a weekend script survives: the data sits in `script#__NEXT_DATA__` under `props.pageProps.initialReduxState`, the site runs an anti-scraping layer that expects a US exit IP, and search pagination stops at 10,000 results no matter how you slice the query. All three are fetching problems rather than parsing problems, which is why the build-or-buy decision usually comes down to whether you want to own a proxy and rendering stack.

If you only need a few thousand pages a month, buy the fetch layer and write the parser yourself. If you're monitoring a metro daily, do the credit math below before you pick a tier, because Realtor.com requests cost more than one credit each.

> **Most readers should start on Hobby: 100,000 credits for $49/mo.** A Realtor.com page that triggers bot-protection bypass costs 11 credits at minimum, 21 if you also need JavaScript rendering, so Hobby covers roughly 4,700 to 9,000 page loads a month. Move up only when your page count does. 👉 [Check current ScraperAPI plans and pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## How many ScraperAPI credits does one Realtor.com page cost?

ScraperAPI bills per request, and the per-request price depends on the domain and the features you switch on. A standard HTML page is 1 credit. Bot-protection bypass adds 10 credits and fires automatically when ScraperAPI has to get past Cloudflare, DataDome, PerimeterX/HUMAN or Akamai. JavaScript rendering adds another 10.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Bot-protection bypass | +10 (added to base cost) |
| JavaScript rendering (`render=true`) | +10 |
| Premium proxy (`premium=true`) | +10 |
| Ultra premium proxy | 30 |
| Ultra premium + JS rendering | 75 |
| Screenshot | +10 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing | 25 |
| LinkedIn | 30 |
| Geotargeting, sessions, device type, output format | 0 |

Realtor.com needs the bypass line. The open-source reference scraper for the site sets an anti-scraping flag in its base config and pins the request country to US, which tells you two things: the site is protected, and the IP has to look American. That puts a floor of 11 credits on every listing page you pull.

Rendering is a separate decision. Property detail pages carry the full record inside `__NEXT_DATA__`, which ships in the server HTML, so test those with `render` off. Search result pages are the ones where the same reference implementation enables JS rendering, which takes them to 21 credits. Turning rendering on everywhere doubles your bill for no gain.

Here's what that means at Hobby's rate of $49 for 100,000 credits, or $0.00049 per credit:

- **11 credits per page:** about 9,090 pages a month, roughly $0.005 each.
- **21 credits per page:** about 4,761 pages a month, roughly $0.010 each.

Those figures are arithmetic on published credit costs, not a quote for your URLs. The [Domain Cost Estimator](https://www.scraperapi.com/?fp_ref=coupons) in the dashboard gives the exact cost for a specific URL, and every API response reports what it actually cost in the `sa-credit-cost` header. Check one search URL and one detail URL there before you commit to a tier.

## All ScraperAPI plans, side by side

Every tier uses the same credit system, so moving up buys volume, concurrency and a few account-level features rather than a different product.

| Plan | Credits / month | Concurrent threads | Geotargeting & analytics | Price | Purchase |
| --- | --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | US & EU; 30-day analytics history | $49/mo | [Start with Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU; 30-day analytics history | Listed on the pricing page | [See Startup pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level; unlimited analytics | Listed on the pricing page | [See Business pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level; unlimited analytics; pay-as-you-go | Listed on the pricing page | [See Scaling pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level; unlimited analytics; pay-as-you-go | Listed on the pricing page | [See Professional pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level; unlimited analytics; pay-as-you-go | Listed on the pricing page | [See Advanced pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level; unlimited analytics; pay-as-you-go | Custom quote | [Talk to ScraperAPI about Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

Hobby's $49 monthly rate is the one figure quoted above; the remaining tiers are priced on the same page. Four billing rules matter more than the sticker prices:

- **Annual billing takes exactly 10% off** on every tier except Enterprise, which is custom.
- **Credits reset at renewal and don't roll over.** Unused allowance is lost, so size the plan to your real monthly volume rather than buying headroom you'll never spend.
- **Pay-as-you-go is a Scaling-and-above feature.** Hobby, Startup and Business return a 429 when credits run out; they don't quietly bill overage.
- **Auto-upgrade is opt-in** on Hobby through Professional, and unavailable on Advanced and Enterprise.

For Realtor.com specifically, the country-level geotargeting on Business and above buys you nothing. The site serves US listings, and Hobby's US & EU coverage already puts you in the right country. Don't jump to Business for the targeting; jump only if you need 3 million credits or analytics history older than 30 days.

## What writing the scraper yourself actually involves

Parsing is the cheap part. The record lives at `props.pageProps.initialReduxState`, and a jmespath query flattens it into listing ID, status, price, beds, baths, square footage, lot size, agent, office, tax history, price history, photos, phone numbers and a feature dict in about thirty lines. Search pages read `pageProps.properties` from the same `__NEXT_DATA__` blob, with a fallback to `script[type='application/ld+json']` when the JSON is missing.

The recurring costs sit elsewhere:

- **US proxies at volume.** Twenty concurrent threads against a protected portal means a residential or datacenter pool big enough to absorb blocks, plus rotation and retry logic.
- **The anti-scraping layer itself.** It changes; your bypass has to keep working after it does.
- **A headless browser pool** if you decide search pages need rendering. ScraperAPI caps rendered requests at a default burst of 10 per second, and that ceiling exists precisely because rendering is the expensive half of the pipeline.
- **Detail-page fan-out.** Tax history, school ratings, flood and wildfire scores, the full feature list and mortgage estimates only exist on the detail page. A 5,000-listing pull with details is 5,000 extra fetches.
- **Markup drift.** Search cards expose beds, baths, square footage and lot size as display strings that need regex-stripping to integers, and those test IDs get renamed.

None of that is impossible. It is a maintenance subscription you pay in engineer hours, and it's the reason a per-request API prices out favorably once your volume passes a few thousand pages.

## Fields and defaults that quietly decide your bill

This is the part the quick-start tutorials skip, and it's where most of the money and most of the bad data live.

**`includePropertyDetails` defaults to true in at least one hosted Realtor.com actor.** That means every listing in a search result opens a detail page, so a 10,000-listing pull is 20,000 requests. If price, beds, baths and square footage are all you need, switch it off.

**Run-size defaults are small.** Another actor defaults to `maxListings: 20` with `maxPages: 0` (unlimited). Leave it alone and you'll conclude the Austin market has twenty homes in it.

**Search stops at 10,000 listings per query.** Pagination simply ends, so a whole-city for-sale search truncates without an error. The workaround is to split the query by price band ($0–250k, $250k–500k, and so on) or by ZIP and neighborhood, then merge. One actor documents up to 200 listings per page, so check your page math against what you actually receive.

**Expired listings only get detected on a complete scan.** In incremental mode a row is classified NEW, UPDATED, UNCHANGED, REAPPEARED or EXPIRED against the previous run. EXPIRED tombstones are only produced when a run scans the search to its natural end. Runs that resume from a checkpoint, hit a max-pages or max-listings cap, or get blocked mid-way skip expiry detection entirely, which means a listing that vanished during a truncated run never shows up as gone.

**Unchanged and expired rows are billed when you ask for them.** `emitUnchanged` and `emitExpired` both default to off in that same actor, and both add billable rows.

**Dates are mixed formats.** List timestamps and scrape timestamps are UTC; `last_sold_date` and property-history dates come through as Realtor.com displays them. Compute days-on-market from the UTC fields and you're fine. Mix a displayed date into the same calculation and your DOM shifts by a day at the boundary.

**Card data and detail data disagree.** The search card gives you rounded display values; the detail JSON is authoritative. Merge with the detail record winning.

## Three listing types, three schemas

For-sale, rental and sold listings don't share a shape, and a pipeline built for one will drop fields on the others.

Rentals replace the single price with a min/max range and add a building name, per-species pet policy (cats, dogs) and a `units` array where each unit carries its own beds, baths, square footage, price and availability date. Sold listings add sold price and sold date, and leave days-on-market empty. Sold searches come back most-recent-first by default, which makes a "recently sold" pull usable as a comps feed without extra sorting.

Inputs are broader than city-and-state: ZIP codes, neighborhoods, counties, whole states, single street addresses, or a pre-filtered Realtor.com search URL pasted straight in.

## What the ready-made Realtor.com scrapers expose

Worth knowing if you're comparing hosted tools, and worth knowing if you're building: field coverage tracks whether detail pages get opened, not which brand you buy.

| Tool | Field coverage | Notable operating detail |
| --- | --- | --- |
| ScrapeHero's Realtor app | 21 fields; address, price, beds, baths, area, broker, coordinates, images, status, sold price and date | Bills 1 data credit per record; no per-credit price published, so it can't be converted to a per-listing cost |
| Scrapfly's open-source Python script | Tax history, property history with price per square foot, agent phones by type, features grouped by category | Needs Python 3.10, poetry and an API key; ships as reference code, not a managed service |
| Apify's parsebird actor | Widest list: multiple value estimates, schools with ratings and distances, flood/fire/heat/wind/air risk scores with FEMA zones, MLS name and number, mortgage and tax rate | `maxItems` defaults to 100 with detail pages on. Vendor claims 1,000 listings with full details in 36 seconds; that's the vendor's own figure, not an independent test |
| Apify's abotapi actor | Adds price-reduction amounts, flags, MLS IDs, up to 10 photos, open houses, rental unit arrays | Documents the incremental change types above, including the truncated-run expiry gap |

None of these publishes enough to compute a comparable per-listing price. ScrapeHero counts one data credit per record without a credit price; another actor's listing title mentions a dollar figure with no stated unit, so don't treat it as a per-1,000 rate.

## Compliance boundaries worth deciding before you scale

Listing data is public. Agent names, emails and phone numbers attached to those listings are personal data in several US frameworks, including CCPA, and handling them needs a lawful basis. Photos and broker marks can carry third-party rights, and Realtor.com's terms of use apply to your traffic regardless of how you fetch it. Internal research and commercial redistribution are very different positions; get your own legal read before you resell anything.

## Which plan for which job

**One-off research, a few hundred to about 5,000 pages:** Hobby. 100,000 credits at 11 to 21 credits per page covers it, and $49 is the whole cost. 👉 [Start on Hobby](https://www.scraperapi.com/?fp_ref=coupons)

**Daily or weekly monitoring of one metro:** multiply your page count by 11 or 21. A 500-listing check with detail pages, run daily, is roughly 15,000 credits a month at 11 credits each; a 10,000-listing sweep with details blows past Hobby in a single run. Startup's 1,000,000 credits and 50 threads is the next step, and Business adds 3,000,000 credits plus unlimited analytics history if you want a year of trend data in the dashboard.

**You can't afford a mid-month 429:** Scaling and above, where pay-as-you-go keeps a run alive past the allowance at a fixed per-credit rate, with an optional monthly spending cap.

**Over 20 million credits a month:** Enterprise, priced by quote.

Three checks before you pay: run the Domain Cost Estimator on one Realtor.com detail URL and one search URL so you know your real per-page cost; decide whether search pages truly need `render=true`, because it's 10 credits every time; and confirm your page count against the credit allowance rather than the thread count, since threads only change how fast you burn them.

👉 [Compare all ScraperAPI plans and pick your credit tier](https://www.scraperapi.com/pricing/?fp_ref=coupons)
