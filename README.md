[README.md](https://github.com/user-attachments/files/29184819/README.md)
# AI Automation Engineering — Portfolio

I'm Jake Gilbert. I design and build production AI agent systems on **n8n**: multi-channel agents, RAG pipelines, voice/SMS automation, and CRM-integrated lead workflows.

This portfolio documents a real, end-to-end multi-agent system I architected and built solo as the founder and sole engineer of **Working Class AI** — an AI automation venture targeting small and medium businesses. Everything here is something I actually built, debugged in production, and shipped. The workflow JSON has been sanitized (credentials, keys, and identifiers removed) so it can be shared publicly.

---

## What I work with

**Orchestration:** n8n (self-hosted via Docker on a VPS), webhooks, scheduled triggers, cron gating
**LLMs & AI:** Anthropic Claude, Google Gemini, Groq (Llama 3.3-70B), prompt engineering for reliable structured output, tool-calling / function-calling agents
**RAG:** Supabase / pgvector embeddings, document ingestion pipelines, chunking, retrieval-augmented agents
**Voice & messaging:** Vapi (voice agents), Twilio (SMS + voice, A2P 10DLC registration), Telegram bots, Facebook/Instagram Messenger (Meta Graph API), email (SMTP/IMAP)
**Data & CRM:** Airtable, Postgres (conversation memory), Google Sheets
**Scraping & enrichment:** Google Maps Places API, Firecrawl, LLM-based qualification/scoring
**Glue:** JavaScript (n8n Code nodes), REST APIs, JSON transformation, multipart uploads

---

## The system at a glance

A complete inbound + outbound AI sales operation, built as a set of interconnected n8n workflows:

| Agent / Workflow | What it does |
|---|---|
| **Lead-gen bot** (Telegram-controlled) | Scrapes business directories, enriches via website crawl, scores fit with an LLM, routes to CRM |
| **Multi-channel inbound agent** | Handles inbound email / SMS / Telegram / chat with persistent memory and a RAG knowledge base |
| **RAG ingestion pipeline** | Pulls documents from a knowledge base, converts and embeds them into a vector store |
| **Outbound voice + SMS sequences** | Time-gated, multi-attempt follow-up campaigns with opt-out handling |
| **Cold email agent** | Multi-attempt personalized email sequences with per-attempt messaging angles |

The three case studies below go deep on the most representative pieces.

I've also adapted this lead-gen pattern for an external business — a fleet-vehicle pressure-washing company — building a tailored lead-finder bot and cold-email agent with fleet-only filtering and full error handling, deployed on my own infrastructure. (Built and set live; not yet in active use by the business.) It's a smaller variant of Case Study 1 rather than a separate write-up, but it's a real example of building to someone else's requirements rather than my own.

---

## Case studies

### 1. [Lead-Gen Pipeline — scrape → enrich → AI-score → route](./case-studies/01-lead-gen-pipeline.md)
A Telegram-controlled workflow that turns a plain-English command ("Find HVAC companies in Dallas") into a stream of pre-qualified, scored leads in a CRM. Covers the Google Maps + Firecrawl scrape, a JavaScript pre-filter, a Gemini qualification prompt extracting seven fields plus a 1–5 fit score, and three-way CRM routing. Includes the real bugs I hit and how I fixed them.

### 2. [Multi-Channel Inbound Agent with RAG + Memory](./case-studies/02-multichannel-agent.md)
My most advanced build: a single agent that answers across email, SMS, Telegram, and chat, normalizes each channel into one pipeline, keys persistent Postgres conversation memory to a lead ID, retrieves from a Supabase vector knowledge base, and writes back to the CRM via tool calls. Covers the architecture and the debugging of tool-call wiring and channel normalization.

### 3. [RAG Document Ingestion Pipeline](./case-studies/03-rag-ingestion.md)
The knowledge-base backbone: an Airtable-driven pipeline that ingests mixed media (text, PDF, video), converts each type to text, and stores vector embeddings in Supabase. Covers the Switch-node media routing, a vector dimension mismatch I diagnosed and fixed, and a subtle data-context bug where the vector store overwrote the record reference mid-workflow.

---

## How to read this

Each case study follows the same shape: **the problem**, **the architecture**, **the hard parts** (real bugs and design decisions), and **what I'd do next**. The point isn't that these systems are perfect — it's to show how I think through building and debugging production automation.

Sanitized workflow JSON for each is in [`/workflows`](./workflows).

---

*Open to AI automation / applied-AI engineering roles. Relocating to the Denver area in late 2026.*
