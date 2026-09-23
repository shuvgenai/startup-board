# Startup Board v1.1.0 — Fit pass tests, Product/Market fit check-in, Scale & growth

**Date:** 2026-09-23
**Status:** Approved design, awaiting written-spec review
**Branch:** `feature/fit-and-scale`

## 1. Intent

**Outcome:** Startup Board keeps helping founders *after* launch, so they have a reason to return with real numbers — not just a one-time idea check.

**Audience:** unchanged — no-code, prompt-first, solo founders.

**Success criteria.** The skill can:
1. Tell a founder, with a concrete evidence-based test, when they have passed each fit stage (especially Problem/Solution fit).
2. Measure Product/Market fit from numbers the founder types or pastes into chat.
3. Guide solo-founder scaling after launch: a repeatable channel, CAC vs LTV, pricing iteration, a sales playbook, and the first hire.

**Invariants (must not change):** one advisor voice (no board-member names or "seat" language), exactly one question per turn in a visible box, never invent numbers, one next action rather than a to-do list, Phases 6–10 never auto-run on a founder who hasn't earned them.

## 2. Decisions

| Topic | Decision |
|---|---|
| Structure | Keep Phases 0–8 as they are. Add **Phase 9 (Product/Market fit check-in)** and **Phase 10 (Scale & growth)**, which run on request or when the founder brings real post-launch data. |
| File layout | Approach A: two new reference files plus small edits to existing ones. No second skill. |
| Data input | Founders type or paste numbers in chat. No file upload or CSV parsing. |
| Stage gates | Warn, then allow: name the unmet test and the missing evidence, give one closing action, then help if the founder insists and stamp the output with the gap. |
| Memory | A founder-editable progress file, `startup-board-progress.md`. |
| Scale scope | Solo-founder scale only: up to the first repeatable growth engine and the first hire. No growth-team, multi-channel budgeting or fundraising-linked growth. |
| Version | 1.0.1 → **1.1.0** |

## 3. File changes

All paths are relative to the repo root. The plugin and standalone skill share `skills/startup-board/`, so one change covers both.

| File | Change |
|---|---|
| `skills/startup-board/references/fit-framework.md` | Add a **Pass test** block to each of the six stages, the warn-then-allow procedure, and a "rules of thumb, adjust for context" note. |
| `skills/startup-board/references/pmf-checkin.md` | **New.** Phase 9 procedure (§5). |
| `skills/startup-board/references/scale-growth.md` | **New.** Phase 10 procedure (§6). |
| `skills/startup-board/references/output-documents.md` | Add the progress-file format and read/write rules (§7), plus the `pmf-scorecard.md` and `growth-plan.md` output formats. |
| `skills/startup-board/references/gtm-launch.md` | Add one line pointing to Phase 10 for post-launch scaling. No other change. |
| `skills/startup-board/SKILL.md` | Frontmatter description + triggers, "Returning founders" rule, Phase 2 and Phase 5 edits, new Phase 9 and 10 sections (§8). |
| `.claude-plugin/plugin.json` | Version 1.1.0; description gets the new triggers. |
| `README.md` | Version badge 1.1.0; Phase 9 and 10 rows in the "Build, launch, and raise" table; FAQ entry on product-market fit and scaling. |

Out of scope: updating the promo films (`promo/`) — a separate follow-up.

## 4. Pass tests (fit-framework.md)

Every test counts **past behaviour and commitments**, never opinions or stated intent. Thresholds are rules of thumb; the skill says so and may adjust them for context (e.g. fewer interviews for enterprise B2B with a tiny buyer pool), stating the adjustment out loud.

| Stage | Pass test |
|---|---|
| 1. Problem Fit (Problem/Solution) | ≥10 Mom Test–style interviews with the target segment; ~7 of 10 describe the problem from their own recent experience; ~half already spend time or money on a workaround. |
| 2. Research Fit | Founder can name how people solve it today (including DIY/manual) and the top 3 alternatives; the riskiest assumption is written as an if-then hypothesis with a test and a pass number. |
| 3. Product Fit | ≥5 target users complete the core loop without help; ≥3 real commitments (pre-order, deposit, signed LOI, or paid pilot). A waitlist email alone does not count. |
| 4. Customer Fit | Founder can describe the single highest-value customer profile; that group returns at its natural cadence for ≥4 weeks; ≥1 unprompted referral. |
| 5. Market Fit (Product/Market) | Measured by Phase 9 (§5): verdict **Yes**. |
| 6. Sales Fit | Measured by Phase 10 (§6): one channel delivers customers at a predictable cost for 3 consecutive months; LTV ≥ 3× CAC; CAC payback ≤ 12 months; someone other than the founder has won a customer using the playbook. |

