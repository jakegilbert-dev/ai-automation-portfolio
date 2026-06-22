# Sanitized Workflow JSON

This folder is where the exported n8n workflow JSON for each case study goes.

**Before you commit anything here, sanitize it.** Export each workflow from n8n, then strip:

- API keys, tokens, and `Authorization` headers (Google Maps, Firecrawl, Gemini, Anthropic, Twilio, Supabase, Meta page tokens)
- Credential IDs and credential names
- Base IDs, table IDs, and any real record IDs
- Phone numbers, email addresses, chat IDs, webhook URLs/paths
- Your VPS hostname and any internal URLs
- Business names or client identifiers

Replace each with an obvious placeholder like `YOUR_API_KEY`, `YOUR_BASE_ID`, `https://your-instance.example.com/webhook/...`.

A quick way to check before committing: search the file for `http`, `key`, `token`, `secret`, `+1`, `@`, and your own domain, and confirm every hit is a placeholder.

Suggested filenames:
- `lead-gen-pipeline.sanitized.json`
- `multichannel-agent.sanitized.json`
- `rag-ingestion.sanitized.json`
