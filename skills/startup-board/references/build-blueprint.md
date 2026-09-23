# Build Blueprint (MVP, no-code/prompt-first)

Default assumption unless told otherwise: solo founder, prompt-first, no dev team, wants to ship fast and cheap. Recommend the lightest stack that proves the core loop — not the most impressive one.

## Default stack recommendation

| Layer | Default pick | When to swap it |
|---|---|---|
| Frontend | React (Claude Artifacts / a no-code builder like Claude Design, or v0/Lovable-style tools) | Swap to a pure no-code builder (e.g., a form/site builder) if there's no real interactivity needed |
| Backend/DB | Supabase (Postgres + auth + storage in one) | Airtable if the "backend" is just structured data an ops person edits directly |
| Automation/glue | Supabase Edge Functions + pg_cron for event-driven and scheduled jobs; a direct transactional email API (Resend/Postmark) for notifications | Zapier/Make only if the founder specifically wants a visual workflow builder over writing Edge Functions |
| AI layer | Anthropic API (Claude) for any generation/reasoning feature | — |
| Auth | Supabase Auth or the no-code builder's built-in auth | — |
| Hosting | Vercel/Netlify for frontend; Supabase handles its own hosting | — |

## MVP scope cut

1. List every feature the founder mentioned.
2. Mark each as: **core loop** (the single thing that delivers the value prop), **nice-to-have**, or **later**.
3. The MVP ships **core loop only** — everything else gets explicitly deferred, not quietly dropped (write it down so it isn't forgotten, just not built yet).
4. Sanity check the core loop against the Lean Canvas's UVP — if the MVP doesn't deliver the UVP, the scope cut went too far.

## Data model sketch

Produce a short entity list (not a full schema) — table/entity names, key fields, and relationships in plain language, enough to hand to Supabase table setup or an AI code tool. Example shape:

```
users (id, email, created_at)
projects (id, user_id -> users, name, status)
project_items (id, project_id -> projects, ...)
```

## Build prompts (ready to hand to a code-gen tool)

For each core-loop piece, produce a specific, ready-to-use prompt rather than a vague instruction. Template:

> "Build a [React component/Supabase table/Edge Function] that [specific behavior]. Input: [...]. Output: [...]. Edge cases to handle: [...]. Use [stack piece] conventions."

Always fill in the brackets concretely from the actual idea — a generic template handed back unfilled isn't useful.

## Sequencing

1. Data model + auth first (Supabase tables + auth).
2. Core loop UI (single flow, not the whole app).
3. Wire the AI layer if the product needs it.
4. Automation/glue (Edge Functions) last — connect what already works, don't build the plumbing before there's anything to plumb.
5. Ship to the 10 people already interviewed before building anything beyond the core loop.
