# Output Documents — Format Guidance

Match the deliverable to how the founder will actually use it. Don't over-produce — only build what's asked for or clearly useful at this stage (see SKILL.md Phase 4).

| Document | When to produce | Format |
|---|---|---|
| Lean Canvas | Always offer at minimum, once Phase 0–2 are done | Markdown table/grid inline, or a saved `.md` file if the founder wants to keep iterating on it |
| Customer interview questions | Whenever discovery hasn't happened yet, or founder asks | Inline markdown list — short enough not to need a file |
| Market research brief (light) | Default research pass | Inline chat, a few paragraphs + a short comparison table if competitors were found |
| Market research brief (medium/heavy) | On request or once idea is past fresh-idea stage | Saved markdown file — in Claude Code, a `.md` file in the current folder; in the claude.ai app, a file artifact (or `.docx` via a docx skill if the founder wants a polished, shareable document) — this is meant to be saved and referenced, not read once in chat |
| Positioning/competitor snapshot | When 2–3+ competitors were researched | Short markdown table (in the claude.ai app, the `comparison_card_display_v0` tool also works if attributes line up cleanly) |
| Full board verdict | End of every session | Inline chat — this is the synthesis, keep it conversational, not another document |
| Progress file (`startup-board-progress.md`) | Created after the Phase 5 verdict; updated after Phases 9 and 10 and whenever a pass test changes | Saved file — format and rules in **Progress file** below |
| Product/Market fit scorecard (`pmf-scorecard.md`) | End of Phase 9 | Saved file — format in **PMF scorecard** below |
| Growth plan (`growth-plan.md`) | End of Phase 10 | Saved file — format in **Growth plan** below |

General rule: a one-time read (verdict, quick questions) stays inline; anything the founder will return to and iterate on (canvas, research brief) becomes a file.

## Progress file

Lets a founder pick up where they left off in a later session.

**Name and location:** `startup-board-progress.md` in the founder's current folder (Claude Code). If that file already exists for a *different* idea, the new idea gets its own file: `startup-board-progress-<short-idea-name>.md`. In the claude.ai app, produce it as a file artifact and tell the founder to keep it; on their return, ask them to paste it in.

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
- **Read** it at the start of a session (see "Returning founders" in SKILL.md).
- **Create** it after the Phase 5 verdict. **Update** it after Phases 9 and 10, and whenever a pass test changes state.
- The evidence log is append-only: add dated rows, never rewrite old ones.
- Founder edits win: if the file differs from what you last wrote, trust the file.
- Store only the idea, stage, verdict, riskiest assumption, the numbers the founder gave, and the next action.
- When you save or update it, tell the founder the path in one line.

## PMF scorecard

```
# Product/Market fit scorecard: <product> — YYYY-MM-DD
> <gap stamp, only if built before an earlier stage was passed>
Active user: <definition> · Natural rhythm: <daily | weekly | monthly>

| Signal | Value | Real or estimated | Benchmark (rule of thumb) | Result |
|---|---|---|---|---|
| "Very disappointed" — all users | <x% of n responses> | Real | ≥40% from ~30+ responses | <Met | Not met | Directional> |
| "Very disappointed" — <segment> | <x% of n responses> | Real | ≥40% from ~30+ responses | <…> |
| Retention (<cohort>) | <period: % · period: %> | Real | Levels off above zero | <…> |
| Pull | <x% of new users without founder push, trend> | Real | Present or growing | <…> |

**Verdict:** <Not yet | In one segment — <segment> | Yes>
**Why:** <2–3 sentences using the numbers above>
**One next action:** <action>
**Not measured yet:** <signal — measurement kit given> (omit this line if everything was measured)
```

## Growth plan

```
# 90-day growth plan: <product> — YYYY-MM-DD
> <gap stamp, only if Market Fit hasn't been passed>
Sales Fit test: <met | not met — missing: …>

## 1. Channel experiment
Channel: <…> · Duration: <…> · Target: <n new customers> · Budget cap: <…>

## 2. Unit economics
| Metric | Working | Inputs (real / estimated) | Result |
|---|---|---|---|
| CAC | <spend + tools> ÷ <new customers> | <…> | <…> |
| LTV | <monthly revenue> × <margin> ÷ <monthly churn> | <…> | <… or "not measurable yet"> |
| LTV : CAC | LTV ÷ CAC | — | <… vs 3× rule of thumb> |
| Payback | CAC ÷ (<monthly revenue> × <margin>) | — | <n months vs ≤12 rule of thumb> |

## 3. Pricing move
<one move, or "Not covered yet">

## 4. Sales playbook (draft)
Lead source → First message → Demo / onboarding → Close → Follow-up
<the founder's steps, or "Not covered yet">

## 5. Hiring trigger
<automation first? · when · who · trial project, or "Not covered yet">
```
