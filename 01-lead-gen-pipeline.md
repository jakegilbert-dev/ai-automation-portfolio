# Case Study 1 — Lead-Gen Pipeline: scrape → enrich → AI-score → route

**Stack:** n8n · Telegram Bot API · Google Maps Places API · Firecrawl · Google Gemini · Airtable
**Pattern:** chat-controlled scraping + LLM qualification + conditional CRM routing

---

## The problem

Sourcing SMB leads by hand is slow, and raw scraped lists are mostly noise — closed businesses, tiny operations that won't convert, places with no website to pitch against. I wanted a single command I could fire from my phone that would return a stream of leads *already qualified and scored*, so outreach effort goes to the businesses most likely to convert.

## The architecture

The whole pipeline is driven by a Telegram bot. I type something like `Find HVAC companies in Dallas Texas` and the workflow does the rest:

1. **Telegram trigger** parses the command and extracts the search query.
2. **Google Maps Places API** pulls matching businesses. I expanded the field mask beyond the basics to also return `rating`, `userRatingCount`, and `businessStatus` — the signals I need to filter on.
3. **JavaScript pre-filter (Code node)** drops low-quality results *before* spending any LLM or crawl budget on them: fewer than 10 reviews, under 3.5 stars, or not operational get cut.
4. **Firecrawl** crawls each surviving business's website to get real text to qualify against.
5. **Empty-page guard** — a filter that removes businesses whose crawl returned empty markdown, so the LLM never gets fed blank content.
6. **Gemini qualification (LLM Chain)** reads the crawled content and extracts **seven fields** — including size signals, hiring activity, and likely call volume — plus a **1–5 fit score** against my ideal customer profile.
7. **Parse LLM Response (Code node)** normalizes the model output into clean fields, handling missing values gracefully.
8. **Score filter** keeps only leads scoring 3+.
9. **Three-way Airtable routing** sends each lead to the right table: *Leads* (has email), *No Email Leads* (phone only), or *Errors* (crawl/qualification failure, with the actual failure reason captured for debugging).

A duplicate-phone check runs before saving to the No-Email table so the same business doesn't get created twice across runs.

## The hard parts

**JSON body expressions in HTTP Request nodes.** The single most time-consuming bug. In n8n's HTTP Request nodes using JSON body mode, expressions must be written as `{{ expression }}` *without* a leading `=`. Writing `={{ expression }}` inside a JSON string causes the literal `=` to appear in the output instead of the evaluated value. This silently corrupts API calls in a way that's hard to spot. Once I understood it, I standardized on it across every workflow.

**Field-name drift across tables.** Early on I had a field referenced as `Company` in some nodes and `Business Name` in others across the three CRM tables. Airtable rejects writes to a field that doesn't exist, so leads silently failed to save. I standardized on `Business Name` everywhere and made schema consistency a rule, not an afterthought.

**Filtering before spending budget.** The ordering matters: pre-filter on cheap Google Maps metadata *first*, crawl *second*, run the LLM *last*. Putting the JavaScript pre-filter ahead of the Firecrawl crawl and the Gemini call meant I wasn't paying for enrichment on businesses I was going to throw away anyway.

## What I'd do next

- Move scoring thresholds into a config table so I can tune the ICP without editing nodes.
- Add a lightweight cache so re-running the same query doesn't re-crawl unchanged sites.
- Emit a run summary (counts per table, average score) back to Telegram after each run.

## Takeaway

The interesting work here wasn't any single node — it was sequencing cheap filters ahead of expensive ones, making the LLM output reliably parseable, and getting the CRM routing to never drop or duplicate a lead. That's the difference between a demo and something you'd actually run every day.
