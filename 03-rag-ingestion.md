# Case Study 3 — RAG Document Ingestion Pipeline

**Stack:** n8n · Airtable · Supabase (pgvector) · PDF/text/video extraction
**Pattern:** source-driven ingestion → media-type routing → text conversion → vector embedding

---

## The problem

A RAG agent is only as good as the knowledge base behind it. I needed a pipeline that takes mixed-format source material — plain text, PDFs, and video — out of a simple Airtable "knowledge base" table, converts each into clean text, embeds it, and lands it in a Supabase vector store the agent can retrieve from. It had to handle each media type correctly and never silently drop a document.

## The architecture

1. **Airtable source.** Documents live in a knowledge-base table with a media-type indicator and a content field. The pipeline pulls records that have source media but haven't been indexed yet.
2. **Switch node — media routing.** Each record is routed by type: text downloads directly, PDFs go through extraction, video goes through transcription.
3. **Text conversion.** Each branch produces plain text — PDF extraction via n8n's built-in `extractFromFile` (PDF operation), video via transcription, text passed through.
4. **Write text back to Airtable.** The converted text is saved to the record's content field so there's a readable record of what got embedded.
5. **Embed into Supabase.** The text is chunked and stored as vector embeddings in a Supabase `documents` table for retrieval.
6. **Mark as indexed.** The record is flagged so it isn't reprocessed on the next run.

## The hard parts

**Vector dimension mismatch.** Indexing failed because the embedding dimensions written didn't match the column the Supabase table expected (768 vs 3072). The fix was dropping and recreating the `documents` table to match the embedding model's output dimensions. Takeaway: embedding dimension is a hard contract between your model and your vector store — they have to agree exactly.

**The vector store overwrote my record reference.** A genuinely subtle one. The node that marked a document as "indexed" referenced `$json.id` — but by that point in the workflow, the Supabase insert had overwritten the data context, so `$json.id` was no longer the Airtable record ID. That threw a 422 from Airtable. The fix was referencing the original record explicitly via the search node (`$('Search records').item.json.id`) instead of relying on `$json`, which had been mutated downstream. The deeper lesson: in a linear workflow, later nodes can clobber the context earlier nodes set — reference data by its source node, not by position.

**A PDF branch that silently dropped files.** The original Switch node had no PDF path, so PDFs fell through and vanished without an error. I added a dedicated PDF branch. Related: the video condition used an exact `equals "Video"` match instead of `contains "video"`, which would silently drop real MIME types like `video/mp4`. Both are the same class of bug — overly strict routing conditions that drop valid input without complaining.

**A single space character broke a filter.** A previous failed run wrote a lone space into a content field. The "needs indexing" filter checked `Content = ''` (empty), and a space isn't empty, so that record was skipped forever. Clearing the space fixed it. Tiny cause, invisible effect — exactly the kind of thing worth knowing to look for.

## What I'd do next

- Replace the manual trigger with a schedule trigger so new documents get ingested automatically.
- Add a validation step that rejects empty/whitespace-only content before it pollutes the index.
- Track embedding model + dimension in config so a model change can't silently corrupt the store.

## Takeaway

This pipeline taught me that RAG reliability lives in the unglamorous details: dimension contracts, routing conditions strict enough to be wrong, and data context getting mutated mid-flow. Getting embeddings into a vector store is easy. Getting *every* document in correctly, every run, without silent drops, is the actual job.