**Warn-then-allow procedure:**
1. State which test is unmet and the exact missing evidence ("You have 4 interviews; the test needs 10").
2. Name the one action that closes the gap.
3. If the founder still wants to proceed, help — and put a one-line stamp at the top of any produced document: `> Built before <Stage> was passed. Missing: <evidence>.`

**Exception:** Phase 9 cannot run without users. With no live users, the skill explains that Product/Market fit can't be measured yet and returns the founder to their actual stage. It does not allow a Phase 9 run on zero data.

## 5. Phase 9 — Product/Market fit check-in (`pmf-checkin.md`)

**Triggers:** "do I have product-market fit", "are we ready to scale", "are people actually using this", or a progress file showing the founder has launched.

**Flow (one question per turn):**
1. Define "active user" for this product and its natural usage cadence (daily / weekly / monthly).
2. Collect three signals, one at a time:
   - **Sean Ellis survey** — share of active users answering "very disappointed" to "How would you feel if you could no longer use [product]?" Benchmark ≥40%, from ~30+ active-user responses. Below ~30 responses: treat the result as directional and say so.
   - **Retention** — for a cohort that started together, how many are still active at a few points (e.g. week 1 / 4 / 8, or month 1 / 2 / 3). Healthy: the curve flattens above zero. Judge the shape, not a single number.
   - **Pull** — the share of last period's new users or customers who arrived without founder push (word of mouth, organic), and whether it's growing. If the product charges, also: are customers paying and renewing?
3. **Segment check** — if the founder has segment breakdowns, compare the survey result per segment. A segment at or above benchmark while the overall number is below changes the verdict to *In one segment*. The same ~30-response rule applies per segment: a smaller segment can still earn *In one segment*, but the skill labels it directional and suggests surveying more of that segment.
4. **Verdict:** *Not yet* / *In one segment* / *Yes*.
5. **One next action:**
   - *Not yet* → the single product change most likely to convert "somewhat disappointed" users, based on what "very disappointed" users say they value most.
   - *In one segment* → narrow focus to that segment.
   - *Yes* → offer Phase 10.

**Missing data:** never estimate a missing signal. Instead, provide a **measurement kit**: the survey text (the Ellis question plus "What type of person would benefit most?", "What is the main benefit you get?", and "How could we improve it for you?"), a retention-cohort table layout to fill in, and plain guidance on where such numbers usually live in the founder's existing tools. Ask them to return with results.

**Output:** `pmf-scorecard.md` — each signal, its value, real/estimated, pass/fail, the verdict, and the one next action. Append a dated evidence row to the progress file; if prior check-ins exist, state the trend ("up from 28% to 36% since your last check-in").

## 6. Phase 10 — Scale & growth (`scale-growth.md`)

**Triggers:** a Phase 9 verdict of *Yes* or *In one segment* (then growth stays inside that segment), or an explicit request ("how do I scale", "which channel", "CAC", "LTV", "pricing", "first hire", "sales playbook"). A request made before Product/Market fit triggers warn-then-allow, with the warning: scaling before people love the product mostly buys more churn.

**First question:** which of the five areas to start with, suggesting the one where the founder's numbers look weakest.

