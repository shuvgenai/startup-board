# Market Research Modes

This replicates the method behind several open-source Claude Skills (genli-ai/market-research-skills, birne-sk/claude-skills, ishwarjha/claude-marketing-research-skill, the market-researcher subagent pattern) directly, using your web search and page-fetch tools (`WebSearch`/`WebFetch` in Claude Code, `web_search`/`web_fetch` in the claude.ai app) — no external install required. Pick the mode that matches what's actually needed; don't default to heavy.

## Light (default first pass — a few searches, ~10-15 min equivalent)
- Quick competitor scan: who else solves this problem today, 3–5 names.
- Rough market read: is this a known/growing category, shrinking, or nonexistent-as-a-category (which can be good or bad — say which).
- One-paragraph industry-fit verdict: which industry/vertical this idea actually belongs in, and why.

## Medium (when the founder has passed the fresh-idea stage, or asks for more depth)
Everything in Light, plus:
- **Competitor deep-dive** (3–5 competitors): pricing model, target customer, apparent differentiation, weaknesses visible from the outside.
- **Positioning map**: 2-axis map (e.g., price vs. specialization, self-serve vs. high-touch) showing where competitors sit and where the gap is.
- **Buyer persona sketch**: role, company size/context, trigger event that makes them look for a solution (tie to Christensen's JTBD lens from board-members.md).

## Heavy (only on explicit request — confirm before running, it's search-heavy)
Everything in Medium, plus:
- **Market sizing (TAM/SAM/SOM)**: cite sources for any market-size figure used; if no reliable public figure exists, say so rather than inventing one, and build a bottom-up estimate instead (number of potential customers × realistic price) with assumptions stated.
- **Industry trend scan**: 2–3 recent developments (funding, regulation, notable launches) affecting this space, each cited.
- **McKinsey/BCG-style 4-level pyramid structure** for the writeup:
  1. **Governing thought** — the one-sentence takeaway/recommendation
  2. **Key arguments** (3–4) — the pillars supporting it
  3. **Supporting evidence** — data/sources under each pillar
  4. **Detailed analysis** — the full backing detail, appendix-style
- Deliver as a markdown or docx report (use the docx skill if the founder wants a polished Word doc) rather than inline chat.

## Method notes (apply to every mode)
- Cite sources briefly per response; never fabricate a number — if something can't be verified, say "estimate, unverified" rather than presenting it as fact.
- Favor primary/original sources (company sites, filings, industry reports) over aggregator blogspam.
- Keep quoted text under copyright limits — paraphrase, don't reproduce.
