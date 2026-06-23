# Jake Gilbert
**AI Automation Engineer — n8n · Agentic Systems · RAG**

Available immediately for remote · Relocating to Denver, CO late 2026
jakegilbert@workingclassai.io · 720-308-6010
GitHub: github.com/jakegilbert-dev/ai-automation-portfolio
LinkedIn: linkedin.com/in/jake-gilbert-015842369

---

## Summary

AI automation engineer who designs and ships production agent systems end to end. Founder and sole engineer of Working Class AI, where I architected a complete multi-agent sales operation on n8n — multi-channel inbound agents with RAG and persistent memory, LLM-scored lead-gen pipelines, and voice/SMS automation. Comfortable across the full stack of modern automation: orchestration, LLM prompting for reliable structured output, vector retrieval, API integration, and the unglamorous debugging that makes agents actually reliable in production.

---

## Core skills

**Orchestration:** n8n (self-hosted on a Hostinger VPS), webhooks, scheduled/cron triggers
**LLMs:** Anthropic Claude, Google Gemini; prompt engineering, tool/function calling, structured-output parsing
**RAG:** Supabase / pgvector, document ingestion pipelines, chunking, embedding, retrieval
**Voice & messaging:** Vapi, Twilio (SMS + voice, A2P 10DLC), Telegram bots, Meta Messenger (Graph API), SMTP/IMAP
**Data & CRM:** Airtable, Postgres (conversation memory), Google Sheets
**Scraping & enrichment:** Google Maps Places API, Firecrawl, LLM qualification/scoring
**Code:** JavaScript (n8n Code nodes), REST APIs, JSON transformation

---

## Experience

### Founder & AI Automation Engineer — Working Class AI
*January 2026 – present*

Built, solo, a production multi-agent system for SMB sales automation. Selected work:

- **Multi-channel inbound agent ("Alex")** — Designed a single agent handling email, SMS, Telegram, and chat through a channel-normalization layer, with persistent Postgres conversation memory keyed to lead ID (so conversations survive across channels) and a Supabase RAG knowledge base for grounded answers. Runs on Vapi for voice. Agent writes qualifying details back to CRM via tool calls.
- **LLM lead-gen pipeline** — Telegram-controlled workflow that scrapes business directories (Google Maps Places API), enriches via website crawl (Firecrawl), and qualifies each lead with a Gemini prompt extracting seven fields plus a 1–5 fit score, then routes to the correct CRM table. Ordered cheap metadata filters ahead of expensive LLM/crawl calls to control cost.
- **RAG ingestion pipeline** — Airtable-driven pipeline ingesting mixed media (text, PDF, video), converting each to text and embedding into a Supabase vector store; diagnosed and fixed an embedding dimension mismatch and a mid-workflow data-context bug.
- **Outbound voice + SMS sequences ("Zoey")** — Time-gated, multi-attempt follow-up campaigns (Vapi + Twilio) with opt-out handling and per-attempt messaging angles.
- **Production reliability** — Hunted down the quiet failure modes that break agents in practice: empty tool-call writes, identifier-format mismatches causing duplicate records, overly strict routing conditions silently dropping valid input, and JSON-body expression handling in n8n HTTP nodes.
- **External build** — Adapted the lead-gen pattern for a fleet-vehicle pressure-washing business: a tailored Telegram lead-finder bot and cold-email agent with fleet-only filtering, malformed-output fallback, and full error handling, deployed on my own infrastructure. (Built and set live; not yet in active use.)

*Note: Working Class AI is my own venture; the work above reflects systems I designed and built, not paid client engagements.*

---

## Hands-on background

Strong mechanical and diagnostic background as a hobbyist — substantial hands-on personal automotive work including engine and drivetrain repair (not professional or certified). The same systematic troubleshooting instinct (isolate the variable, find the quiet failure, verify the fix) is what I bring to debugging production automation.

---

## Portfolio

Detailed case studies with architecture write-ups and sanitized workflow JSON: **github.com/jakegilbert-dev/ai-automation-portfolio**

- Lead-Gen Pipeline — scrape → enrich → AI-score → route
- Multi-Channel Inbound Agent with RAG + Memory
- RAG Document Ingestion Pipeline
