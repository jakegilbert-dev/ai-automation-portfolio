# Case Study 2 — Multi-Channel Inbound Agent with RAG + Memory

**Stack:** n8n · Anthropic / Gemini · Supabase (pgvector) · Postgres · Airtable · Twilio · Telegram · Meta Graph API · SMTP/IMAP
**Pattern:** channel normalization → unified agent → persistent memory + RAG → tool-based CRM writes → multi-channel response

---

## The problem

A lead might first reach out by email, then reply by SMS, then ask a follow-up over Facebook Messenger. To them it's one conversation. Most automations treat each channel as a separate silo, which means no memory, repeated questions, and duplicate CRM records. I wanted **one agent** that behaves like a single person who remembers the whole relationship regardless of which channel a message comes in on.

## The architecture

The agent (I call it Alex) runs as one n8n workflow fed by multiple channel triggers:

1. **Channel normalizer.** Inbound messages from email, SMS, Telegram, and chat each arrive in a different shape. A normalization step maps every channel into one common envelope — sender identity, message body, channel type — so everything downstream is channel-agnostic.
2. **Lead lookup / create.** The agent searches Airtable for an existing lead by identifier. If none exists, it creates one — then reloads the freshly created record so downstream nodes have the real record ID to work with.
3. **Persistent memory.** Conversation memory lives in Postgres, **keyed to the lead ID** rather than to a session. That's what lets a conversation survive across channels and across time — an SMS today continues the email thread from last week.
4. **RAG knowledge base.** The agent retrieves from a Supabase vector store containing the business's service info, pricing, and onboarding process, so answers are grounded in real source material instead of being made up.
5. **Tool-based CRM writes.** The agent can update fields (name, email, company, scope of work) via tool calls, so qualifying details captured mid-conversation get written back to the CRM automatically.
6. **Multi-channel response.** The reply goes back out on whichever channel the message came in on.

## The hard parts

**Tool calls that silently did nothing.** When I first wired the CRM-update tools, the value expressions were left blank (`={{}}`) — the agent was *calling* the tools but writing empty values, so nothing actually saved. The fix was making sure each tool's parameters were properly populated by the agent's structured output. Lesson: a tool that's wired but unparameterized fails *quietly*, which is the worst kind of failure.

**A placeholder that never got filled.** The "reload lead after create" path referenced a placeholder table ID and a node alias that didn't exist, which crashed the entire new-lead branch. Existing leads worked fine, so the bug only showed up for first-time contacts — exactly the people you can't afford to drop. I traced it to the bad reference and pointed it at the real "create lead" node.

**Two CRMs that didn't talk.** At one point the cold-email side wrote to Google Sheets while the agent read from Airtable, so a lead emailed from one system didn't exist when they replied into the other. I consolidated onto a single source of truth so inbound replies always resolve to the right record.

**Channel identity formatting.** Twilio delivers numbers as `+1XXXXXXXXXX` while the CRM stored them as `(XXX) XXX-XXXX`. Without normalization, lookups missed and the agent created duplicate records for people it had already met. A formatting/normalization step on the identifier fixed the match rate.

## What I'd do next

- Add a confidence check before tool-writes so the agent confirms ambiguous details instead of overwriting good data.
- Unify outbound and inbound call handling through a single result handler keyed on a metadata source flag.
- Add structured logging per turn for easier debugging of long multi-channel threads.

## Takeaway

The architecture — normalize channels in, key memory to the lead, ground answers in RAG, write back via tools — is the part I'm proud of. But the real lesson was in the failure modes: the bugs that mattered most were the *quiet* ones (empty tool writes, unmatched phone formats, a crash on a path that only first-time leads hit). Building reliable agents is mostly about hunting down the failures that don't announce themselves.
