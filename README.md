# airbnb scraper api: is there an official Airbnb API, what does ScraperAPI return, and how many credits does a 1,000-listing job burn? (plans, fields, limits)

Airbnb does not issue public API keys. Its official API is offered only to selected partners and, per the company's own documentation as relayed by vendor guides, isn't accepting new requests, so anyone building a rental dataset today is scraping public pages with a proxy or scraping API. ScraperAPI can fetch those pages at scale. What it hands back is the page itself, HTML or markdown or text, and you turn that into a listings table. That single fact decides whether it's the right purchase for you, and the rest of this article is about the money side: what each plan includes, what a 1,000-listing run costs in credits, and where Airbnb's own paging limits will stop you no matter who you buy from.

> **If you want the short version:** start on the free plan (1,000 credits, 5 concurrent connections) or the 7-day trial that comes with 5,000 credits and asks for no card. Run ten real Airbnb URLs through it, read the per-URL credit cost in the dashboard's Domain Cost Estimator, then buy the tier that matches. 👉 [Check the plans and today's prices](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The plans ScraperAPI sells, and which one a scraping job actually needs

ScraperAPI's pricing page lists eight tiers: Free, Hobby, Startup, Business, Scaling, Professional, Advanced and Enterprise. Dollar prices and credit allowances per tier change, and the page is the only place showing what those are right now, so the table below sticks to what the documentation states as fact, plus a direct link to each purchase decision.

| Plan | What the documentation confirms | Who it fits | Start |
| --- | --- | --- | --- |
| **Free** | 1,000 free API credits, capped at 5 concurrent connections | Parsing a handful of Airbnb search pages to see what comes back | [ Start on the free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Hobby** | Auto-upgrade available (opt-in) | One developer running a small recurring job | [ See the Hobby tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Startup** | Auto-upgrade available | A two or three person team tracking a few cities | [ See the Startup tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Business** | Auto-upgrade available | Steady monthly volume where occasional spikes need headroom | [ See the Business tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Scaling** | Auto-upgrade **and** pay-as-you-go with a monthly cap you set | Workloads that fluctuate by season | [ See the Scaling tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Professional** | Auto-upgrade and pay-as-you-go | Continuous monitoring with budget control | [ See the Professional tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Advanced** | Pay-as-you-go only; auto-upgrade is **not** listed for this tier | High volume where you'd rather cap spending than jump tiers | [ See the Advanced tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| **Enterprise** | Pay-as-you-go; custom arrangements on the Airbnb solutions page mention 300+ concurrent threads, up to 50M credits, a Slack channel and premium support | Multi-million-request operations | [ Ask about the Enterprise tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two details in that table matter more than the tier names. First, auto-upgrade is opt-in: when credits run out, your account moves to the next plan automatically, which protects a running job from stalling but also means an unattended weekend can cost you. Second, Advanced doesn't appear in the auto-upgrade list, so on that tier an exhausted balance simply pauses scraping until renewal. Annual plans can renew early to top up credits without waiting for the month to roll over.

For everything else, billing runs on credits rather than requests. A standard page is 1 credit. Amazon costs 5, Google and Bing cost 25, LinkedIn costs 30, and any site behind Cloudflare, Datadome or PerimeterX adds 10 credits per request **when the bypass fires**. You can set a `max_cost` per scrape so no single URL can run past your ceiling.

## Why there is no airbnb scraper api from Airbnb itself

The official Airbnb API exists, but it's reserved for approved partner integrations. Apify's guide to scraping Airbnb, published in March 2025 and still the most explicit vendor statement on the subject, puts it plainly: the official API "is only offered to selected partners," Airbnb "don't currently accept new requests," and it "usually only provides the first 300 results on the website and limits the API to 1,000 results." Treat that second half as one vendor's description of what partners report, not as a published Airbnb specification, because Airbnb doesn't document those numbers publicly.

So the practical position for a developer in 2026 is that every rental dataset is built from public search and listing pages. Airbnb renders them in React, prices them dynamically against your dates and party size, and protects them with bot detection. That's the layer ScraperAPI sells into: proxies, rendering, retries, geo-targeting, so your request comes back as a page instead of a CAPTCHA.

## What ScraperAPI does with an Airbnb URL, and what it leaves to you

Send an Airbnb URL through the API with `render=true` and you get the fully rendered page. The Airbnb solutions page documents a pool of 200M+ IPs across residential, datacenter and mobile, `country_code` geo-targeting so you see the version of the page a shopper in that market sees, smart retry logic, and an Async API for batches that delivers results to a webhook with automatic retries on failures. Payload parameters include `output_format` for markdown or text instead of raw HTML, `premium` to restrict to residential and mobile proxies, `ultra_premium` for the harder targets, and `device_type` for desktop or mobile page versions.

What it doesn't do is return `nightlyPrice`, `ratingScore` or `reviewCount` as named fields. There's no Airbnb-specific parser in the payload, so the structuring is your job. Airbnb ships listing JSON inside the page (the `__NEXT_DATA__` script tag is where it normally lives), which is a far better parsing target than the visible DOM because it survives layout changes. If you'd rather buy the parsing too, that's a different product category: Apify's and Oxylabs' Airbnb tools return named fields straight out of the box, and the price difference reflects it.

The workflow is short. Build the Airbnb search URL with your dates, guests and currency, wrap it with `render=true` and your `country_code`, pull the embedded JSON, and write one row per listing keyed on the listing ID. Keep the ID as your dedupe key from day one, because splitting searches will hand you the same listing twice.

## Which fields you can pull, and the two limits that trip people up

Airbnb exposes more than the search cards suggest. Parsed from the listing page, the available data spans description, grouped amenities, host name and Superhost status, hosting tenure, capacity and bed configuration, room and property type, overall rating with review count, per-category sub-ratings for cleanliness, accuracy, check-in, communication, location and value, star distribution, discount percentage against the struck-through original, cancellation policy, house rules, check-in and check-out windows, safety items and photo captions. Pricing is the part people get wrong: a total only means something with the dates and party size attached, so store check-in, check-out, nights, adults and currency on the same row as the number.

Reviews and availability are separate modes with separate costs, and this is where most budget estimates go sideways.

| Data mode | What you get | Documented ceiling |
| --- | --- | --- |
| Search / listing rows | One row per listing, search-level fields | Paging stops well before the whole city |
| Listing detail | Description, amenities, host, house rules, sub-ratings | One extra fetch per listing |
| Reviews | One row per guest review, with reviewer name, location, date, text, host response, sortable by most recent, lowest or highest rated | Up to 2,000 reviews per listing in dedicated review mode, but only up to about 200 when reviews ride along on the listing row |
| Availability calendar | Day-by-day availability, minimum nights, occupancy per month | Up to 12 months ahead |

That 200 versus 2,000 gap is a real budgeting fork. If you're screening hosts for complaint patterns, 200 recent reviews on the listing row is usually enough and it's the cheaper path. If you're doing sentiment work on a single market leader, the dedicated review mode is the only way to get the full history.

Coordinates deserve a warning too. Airbnb shows an approximate pin before booking, and the scraped latitude and longitude inherit that fuzziness. Don't build a distance-to-beach feature on it.

## Credits vs per-row billing: what 1,000 listings really costs

ScraperAPI bills by page request. Apify bills by result row. Those two models produce very different invoices for the same dataset, so run the arithmetic before you pick.

On the ScraperAPI side, assume a results page carries roughly 20 listings. A 1,000-listing search-only job is therefore about 50 page requests. At 1 credit each that's 50 credits, comfortably inside the 1,000-credit free plan. If Airbnb's bot protection triggers the bypass on every request, the same 50 pages cost 50 × 11 = 550 credits, which still fits the free plan but eats half of it. Add detail pages and the shape changes completely: 1,000 listings means 1,000 extra requests, so 1,000 credits at base cost and around 11,000 if the bypass fires on each one. That's the number to model, not the search pages.

Apify's published samples, for comparison, on the lergassy Airbnb Scraper page dated from 14 November 2026: Listings mode from $0.40 per 1,000 rows and $0.80 per 1,000 with details on the Business plan, Reviews at $2.00 per 1,000 on Free and $1.40 on Business, calendar at $2.40 and $1.68. A separate Apify Airbnb Actor, parsebird, prices a listing event at $1.99 per 1,000 on Free and a detail event at an extra $2.49 per 1,000, which works out to about $4.48 per 1,000 for listings with detail pages. Oxylabs' Airbnb workflow runs through its Web Scraper API with a 7-day free trial, batch submission of up to 1,000 Airbnb URLs per request, and a trial rate limit that its own guide advises you keep to about 10 URLs or you'll get "Too many requests."

The takeaway is structural. Per-page billing rewards you for scraping search results only and parsing hard. Per-row billing rewards you for not requesting detail pages you won't read. Either way, deciding you don't need descriptions and amenities cuts the bill by an order of magnitude.

## Where the job breaks: search caps, 429s and paused credits

Airbnb stops returning results long before the city is exhausted. Apify's own Actor page, updated September 2026, says Airbnb stops a search after about 270 results and that its workaround is re-running the same search across nine price bands, which produced 400 listings for Seminyak in a single 36-second run. A competing Apify Actor describes paginating through Airbnb's own search API and splitting the price range into sub-searches whenever a location has more listings than one search can page through. Those are two vendor implementations of the same reality, not competing specs, and neither is an official Airbnb limit. The number to take away is that a single unfiltered search will not give you a whole market.

What that means for your pipeline: split by price band first, then by check-in window if a band still saturates, and dedupe on listing ID. Budget for the duplicates that splitting produces.

On the ScraperAPI side, three behaviours are worth knowing before you scale up. Exceeding your concurrency limit returns a 429 asking you to come back within your plan's limit; the request is turned away rather than queued, and you aren't charged for it. When credits run out, scraping pauses until they renew unless you've enabled auto-upgrade or pay-as-you-go. And the free plan's five concurrent connections is a hard ceiling on parallelism, which matters if you're testing an Async API batch and wondering why it feels slow.

## Is scraping Airbnb legal?

Publicly visible data is generally scrapable, but "generally" is doing a lot of work in that sentence. Personal data rules apply: GDPR in Europe and CCPA in California cover names, photos, profile links and locations, and Airbnb review records contain all four. The EU's DSM Directive and US fair use doctrine shape how much listing text and photography you can reproduce. Host-written descriptions and guest photos are copyrighted works.

Practical rules that keep most research projects out of trouble: store reviewer identity only if you need it, aggregate before you publish, never republish listing photos or descriptions verbatim, and check Airbnb's terms for the jurisdiction you're operating in. This is general orientation, not legal advice, and the compliance bar rises sharply the moment you resell the data as a product.

## FAQ

**Can I get an Airbnb API key?** No. Access is limited to approved partner integrations and new requests aren't being taken. Every commercial "Airbnb API" you'll find is a scraper with a friendly name.

**Does ScraperAPI return JSON for Airbnb listings?** It can return markdown or text instead of HTML via `output_format`, but the structured listing data comes from the page's embedded JSON, which you parse. If you want ready-made fields with no parsing, that's a scraper product, not a proxy API.

**Do I need residential proxies for Airbnb?** Not always, but anti-bot protection on the site means the bypass may fire and cost 10 extra credits per request. The `premium` parameter restricts traffic to residential and mobile IPs; `ultra_premium` is there for the toughest targets.

**What happens if I go over my plan?** You get a 429, the request is rejected rather than queued, and there's no surcharge. Scraping resumes when credits renew, or immediately if auto-upgrade or pay-as-you-go is switched on.

**Is there a free trial?** Yes, a 7-day trial with 5,000 API credits and no credit card required, on top of the 1,000-credit free plan. Paid plans carry a 7-day no-questions-asked refund. 👉 [Start the trial and confirm the current terms](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How to choose, and what to check before you pay

Match the plan to the shape of the job, not to the maximum you might one day need.

- **Ten to fifty Airbnb URLs, one-off** → the free plan. 1,000 credits and five connections will cover a test and then some.
- **A few cities, refreshed weekly, search pages only** → Hobby or Startup, with auto-upgrade on so a busy week doesn't stall the run.
- **Continuous monitoring with a fixed budget** → Scaling or Professional, where pay-as-you-go lets you set a monthly cap instead of jumping tiers.
- **You need descriptions, amenities, host profiles and reviews as named fields, and you don't want to write a parser** → a scraper product that returns rows is the better buy, and per-row pricing will likely come out cheaper than 1,000 detail-page requests.

Two things to confirm before checkout. Run your actual Airbnb URLs through the Domain Cost Estimator first and write down the per-URL credit cost, because a rendered page with a bot-protection bypass is not a one-credit page. Then decide whether your job needs detail pages at all, since that decision, not the tier, is what moves the invoice.

👉 [Compare the ScraperAPI plans and start with the free credits](https://www.scraperapi.com/pricing?fp_ref=coupons)