1. **One repeatable channel** — start from the channel that already brought the best customers (from the founder's data). Run it as a fixed experiment: duration, target number, budget cap. No second channel until the first repeats.
2. **Unit economics** — calculate and show the working:
   - CAC = (acquisition spend + tool costs for the period) ÷ new customers in the period.
   - LTV = average monthly revenue per customer × gross margin ÷ monthly churn rate.
   - Payback (months) = CAC ÷ (monthly revenue per customer × gross margin).
   Label every input *real* or *estimated*. Rules of thumb: LTV ≥ 3× CAC; payback ≤ 12 months, ideally ≤ 6 for a self-funded founder.
3. **Pricing iteration** — price on the value metric customers actually care about; test changes on new customers only; if no one ever pushes back on price, it's likely too low.
4. **Sales playbook** — write down the founder's real process: lead source → first message → demo/onboarding → close → follow-up, in enough detail for someone else to run it.
5. **First hire** — *when:* a proven, repeatable task consumes a large share of the founder's week; *who:* the role that removes the biggest founder bottleneck, often a part-time contractor first; *how:* a paid trial project. Always ask "can a no-code automation do this first?" before recommending a hire.

**Output:** `growth-plan.md` — a 90-day plan: one channel experiment with a target, the unit-economics table (real/estimated labelled), one pricing move, the playbook draft, and a hiring trigger. Append a dated row to the progress file. Report whether the Sales Fit pass test (§4) is met.

## 7. Progress file (`output-documents.md`)

**Name/location:** `startup-board-progress.md` in the founder's current working folder (Claude Code). In the claude.ai app, the skill produces it as a file artifact and, on return, asks the founder to paste it in.

**Format:**
```
# Startup Board progress: <idea name>
Last updated: YYYY-MM-DD
Current stage: <stage> (<passed | not yet passed: missing evidence>)
Latest verdict: <verdict>
Riskiest assumption: <one sentence>

## Evidence log
| Date | Stage | Signal | Value | Real or estimated |
|---|---|---|---|---|

## Next action
<one action>
```

**Rules:**
- **Read** at session start. If found: one-line summary of what's remembered, ask the founder to confirm (one boxed question), skip already-answered intake, resume at the recorded stage.
- **Create** after the Phase 5 verdict. **Update** after Phases 9 and 10, and whenever a pass test changes state.
- Founder edits win: if the file differs from what the skill last wrote, trust the file.
- Store only idea, stage, verdict, assumption, numbers, and next action — no personal data beyond what the founder chose to share.
- Evidence log is append-only; never rewrite past rows.

## 8. SKILL.md edits

- **Frontmatter description:** add PMF/scale triggers ("product-market fit", "ready to scale", "CAC/LTV", "which channel", "first hire"). Must stay ≤1024 characters; tighten existing wording if needed. Mirror the change in `plugin.json`.
- **"When this skill fires":** add the post-launch triggers.
- **New "Returning founders" rule** (before Phase 0): check for the progress file per §7.
- **Phase 2:** use the pass tests in `fit-framework.md`; apply warn-then-allow.
- **Phase 5:** create/update the progress file; mention Phases 9–10 exist for after launch.
- **Downstream phases intro:** now "Phases 6–10".
- **New Phase 9 and Phase 10 sections:** 3–5 lines each, pointing to `pmf-checkin.md` and `scale-growth.md`, restating the one-question rule and "never estimate missing signals".

## 9. Testing

Skills are tested by scenario, not unit tests. Subagents role-play a founder against the updated skill; each transcript is checked against the scenario's pass criteria **and** the invariants in §1.

| # | Scenario | Pass criteria |
|---|---|---|
| 1 | New idea, 4 interviews done | Phase 2 says Problem Fit not passed, cites "4 of 10", gives exactly one action. |
| 2 | Returning founder with a progress file | Reads the file, confirms in one line, skips answered intake, resumes at the recorded stage. |
| 3 | Launched: Ellis 32% overall / 46% in one segment, retention flattening | Verdict *In one segment*; `pmf-scorecard.md` produced; one next action. |
| 4 | Asks to scale before Product/Market fit | Warns, then helps; gap stamp on `growth-plan.md`; CAC/LTV working shown; inputs labelled real/estimated. |
| 5 | Regression: freelancer late-invoice idea | Still one boxed question per turn, one voice, no board names, no invented numbers. |

Any failure → fix the relevant reference/SKILL.md wording and re-run that scenario.

## 10. Out of scope

- Updating promo films (follow-up task).
- CSV/file import of metrics.
- Growth-team structure, multi-channel budgeting, fundraising-linked growth.
- Any change to Phases 0–8 beyond the Phase 2/5 hooks above.
