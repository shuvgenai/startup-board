# Startup Board v1.1.0 — Fit Pass Tests, PMF Check-in, Scale & Growth — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend the Startup Board skill so it (a) tells founders when they've passed each fit stage, (b) runs a Product/Market fit check-in from pasted numbers, and (c) guides solo-founder scaling — with a progress file so returning founders resume where they left off.

**Architecture:** The skill is Markdown only: `SKILL.md` routes to reference files in `references/`. We add two reference files (`pmf-checkin.md`, `scale-growth.md`), extend three (`fit-framework.md`, `output-documents.md`, `gtm-launch.md`), add Phase 9/10 + a returning-founder rule to `SKILL.md`, and bump metadata. Behaviour is tested with role-play scenario files run by subagents (the skill equivalent of unit tests).

**Tech Stack:** Markdown (Claude Code skill + plugin format), PowerShell 5.1 for checks and git, Agent tool subagents for scenario tests.

**Spec:** `docs/superpowers/specs/2026-09-23-fit-and-scale-design.md`

**Repo root (all paths below are relative to it):** `E:\Projects\Startupboards_SKills\Startup board-v1` — branch `feature/fit-and-scale`. The untracked `promo/` folder is out of scope: never `git add` it.

## Global Constraints

- One advisor voice: never narrate what a named expert thinks, no "board"/"seat"/"panel" language to the founder. Naming a *method* (Mom Test, Lean Canvas, Sean Ellis test) is allowed.
- Exactly one question per advisor turn, shown as a box (markdown blockquote, or the multiple-choice question tool).
- Never invent numbers. Every figure is the founder's, a labelled rule of thumb, or shown arithmetic on those.
- One next action, never a to-do list.
- Phases 6–10 never auto-run for a founder who hasn't earned them; warn-then-allow applies, except Phase 9 with zero users (never runs).
- Founder numbers arrive by chat (typed/pasted). No CSV/file import.
- Scale scope is solo-founder only: up to the first hire. No growth teams, multi-channel budgets, or fundraising-linked growth.
- Pass-test thresholds (exact): Problem Fit ≥10 interviews, ~7 of 10 unprompted, ~half with a workaround · Product Fit ≥5 unaided core-loop completions + ≥3 real commitments (waitlist email doesn't count) · Customer Fit ≥4 weeks at natural cadence + ≥1 unprompted referral · Sean Ellis ≥40% "very disappointed" from ~30+ responses · Sales Fit 3 consecutive months predictable-cost channel, LTV ≥ 3× CAC, payback ≤ 12 months, a non-founder wins a customer with the playbook.
- Gap stamp text (exact): `> Built before <Stage> was passed. Missing: <evidence>.`
- Progress file name: `startup-board-progress.md` (a second idea in the same folder: `startup-board-progress-<short-idea-name>.md`).
- Version: `1.1.0`. `SKILL.md` frontmatter `description` ≤ 1024 characters; `plugin.json` description identical to it.

## Review Focus

1. **Progress file for a different idea than the founder brings** → the skill should ask whether to continue the saved idea or start fresh, and never overwrite the old file. *(Test: scenario s03, Task 3.)*
2. **Returning founder in the claude.ai app, where files aren't visible** → the skill should ask them to paste the progress file, not pretend to remember. *(Test: s04, Task 3.)*
3. **Too few survey responses (<30) that look above benchmark** → the skill should call it directional and not declare PMF. *(Test: s06, Task 4.)*
4. **Numbers that don't add up (survey % summing past 100%)** → the skill should flag it and ask one clarifying question before any verdict. *(Test: s09, Task 4.)*
5. **Zero churn / very short history** → the skill must not show an infinite LTV; it leaves LTV unmeasured or uses a capped, labelled estimate. *(Test: s11, Task 5.)*

---

## File Structure

| File | Responsibility | Task |
|---|---|---|
| `tests/scenarios/README.md` (create) | How to run and grade scenario tests; the advisor prompt; the invariants | 1 |
| `tests/scenarios/s01…s12-*.md` (create) | One role-play scenario each: setup, conversation, pass criteria | 1 |
| `skills/startup-board/references/fit-framework.md` (modify) | Six fit stages + pass tests + warn-then-allow | 2 |
| `skills/startup-board/references/output-documents.md` (modify) | Output formats: progress file, PMF scorecard, growth plan | 3, 4, 5 |
| `skills/startup-board/references/pmf-checkin.md` (create) | Phase 9 procedure | 4 |
| `skills/startup-board/references/scale-growth.md` (create) | Phase 10 procedure | 5 |
| `skills/startup-board/references/gtm-launch.md` (modify) | One pointer to Phases 9–10 | 5 |
| `skills/startup-board/SKILL.md` (modify) | Routing: triggers, returning founders, Phase 2/5 hooks, Phases 9–10 | 2, 3, 4, 5, 6 |
| `.claude-plugin/plugin.json` (modify) | Version + description | 6 |
| `README.md` (modify) | Public docs for v1.1.0 | 6 |

---

### Task 1: Scenario test harness

**Files:**
- Create: `tests/scenarios/README.md`
- Create: `tests/scenarios/s01-problem-fit-gate.md` … `tests/scenarios/s12-regression-first-turn.md` (12 files, content below)

**Interfaces:**
- Consumes: nothing.
- Produces: the scenario ids `s01`–`s12` and the "run a scenario" procedure every later task uses. Invariants are referred to as I1–I5.

- [ ] **Step 1: Create `tests/scenarios/README.md`**

````markdown
# Startup Board scenario tests

Skills are tested by role-play, not unit tests. Each scenario file holds a conversation so far and its pass criteria. A fresh subagent plays the advisor using the skill files exactly as they are on disk and writes the advisor's next message. You then grade that reply.

## Running a scenario

1. Dispatch a subagent with the Agent tool: `subagent_type: general-purpose`, `model: sonnet` (a mid-tier model is a stricter test of the written instructions). Use the **Advisor prompt** below, replacing `<SCENARIO FILE>` with the scenario's absolute path.
2. Grade the reply against the scenario's **Pass criteria** and every **Invariant** below. Every item must hold for PASS.
3. Record in chat: scenario id, PASS or FAIL, and for each failed item a short quote from the reply.

Independent scenarios can run in parallel (several Agent calls in one message).

## Advisor prompt

```
You are helping test a Claude skill called Startup Board. The skill lives at:
E:\Projects\Startupboards_SKills\Startup board-v1\skills\startup-board\
Read SKILL.md, then read whichever files in references/ SKILL.md tells you to use for this situation.

Then read the scenario at <SCENARIO FILE>. Play the advisor exactly as the skill instructs and write ONLY the advisor's next message, replying to the last founder turn. Simulation rules:
- Environment: use the one named in the scenario's Setup (Claude Code or the claude.ai app).
- Files in the founder's folder are listed in Setup with their contents. Treat them as readable only if that environment could read them. Do not read or write any real files other than the skill files and the scenario file.
- If the skill would save or update a file, do not write it. Put its full contents in your reply as a fenced block preceded by a line `FILE: <filename>`.
- If the skill would use a multiple-choice question tool, render it as:
  [QUESTION TOOL] <question>
  - <option>
  - <option>
- Today's date is the one in Setup.
- Do not explain your reasoning or mention that this is a test. Output only the advisor's message.
```

## Invariants (apply to every scenario)

- **I1** At most one question to the founder, shown as a box: a markdown blockquote or a `[QUESTION TOOL]` block. Never a numbered or bulleted list of questions.
- **I2** One advisor voice: no narrating what a named expert thinks ("Fitzpatrick would say…"), and no "board", "seat" or "panel" language addressed to the founder. Naming a method (Mom Test, Lean Canvas, Sean Ellis test) is fine.
- **I3** No invented numbers: every figure comes from the founder, the scenario, a rule of thumb labelled as one, or arithmetic on those with the working shown.
- **I4** When an action is recommended, it is one next action — not a to-do list.
- **I5** A brief, warm acknowledgement of what the founder just said comes before any question.
````

- [ ] **Step 2: Create `tests/scenarios/s01-problem-fit-gate.md`**

```markdown
# s01 — Problem Fit gate with 4 interviews

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** I want to validate an idea: a scheduling app for solo dog walkers.

**Advisor:** Nice — scheduling is a daily headache in service businesses. Let's pin down who it's for.
> **Who exactly is this for?**

**Founder:** Independent dog walkers with 10–30 regular clients and no staff.

**Advisor:** Got it — one-person businesses, so every missed walk is lost income.
> **What problem does it solve for them today?**

**Founder:** Double bookings and last-minute changes. They juggle WhatsApp messages and a paper diary.

**Advisor:** That's a concrete pain.
> **What stage are you at — just an idea, a prototype, or paying customers?**

**Founder:** Just the idea. I have a Figma mockup.

**Advisor:** Thanks — a mockup is a useful start.
> **How many dog walkers have you actually talked to about how they schedule today?**

**Founder:** 4. Three of them brought up double bookings on their own. Two pay a friend to help with admin; the others just use WhatsApp and paper. So — where do I stand? Can I start building?

## Pass criteria
- States the founder is at Problem Fit and has not passed it yet.
- Names the gap in numbers against the pass test: 4 interviews versus about 10.
- Recognises the evidence they do have: 3 of 4 raised it unprompted, and existing workarounds (paying a friend, WhatsApp/paper).
- Gives exactly one next action (more interviews with this segment).
- Does not produce a build blueprint, tech stack or feature list.
- If it offers to help build anyway, it says that output will carry the "Built before Problem Fit was passed" note.
```

- [ ] **Step 3: Create `tests/scenarios/s02-returning-founder.md`**

````markdown
# s02 — Returning founder with a progress file

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: `startup-board-progress.md` with this content:

```
# Startup Board progress: WalkSync — scheduling for solo dog walkers
Last updated: 2026-08-30
Current stage: Problem Fit (not yet passed: 4 of ~10 interviews)
Latest verdict: Talk to 6 more dog walkers before building
Riskiest assumption: Solo walkers lose enough money to double bookings to pay for a fix

## Evidence log
| Date | Stage | Signal | Value | Real or estimated |
|---|---|---|---|---|
| 2026-08-30 | Problem Fit | Interviews done | 4 | Real |
| 2026-08-30 | Problem Fit | Raised the problem unprompted | 3 of 4 | Real |

## Next action
Interview 6 more solo dog walkers using the Mom Test question set
```

## Conversation so far
**Founder:** Hi, I'm back. I've done 7 more interviews since last time.

## Pass criteria
- Shows it read the progress file: a one-line recap naming WalkSync (or dog-walker scheduling), Problem Fit, and the previous gap.
- Does not re-ask intake already answered by the file (what the product is, who it's for, stage).
- Uses the new total correctly: 11 interviews, now at or past the ~10 in the pass test.
- Does not declare Problem Fit passed on interview count alone; asks one question about the rest of the test (how many of the new 7 raised the problem unprompted, or use a workaround).
````

- [ ] **Step 4: Create `tests/scenarios/s03-progress-other-idea.md`**

````markdown
# s03 — Progress file belongs to a different idea

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: `startup-board-progress.md` with this content:

```
# Startup Board progress: WalkSync — scheduling for solo dog walkers
Last updated: 2026-08-30
Current stage: Problem Fit (not yet passed: 4 of ~10 interviews)
Latest verdict: Talk to 6 more dog walkers before building
Riskiest assumption: Solo walkers lose enough money to double bookings to pay for a fix

## Evidence log
| Date | Stage | Signal | Value | Real or estimated |
|---|---|---|---|---|
| 2026-08-30 | Problem Fit | Interviews done | 4 | Real |

## Next action
Interview 6 more solo dog walkers using the Mom Test question set
```

## Conversation so far
**Founder:** I want to validate a new idea: an app that helps landlords track repair requests.

## Pass criteria
- Notices the saved progress is for a different idea (WalkSync).
- Asks exactly one question: continue WalkSync, or start fresh on the landlord idea.
- Does not modify or overwrite the WalkSync progress file; if it mentions a file for the new idea, it uses a separate name.
- Does not start the landlord intake mixed with WalkSync facts.
````

- [ ] **Step 5: Create `tests/scenarios/s04-claude-ai-paste.md`**

```markdown
# s04 — Returning founder in the claude.ai app

## Setup
- Environment: claude.ai app (the advisor cannot see the founder's files)
- Date: 2026-09-23
- Files in founder's folder: not visible in this environment

## Conversation so far
**Founder:** Hey, I used Startup Board last month for my dog-walker scheduling idea. Can we pick up where we left off?

## Pass criteria
- Does not claim to remember the previous session or invent details from it.
- Asks the founder, as its one question, to paste their `startup-board-progress.md` (or its contents).
- Does not restart the full intake in the same message.
```

- [ ] **Step 6: Create `tests/scenarios/s05-pmf-segment.md`**

```markdown
# s05 — PMF check-in: fit in one segment

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** We launched InvoiceNudge 4 months ago — it chases late invoices for freelancers. 210 weekly active users. Do we have product-market fit?

**Advisor:** Good moment to check — four months of real usage is enough to look at.
> **What counts as an "active user" for InvoiceNudge, and how often would a real user naturally use it?**

**Founder:** Someone who sends at least one reminder in a week. Weekly is the natural rhythm.

**Advisor:** Weekly makes sense for invoicing.
> **Have you asked your active users how they'd feel if they could no longer use InvoiceNudge — very, somewhat, or not disappointed?**

**Founder:** Yes. 88 responses: 32% very disappointed. Split by type: agency designers (41 responses) 46% very disappointed; developers (30 responses) 23%; everyone else (17 responses) 18%.

**Advisor:** That split is really useful.
> **For users who started in the same week, how many are still active later on?**

**Founder:** May cohort: week 1 100%, week 4 52%, week 8 44%, week 12 42%.

**Advisor:** Thanks — that's a clear curve.
> **Where did last month's new users come from?**

**Founder:** 30% came from referrals or word of mouth, up from 18% two months ago. The rest from my LinkedIn posts. About 60 users pay $12/month and almost all renew. That's everything I have — what's the verdict?

## Pass criteria
- Verdict is "In one segment" (agency designers) — not "Yes", not "Not yet".
- Uses the founder's numbers: 46% of 41 agency-designer responses meets the ≥40% / ~30+ rule of thumb; 32% overall does not.
- Reads retention as levelling off (around 42–44%), not sliding to zero.
- Notes pull is growing (18% → 30%).
- Exactly one next action: focus on agency designers.
- Includes `FILE: pmf-scorecard.md` with one row per signal (value, real or estimated, result), the verdict, and the one next action.
- Includes `FILE: startup-board-progress.md` (created, since none existed) with dated evidence rows for this check-in.
```

- [ ] **Step 7: Create `tests/scenarios/s06-pmf-small-sample.md`**

```markdown
# s06 — PMF check-in: too few survey responses

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** I launched ShiftMeals — prepared meals delivered for night-shift nurses — 10 weeks ago. 34 weekly active customers. Do we have product-market fit?

**Advisor:** Ten weeks of real orders is a good start.
> **What counts as an "active customer" for ShiftMeals, and how often would one naturally order?**

**Founder:** Someone who orders at least once a week. Weekly.

**Advisor:** Clear.
> **Have you asked your active customers how they'd feel if they could no longer use ShiftMeals — very, somewhat, or not disappointed?**

**Founder:** 19 people answered: 47% said very disappointed. That's above 40%, so we have product-market fit, right?

## Pass criteria
- Does not confirm product-market fit.
- Explains that 19 responses is below the ~30 rule of thumb, so 47% is directional, not proof.
- Suggests getting more responses (e.g. surveying more of the 34 active customers).
- Continues with at most one question (the next signal, or widening the survey).
- Gives no final verdict of "Yes".
```

- [ ] **Step 8: Create `tests/scenarios/s07-pmf-missing-retention.md`**

```markdown
# s07 — PMF check-in: several signals at once, retention not tracked

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** I launched FocusFrame, a Chrome extension for deep-work sessions, 3 months ago. 400 weekly active users. Do we have product-market fit?

**Advisor:** Three months and 400 weekly users — worth a proper look.
> **What counts as an "active user" for FocusFrame, and how often would a real user naturally use it?**

**Founder:** Anyone who starts at least one focus session a week. Also — I already ran the survey: 64 responses, 41% very disappointed. And 45% of new installs last month came from word of mouth. We don't track retention at all.

## Pass criteria
- Accepts the active-user definition, survey and pull numbers without asking for them again.
- Does not estimate or assume any retention number.
- Gives a measurement kit for retention: a cohort table layout (start week, users who started, active at later weeks) and plain guidance on where the dates usually live.
- Does not give a final "Yes" verdict while retention is unmeasured (may say the signals so far look promising).
- At most one question.
```

- [ ] **Step 9: Create `tests/scenarios/s08-pmf-no-users.md`**

```markdown
# s08 — PMF asked before launch

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** My idea is a meal-kit app for night-shift nurses. I haven't launched yet — no users, just a waitlist of 120 emails. Do I have product-market fit?

## Pass criteria
- Explains that product-market fit can't be measured yet because there are no active users.
- Does not run the check-in: no request for survey, retention or pull numbers.
- Points out a waitlist email isn't a real commitment, and/or routes the founder to their actual stage with one next action.
- At most one question.
```

- [ ] **Step 10: Create `tests/scenarios/s09-pmf-inconsistent-numbers.md`**

```markdown
# s09 — PMF check-in: survey numbers don't add up

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** We launched RotaPal, shift-swapping for restaurant staff, 5 months ago. 150 weekly active users. Do we have product-market fit?

**Advisor:** Five months of usage is plenty to check.
> **What counts as an "active user" for RotaPal, and how often would one naturally use it?**

**Founder:** Anyone who views or swaps a shift in a week. Weekly.

**Advisor:** Makes sense for rotas.
> **Have you asked your active users how they'd feel if they could no longer use RotaPal — very, somewhat, or not disappointed?**

**Founder:** 52 responses: 55% very disappointed, 50% somewhat disappointed, 10% not disappointed.

## Pass criteria
- Points out that the percentages add up to 115%, so something is off.
- Asks one clarifying question before using the survey result.
- Gives no verdict and does not treat 55% as passing the benchmark.
```

- [ ] **Step 11: Create `tests/scenarios/s10-scale-before-pmf.md`**

```markdown
# s10 — Scale requested before product-market fit

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** I run TutorLoop, a booking tool for private tutors. 40 paying customers at $29/month. I want to scale — run Facebook ads and hire a salesperson.

**Advisor:** Forty paying tutors is real traction — congratulations.
> **Have you asked your active customers how they'd feel if they could no longer use TutorLoop — very, somewhat, or not disappointed?**

**Founder:** Yes, 52 responses, 22% very disappointed. Look, I know. Last month I spent $1,200 on ads and got 8 new customers, no other tool costs. Gross margin is roughly 85% — that's my guess. Monthly churn is 6%. Just give me the growth plan.

## Pass criteria
- Warns first: 22% is below the ~40% Market Fit benchmark; scaling before people love the product mostly buys churn.
- Then helps anyway (does not refuse).
- Includes `FILE: growth-plan.md` whose first content line after the title is the stamp `> Built before Market Fit was passed. Missing: …`.
- CAC shown as $1,200 ÷ 8 = $150.
- LTV shown as $29 × 0.85 ÷ 0.06 ≈ $411 (accept $410–$411).
- LTV : CAC ≈ 2.7, flagged as below the 3× rule of thumb.
- Payback ≈ 6.1 months ($150 ÷ $24.65).
- Gross margin labelled estimated; ad spend, customers, price and churn labelled real.
- Does not recommend hiring a salesperson now: asks whether automation can handle it first and/or ties the hire to a trigger.
- States the Sales Fit test is not met.
```

- [ ] **Step 12: Create `tests/scenarios/s11-scale-zero-churn.md`**

````markdown
# s11 — Unit economics with zero churn

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: `startup-board-progress.md` with this content:

```
# Startup Board progress: ClinicSlot — booking for physio clinics
Last updated: 2026-09-01
Current stage: Market Fit (passed: Phase 9 verdict Yes)
Latest verdict: Product-market fit — Yes
Riskiest assumption: Clinics will keep paying once the novelty wears off

## Evidence log
| Date | Stage | Signal | Value | Real or estimated |
|---|---|---|---|---|
| 2026-09-01 | Market Fit | Very disappointed | 44% of 36 responses | Real |

## Next action
Start Phase 10: pick one channel to prove
```

## Conversation so far
**Founder:** Back for growth help. We have 12 clinics at $49/month, launched 3 months ago, and nobody has cancelled — so churn is 0%. I spent $900 on ads last quarter and that got all 12. Margin is about 90%. What's my LTV and CAC?

## Pass criteria
- CAC = $900 ÷ 12 = $75, with the working shown.
- Does not present LTV as infinite or divide by 0% churn.
- Explains that 3 months with no cancellations isn't enough history to measure churn.
- Either leaves LTV as "not measurable yet" or uses a capped 24-month lifetime clearly labelled estimated ($49 × 0.9 × 24 = $1,058.40).
- One next action / at most one question.
````

- [ ] **Step 13: Create `tests/scenarios/s12-regression-first-turn.md`**

```markdown
# s12 — Regression: first turn of a fresh idea

## Setup
- Environment: Claude Code
- Date: 2026-09-23
- Files in founder's folder: none

## Conversation so far
**Founder:** I want to validate an idea for an app that helps freelancers chase late invoices.

## Pass criteria
- Brief, warm acknowledgement, then exactly one boxed intake question (who it's for, the problem, or stage).
- No documents, no verdict, no fit-stage claims, no numbers.
- Does not mention progress files, phases or internal structure.
```

- [ ] **Step 14: Validate the harness with the regression scenario (baseline must PASS)**

Run: dispatch one Agent per the README with `<SCENARIO FILE>` = `E:\Projects\Startupboards_SKills\Startup board-v1\tests\scenarios\s12-regression-first-turn.md`.
Expected: PASS against s12 criteria and I1–I5 (the current v1.0.1 skill already does this). If it FAILS, fix the README prompt wording (not the skill) until the harness produces a sensible advisor reply.

- [ ] **Step 15: Commit**

```powershell
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" add tests/scenarios
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" commit -m "test: add Startup Board role-play scenario harness (s01-s12)"
```

---

### Task 2: Pass tests and warn-then-allow (fit-framework.md + Phase 2)

**Files:**
- Modify: `skills/startup-board/references/fit-framework.md` (full replacement below)
- Modify: `skills/startup-board/SKILL.md` — Phase 2 paragraph (line 67)
- Test: `tests/scenarios/s01-problem-fit-gate.md`

**Interfaces:**
- Consumes: scenario procedure from Task 1.
- Produces: the section heading `## When a pass test isn't met — warn, then allow` and the exact gap stamp `> Built before <Stage> was passed. Missing: <evidence>.` — Tasks 3–5 reference both by name.

- [ ] **Step 1: Run s01 against the current skill (expect FAIL)**

Run: dispatch the advisor Agent for `tests\scenarios\s01-problem-fit-gate.md`.
Expected: FAIL — at minimum the "4 interviews versus about 10" criterion, because v1.0.1 has no numeric pass test. Record which criteria failed.

- [ ] **Step 2: Replace `skills/startup-board/references/fit-framework.md` with:**

```markdown
# The Six Fits

Use this to locate the founder honestly, not optimistically. Most fresh ideas are at Problem Fit or Research Fit — say so plainly.

Each stage ends with a **Pass test**. Pass tests count what people have *done* — past behaviour, money or time spent, real commitments — never opinions or "I would use that." The numbers are rules of thumb: say so when you use them, and adjust out loud when the context demands it (e.g. "with only ~40 possible buyers in this niche, 6 interviews is a reasonable bar").

## 1. Problem Fit (Problem-Solution Fit)
Validate the problem exists and is painful enough that people already try to solve it.
- Customer discovery & empathy: interviews with no pitching, focused on current workflow and pain.
- Problem scoping: "nice-to-have annoyance" vs. "hair-on-fire" problem.
- Root cause analysis: past the symptom to the real underlying issue.

**Pass test:** at least **10** Mom Test–style interviews with the target segment; about **7 of 10** describe the problem from their own recent experience without being prompted; about **half** already spend time or money on a workaround (a spreadsheet, a paper diary, paid help, another tool).

## 2. Research Fit
Bridge from "I think there's a problem" to objective data worth building on.
- Competitive intelligence: how people solve this today, including manual/DIY alternatives.
- Hypothesis generation: turn vision into testable if-then statements.
- Data synthesis: filter interview notes + market data into decisions.

**Pass test:** the founder can name how people solve it today (including manual/DIY) and the **top 3 alternatives**, and has written the riskiest assumption as an **if-then hypothesis** with a test and a pass number.

## 3. Product Fit (Product-Solution Fit)
Prove *this specific product* solves the problem for a small core group.
- MVP scope management: strip to the core value loop, nothing else.
- Rapid prototyping: wireframes/mockups in front of users fast.
- UX: the core loop is usable without hand-holding.

**Pass test:** at least **5** target users complete the core loop **without help**, and at least **3** make a real commitment — a pre-order, deposit, signed letter of intent, or paid pilot. A waitlist email alone does not count.

## 4. Customer Fit
Find the exact profile that gets massive value and becomes an advocate.
- Persona development: psychographics and behavioral triggers, not just demographics.
- Cohort analysis: retention and usage depth over time.
- Feedback loop management: a standing channel for critique, not one-off comments.

**Pass test:** the founder can describe the **single highest-value customer profile**; that group keeps using the product at its natural rhythm (daily / weekly / monthly) for **4+ weeks**; and at least **1** of them has referred someone without being asked.

## 5. Market Fit (Product-Market Fit)
Move from a handful of happy early adopters to pull-driven demand.
- Market sizing (TAM/SAM/SOM) — is the addressable market actually big enough.
- PMF metric tracking: the "very disappointed" survey, retention, organic growth.
- Value proposition refinement: messaging so aligned the product starts selling itself.

**Pass test:** a Phase 9 verdict of **Yes** (see `pmf-checkin.md`). A verdict of *In one segment* passes Market Fit for that segment only.

## 6. Sales Fit (Go-To-Market Fit)
Move from founder-led sales to a repeatable, profitable acquisition machine.
- Sales playbook: a repeatable pipeline a non-founder could run.
- Unit economics: CAC vs. LTV.
- Channel experimentation: test channels one at a time, then scale what works, rather than spreading thin across all of them.

**Pass test (measured in Phase 10, see `scale-growth.md`):** one channel has delivered customers at a predictable cost for **3 consecutive months**; **LTV ≥ 3× CAC**; CAC payback **≤ 12 months**; and **someone other than the founder** has won a customer using the playbook.

## When a pass test isn't met — warn, then allow

1. Say which test isn't met and exactly what evidence is missing, in numbers: "You've done 4 interviews; the Problem Fit test needs about 10."
2. Name the **one** action that closes the gap.
3. If the founder still wants to move ahead, help them — and put this line at the top of any document you produce for a later stage:

   `> Built before <Stage> was passed. Missing: <evidence>.`

Never hide the gap, and never refuse. The one exception is Phase 9 with zero active users: there is nothing to measure, so explain why and return the founder to their real stage (see `pmf-checkin.md`).

## How to use this in the board session
State which stage the founder is actually at, then name the *one* unblocking action for that stage — not a to-do list across all six. A founder at Problem Fit doesn't need a GTM playbook yet; they need five more customer conversations.

Count the evidence the founder has actually given you against the pass test for their current stage. If a piece of evidence hasn't come up, ask for it (one question) rather than assuming it either way.
```

- [ ] **Step 3: Edit the Phase 2 paragraph in `skills/startup-board/SKILL.md`**

Replace:
```
Score the idea against the six fit stages from `references/fit-framework.md` (Problem Fit → Research Fit → Product Fit → Customer Fit → Market Fit → Sales Fit). State plainly which stage the founder is actually at (usually Problem Fit or Research Fit for a fresh idea — say so even if it's not what they want to hear) and what the single next unblocking action is. This can be a direct statement rather than a question, since it's a synthesis, not an interrogation — but keep it short and let the founder respond before moving on.
```
with:
```
Score the idea against the six fit stages from `references/fit-framework.md` (Problem Fit → Research Fit → Product Fit → Customer Fit → Market Fit → Sales Fit). State plainly which stage the founder is actually at (usually Problem Fit or Research Fit for a fresh idea — say so even if it's not what they want to hear) and what the single next unblocking action is. Use each stage's **pass test** in that file: count the evidence the founder has actually given you and name any gap in numbers ("4 of the ~10 interviews the test needs"). If the founder wants to move ahead anyway, follow the file's *warn, then allow* rule. This can be a direct statement rather than a question, since it's a synthesis, not an interrogation — but keep it short and let the founder respond before moving on.
```

- [ ] **Step 4: Run s01 again (expect PASS) and s12 (regression, expect PASS)**

Run: dispatch both advisor Agents in parallel for `s01-problem-fit-gate.md` and `s12-regression-first-turn.md`.
Expected: both PASS all criteria and I1–I5. If s01 fails, tighten the wording in `fit-framework.md` (not the scenario) and re-run.

- [ ] **Step 5: Commit**

```powershell
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" add skills/startup-board/references/fit-framework.md skills/startup-board/SKILL.md
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" commit -m "feat: add pass tests and warn-then-allow to the six fit stages"
```

---

### Task 3: Progress file and returning founders

**Files:**
- Modify: `skills/startup-board/references/output-documents.md` (table row + new section)
- Modify: `skills/startup-board/SKILL.md` — new section before `### Phase 0`, Phase 5 paragraph
- Test: `tests/scenarios/s02-returning-founder.md`, `s03-progress-other-idea.md`, `s04-claude-ai-paste.md`

**Interfaces:**
- Consumes: gap stamp and stage names from Task 2.
- Produces: the section heading `## Progress file` in `output-documents.md` and the file name `startup-board-progress.md`; Tasks 4–5 say "append dated rows to the progress file" and point here.

- [ ] **Step 1: Run s02, s03, s04 against the current skill (expect FAIL)**

Run: dispatch three advisor Agents in parallel.
Expected: FAIL on s02 (no recap of the file) and s04 (no paste request); s03 likely FAIL (no different-idea check). Record failures.

- [ ] **Step 2: Add a row to the table in `output-documents.md`**

Insert after the line starting `| Full board verdict |`:
```
| Progress file (`startup-board-progress.md`) | Created after the Phase 5 verdict; updated after Phases 9 and 10 and whenever a pass test changes | Saved file — format and rules in **Progress file** below |
```

- [ ] **Step 3: Append the Progress file section to the end of `output-documents.md`**

````markdown

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
````

- [ ] **Step 4: Insert the returning-founders section in `SKILL.md`**

Insert immediately before the line `### Phase 0 — Intake (always first, unless the founder already answered these in-thread)`:
```
### Returning founders — check for a progress file first

Before Phase 0, look for `startup-board-progress.md` (format and rules in `references/output-documents.md`). In Claude Code, check the current folder. In the claude.ai app you can't see the founder's files — if they say they've used Startup Board before, ask them (one boxed question) to paste their progress file. Never pretend to remember a past session.

If you find one: give a one-line recap (idea, current stage, the gap or next action), then continue at the recorded stage — don't re-ask intake questions the file already answers. If the founder brings a different idea than the one in the file, ask one question: continue the saved idea, or start fresh on the new one. A new idea gets its own file; never overwrite the old one.

```

- [ ] **Step 5: Edit the Phase 5 paragraph in `SKILL.md`**

Replace:
```
Mention plainly that build, launch, naming, and pitch-deck help are available whenever the founder wants them — don't auto-run Phases 6–8 unless they're clearly past validation or ask directly.
```
with:
```
Mention plainly that build, launch, naming, and pitch-deck help are available whenever the founder wants them, and that after launch they can come back with real numbers for a product-market fit check-in and help scaling — don't auto-run Phases 6–10 unless they're clearly past validation or ask directly.

Then save the session to the progress file — create it, or update it if one exists (see `references/output-documents.md`) — and tell the founder its path in one line.
```

- [ ] **Step 6: Run s02, s03, s04, s12 (expect PASS)**

Run: dispatch four advisor Agents in parallel.
Expected: all PASS. Fix wording in the section added in Step 4 if not.

- [ ] **Step 7: Commit**

```powershell
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" add skills/startup-board/references/output-documents.md skills/startup-board/SKILL.md
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" commit -m "feat: add progress file so returning founders resume where they left off"
```

---

### Task 4: Phase 9 — Product/Market fit check-in

**Files:**
- Create: `skills/startup-board/references/pmf-checkin.md`
- Modify: `skills/startup-board/references/output-documents.md` (table row + scorecard format)
- Modify: `skills/startup-board/SKILL.md` — "When this skill fires" list, downstream intro, new Phase 9 section
- Test: `s05-pmf-segment.md`, `s06-pmf-small-sample.md`, `s07-pmf-missing-retention.md`, `s08-pmf-no-users.md`, `s09-pmf-inconsistent-numbers.md`

**Interfaces:**
- Consumes: `## Progress file` (Task 3), warn-then-allow + Market Fit pass test (Task 2).
- Produces: verdict labels exactly *Not yet* / *In one segment* / *Yes*; file name `pmf-scorecard.md`; heading `### Phase 9 — Product/Market Fit Check-in` in SKILL.md. Task 5 keys off the *Yes* / *In one segment* labels.

- [ ] **Step 1: Run s05 and s08 against the current skill (expect FAIL)**

Run: dispatch two advisor Agents in parallel.
Expected: FAIL — no `pmf-scorecard.md`, no "In one segment" verdict (s05); likely asks for numbers or hand-waves instead of the no-users rule (s08).

- [ ] **Step 2: Create `skills/startup-board/references/pmf-checkin.md`:**

```markdown
# Phase 9 — Product/Market Fit Check-in

For a founder with a live product and real users. The goal is an honest verdict — *Not yet*, *In one segment*, or *Yes* — built only from numbers the founder gives you. One boxed question per turn, as always.

## Before you start
- **No active users → don't run it.** Explain that product-market fit is measured from how real users behave, so there's nothing to measure yet. A waitlist is not usage. Return the founder to their real stage (usually Product Fit — see `fit-framework.md`) with one next action.
- If a progress file exists, read the last check-in so you can report the trend.
- If the founder has already given some numbers, don't ask for them again — go straight to what's missing.

## Step 1 — Define "active user"
Ask what counts as an active user for this product and how often a real user would naturally use it: daily, weekly or monthly. Every later number uses this definition.

## Step 2 — Collect three signals, one question each

### Signal A — the "very disappointed" survey (Sean Ellis test)
Asked of active users: "How would you feel if you could no longer use [product]?" — *Very disappointed / Somewhat disappointed / Not disappointed*.
- Benchmark (rule of thumb): **40% or more "very disappointed"**, from about **30+ active-user responses**.
- Fewer than ~30 responses: call the result *directional*, not proof, and suggest surveying more active users.
- Ask for splits by customer type if they have them — this feeds the segment check.

### Signal B — retention
For a group of users who started in the same week or month, what share is still active at a few later points (e.g. week 1 / 4 / 8, or month 1 / 2 / 3)?
- Healthy: the curve **levels off above zero** — it stops falling.
- Warning: it keeps sliding toward zero.
- Judge the shape. Don't invent a target percentage.

### Signal C — pull
What share of last period's new users or customers arrived without the founder pushing (word of mouth, referrals, organic search) — and is that share growing? If the product charges money: are customers paying and renewing?

## Numbers that don't add up
If figures conflict — survey percentages summing to more than 100%, more responses than active users, a cohort that grows over time — say exactly what doesn't add up and ask one clarifying question before using them. Never "fix" the numbers yourself.

## Missing signals — the measurement kit
Never estimate a missing signal. Give the founder what they need to measure it, then ask them to come back with results:
- **Survey:** the question above, plus "What type of person do you think would benefit most from [product]?", "What is the main benefit you get from [product]?" and "How could we improve [product] for you?" Send it to active users only.
- **Retention table** — one row per start week (or month):

  | Start week | Users who started | Active week 1 | Active week 4 | Active week 8 |
  |---|---|---|---|---|

- **Where the numbers usually live:** sign-up and last-active dates in the app's database or login tool; payments and cancellations in the payment tool; a "How did you hear about us?" question at sign-up for pull.

## Step 3 — Segment check
If the founder gave survey results by customer type, compare each segment with the benchmark. A segment at or above 40% (with ~30+ responses) while the overall figure is below → *In one segment*. A segment above 40% with fewer than ~30 responses can still earn *In one segment*, labelled directional, with a suggestion to survey more of that segment.

## Step 4 — Verdict and one next action

| Verdict | When | One next action |
|---|---|---|
| *Not yet* | Survey below benchmark overall and in every segment, or retention keeps sliding toward zero | The single product change most likely to win over "somewhat disappointed" users, based on what "very disappointed" users value most |
| *In one segment* | A segment meets the survey benchmark and its retention isn't collapsing | Focus product, messaging and outreach on that segment |
| *Yes* | Survey benchmark met overall, retention levels off, and pull is present or growing | Offer Phase 10 (`scale-growth.md`) |

Never give *Yes* while a signal is unmeasured: say the signals so far look promising, name the missing one, and hand over the measurement kit.

## Output
Save `pmf-scorecard.md` (format in `output-documents.md`) and append dated rows to the progress file (create it if none exists). If a previous check-in exists, state the trend in one line ("up from 28% to 36% since your last check-in"). Mark Market Fit as passed in the progress file only on *Yes* — or for that segment only, on *In one segment*.
```

- [ ] **Step 3: Add the scorecard row and format to `output-documents.md`**

Insert after the `| Progress file (...` table row:
```
| Product/Market fit scorecard (`pmf-scorecard.md`) | End of Phase 9 | Saved file — format in **PMF scorecard** below |
```

Append to the end of the file:
````markdown

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
````

- [ ] **Step 4: Update `SKILL.md` — triggers, downstream intro, Phase 9 section**

(a) In "When this skill fires", insert after the line starting `- "how should I build the MVP"`:
```
- "do I have product-market fit" / "are people actually using this" / "am I ready to scale" / "which channel should I double down on" / "what's my CAC / LTV" / "should I hire" / "write me a sales playbook"
```

(b) Replace:
```
Phases 0–5 are the validation core and run on essentially every session. Phases 6–8 (Build, Launch, Pitch/Fundraising) are downstream extensions — run them when the founder is past validation or explicitly asks for that stage; don't front-load a build blueprint onto an idea that hasn't cleared Phase 2 yet.
```
with:
```
Phases 0–5 are the validation core and run on essentially every session. Phases 6–10 (Build, Launch, Pitch/Fundraising, Product/Market Fit Check-in, Scale & Growth) are downstream extensions — run them when the founder is past validation or explicitly asks for that stage; don't front-load a build blueprint onto an idea that hasn't cleared Phase 2 yet.
```

(c) Insert after the Phase 8 paragraph (the line ending `...and likely investor objections.`) and before `## Style notes`:
```

### Phase 9 — Product/Market Fit Check-in

For founders with a live product and real users — on request ("do I have product-market fit?") or when the progress file shows they've launched. Use `references/pmf-checkin.md`: define an active user, collect three signals one boxed question at a time (the "very disappointed" survey, retention, pull), then give a verdict of *Not yet*, *In one segment* or *Yes* with one next action. Never estimate a missing signal — hand over the measurement kit instead. With zero active users, don't run it: explain why and return the founder to their real stage.
```

- [ ] **Step 5: Run s05–s09 and s12 (expect PASS)**

Run: dispatch six advisor Agents in parallel.
Expected: all PASS. Fix `pmf-checkin.md` wording for any failure and re-run that scenario.

- [ ] **Step 6: Commit**

```powershell
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" add skills/startup-board/references/pmf-checkin.md skills/startup-board/references/output-documents.md skills/startup-board/SKILL.md
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" commit -m "feat: add Phase 9 product-market fit check-in"
```

---

### Task 5: Phase 10 — Scale & growth

**Files:**
- Create: `skills/startup-board/references/scale-growth.md`
- Modify: `skills/startup-board/references/output-documents.md` (table row + growth plan format)
- Modify: `skills/startup-board/references/gtm-launch.md` (append pointer)
- Modify: `skills/startup-board/SKILL.md` — new Phase 10 section
- Test: `s10-scale-before-pmf.md`, `s11-scale-zero-churn.md`

**Interfaces:**
- Consumes: Phase 9 verdict labels (Task 4), Sales Fit pass test + gap stamp (Task 2), progress file (Task 3).
- Produces: file name `growth-plan.md`; heading `### Phase 10 — Scale & Growth` in SKILL.md.

- [ ] **Step 1: Run s10 and s11 against the current skill (expect FAIL)**

Run: dispatch two advisor Agents in parallel.
Expected: FAIL — no gap stamp / `growth-plan.md` (s10); likely an unbounded or unlabelled LTV (s11).

- [ ] **Step 2: Create `skills/startup-board/references/scale-growth.md`:**

```markdown
# Phase 10 — Scale & Growth (solo-founder scale)

Goal: the founder's first repeatable growth engine — one channel that keeps working at a known cost, customers worth more than they cost to win, and a sales process someone else could run. Stop at the first hire; growth teams and multi-channel budgets are out of scope.

## Before you start
- Check the Market Fit pass test in `fit-framework.md`. If Phase 9 hasn't given *Yes* or *In one segment*, warn first: scaling before people love the product mostly buys more people who leave. Then follow *warn, then allow*: help, and put the gap stamp at the top of `growth-plan.md`.
- *In one segment* → keep all growth work inside that segment.
- First question: which of the five areas below to start with. Suggest the one where the founder's numbers look weakest. If the founder asks for the whole plan at once, produce it.

## 1. One repeatable channel
- Start from the channel that already brought the best customers, based on the founder's data — not the channel that's fashionable.
- Run it as an experiment: a duration (e.g. 4–6 weeks), a target number of new customers, and a budget cap.
- No second channel until the first one repeats — it keeps bringing customers at a predictable cost.

## 2. Unit economics — show the working
Use the founder's numbers and label every input **real** or **estimated**.
- **CAC** = (acquisition spend + tool costs for the period) ÷ new customers in the period
- **LTV** = average monthly revenue per customer × gross margin ÷ monthly churn rate
- **Payback (months)** = CAC ÷ (monthly revenue per customer × gross margin)
- **LTV : CAC** = LTV ÷ CAC

Rules of thumb (say they're rules of thumb): LTV at least **3× CAC**; payback **12 months or less**, ideally **6 or less** for a self-funded founder.

Edge cases:
- **Churn of 0%, or under 3 months of history:** don't divide by it. Say churn can't be measured yet. Either leave LTV as "not measurable yet", or use a capped 24-month customer lifetime (LTV = monthly revenue × gross margin × 24) clearly labelled *estimated*. Never show an infinite LTV.
- **Founder time:** leave it out of CAC unless the founder wants it in, but mention that founder-led sales hides the real cost.
- **A missing input:** ask for it (one question), or mark the result *estimated* with the assumption written next to it.

## 3. Pricing from real usage
- Price against the value customers actually care about (per client, per project, per seat…), found in survey "main benefit" answers and usage.
- Test price changes on new customers only.
- If nobody ever pushes back on price, it's probably too low.

## 4. Sales playbook
Write down how the founder actually wins customers today, in enough detail for someone else to follow: lead source → first message → demo or onboarding → close → follow-up. Use the founder's real steps and wording; mark any step that's still improvised.

## 5. First hire
- **Ask first:** can a no-code automation handle this? (scheduled follow-up emails, an onboarding sequence, a booking link)
- **When:** a proven, repeatable task takes a large share of the founder's week.
- **Who:** the role that removes the biggest founder bottleneck — often a part-time contractor first.
- **How:** a small paid trial project before any longer commitment.
- Don't recommend a hire for a channel or sales process that hasn't repeated yet.

## Sales Fit pass test
Met when one channel has delivered customers at a predictable cost for **3 consecutive months**, **LTV ≥ 3× CAC**, **payback ≤ 12 months**, and **someone other than the founder** has won a customer using the playbook. Say which parts are met and which aren't.

## Output
Save `growth-plan.md` (format in `output-documents.md`) — a 90-day plan — and append dated rows to the progress file.
```

- [ ] **Step 3: Add the growth-plan row and format to `output-documents.md`**

Insert after the `| Product/Market fit scorecard (...` table row:
```
| Growth plan (`growth-plan.md`) | End of Phase 10 | Saved file — format in **Growth plan** below |
```

Append to the end of the file:
````markdown

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
````

- [ ] **Step 4: Append a pointer to `skills/startup-board/references/gtm-launch.md`**

Append to the end of the file:
```markdown

## After launch
Once there's real usage, check product-market fit in Phase 9 (`pmf-checkin.md`) before scaling any channel from this plan, then use Phase 10 (`scale-growth.md`) to grow.
```

- [ ] **Step 5: Add the Phase 10 section to `SKILL.md`**

Insert immediately after the Phase 9 section added in Task 4 (before `## Style notes`):
```

### Phase 10 — Scale & Growth

On request, or after a Phase 9 verdict of *Yes* or *In one segment*. Use `references/scale-growth.md` — solo-founder scale only: one repeatable channel, unit economics (CAC, LTV, payback, with the working shown and every input labelled real or estimated), pricing, a sales playbook, and the first hire. Before product-market fit, warn first (the *warn, then allow* rule in `references/fit-framework.md`), then help. Same interaction rule: one boxed question at a time.
```

- [ ] **Step 6: Run s10, s11, s12 (expect PASS)**

Run: dispatch three advisor Agents in parallel.
Expected: all PASS. Check the arithmetic in s10 by hand ($150; ≈$411; ≈2.7; ≈6.1 months) and s11 ($75; $1,058.40 if capped). Fix `scale-growth.md` wording for any failure.

- [ ] **Step 7: Commit**

```powershell
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" add skills/startup-board/references/scale-growth.md skills/startup-board/references/output-documents.md skills/startup-board/references/gtm-launch.md skills/startup-board/SKILL.md
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" commit -m "feat: add Phase 10 scale and growth for solo founders"
```

---

### Task 6: Metadata, README, and full regression run

**Files:**
- Modify: `skills/startup-board/SKILL.md` — frontmatter `description` (line 3)
- Modify: `.claude-plugin/plugin.json` — `version`, `description`
- Modify: `README.md` — intro, version badge, "Build, launch, and raise" table, "How to use it" list, FAQ
- Test: all scenarios s01–s12

**Interfaces:**
- Consumes: everything above.
- Produces: release-ready v1.1.0 on the branch.

- [ ] **Step 1: Replace the `description:` line in `SKILL.md` frontmatter with exactly:**

```
description: End-to-end startup product creation system — runs a fully interactive, one-question-at-a-time advisory session with a founder to validate their idea, then carries it through market research, a Lean Canvas, an MVP build blueprint (no-code/prompt-first stack), go-to-market/launch plan, naming/branding, pitch/fundraising materials, a product-market fit check-in, and solo-founder scaling. Use whenever the user wants to validate a startup or product idea, pressure-test an MVP concept, get expert feedback, generate Mom Test-style interview questions, build a Lean Canvas, run market/industry research, plan an MVP build with no-code tools, write a go-to-market plan, name/brand a product, build a pitch deck, check product-market fit, or scale after launch (channels, CAC/LTV, pricing, first hire). Trigger even without the word "skill" or "board" — any request to take a product idea from problem through build, launch and growth routes here.
```

- [ ] **Step 2: Update `.claude-plugin/plugin.json`**

Set `"version": "1.1.0"` and set `"description"` to the exact same text as Step 1 (JSON-escape the inner double quotes as `\"skill\"` and `\"board\"`). Leave all other fields unchanged.

- [ ] **Step 3: Verify lengths, JSON validity and description match**

Run:
```powershell
$root = "E:\Projects\Startupboards_SKills\Startup board-v1"
$s = Get-Content -Raw "$root\skills\startup-board\SKILL.md"
$null = $s -match '(?m)^description: (.+)$'; $d = $Matches[1].Trim()
$j = Get-Content -Raw "$root\.claude-plugin\plugin.json" | ConvertFrom-Json
"SKILL description chars: $($d.Length)"
"plugin version: $($j.version)"
"descriptions identical: $($d -eq $j.description)"
```
Expected: chars ≤ 1024, `plugin version: 1.1.0`, `descriptions identical: True`. `ConvertFrom-Json` throwing means invalid JSON — fix it.

- [ ] **Step 4: Verify every reference file SKILL.md names exists**

Run:
```powershell
$root = "E:\Projects\Startupboards_SKills\Startup board-v1\skills\startup-board"
$names = Select-String -Path "$root\SKILL.md","$root\references\*.md" -Pattern '([a-z-]+\.md)' -AllMatches | ForEach-Object { $_.Matches | ForEach-Object { $_.Groups[1].Value } } | Sort-Object -Unique
$names | ForEach-Object { if (-not (Test-Path "$root\references\$_") -and $_ -notin @('SKILL.md','startup-board-progress.md','pmf-scorecard.md','growth-plan.md')) { "MISSING: $_" } }
"check done"
```
Expected: only `check done` (no `MISSING:` lines). Founder-output files (`startup-board-progress.md`, `pmf-scorecard.md`, `growth-plan.md`) are intentionally excluded.

- [ ] **Step 5: Update `README.md`**

(a) Badge — replace `version-1.0.1-green` with `version-1.1.0-green`.

(b) Intro — replace:
```
interviews you one question at a time, tells you honestly where your idea stands, and then helps you plan the MVP, launch, brand, and pitch.
```
with:
```
interviews you one question at a time, tells you honestly where your idea stands, and then helps you plan the MVP, launch, brand, and pitch — and, after launch, check product-market fit and grow.
```

(c) Replace the heading `### Build, launch, and raise (when you're ready or when you ask)` with `### Build, launch, raise, and grow (when you're ready or when you ask)`, and add two rows after the `| **8. Naming, branding & pitch** |` row:
```
| **9. Product/Market fit check-in** | Bring your real numbers after launch — the "very disappointed" survey, retention, and word-of-mouth growth — and get a clear verdict: not yet, in one segment, or yes |
| **10. Scale & growth** | One repeatable channel, customer acquisition cost vs lifetime value with the maths shown, pricing moves, a simple sales playbook, and when to make your first hire |
```
Then add this paragraph directly under that table:
```

Every stage has a clear "you've passed" test based on evidence, not opinions. Startup Board saves a small `startup-board-progress.md` file in your folder so it can pick up where you left off next time.
```

(d) In "How to use it", add after the `- "Help me launch this" / "Name this product" / "Build me a pitch deck"` line:
```
- "Do I have product-market fit?" / "How do I scale this?" / "What's my CAC and LTV?"
```

(e) In the FAQ, add before `**Is it free?**`:
```
**Can it tell me if I have product-market fit?**
Yes. After launch, bring your numbers — survey results, retention, where new users come from — and it gives you a verdict (not yet, in one segment, or yes) and one next step. If you're missing a number, it shows you how to measure it instead of guessing.

**Will it remember my idea next time?**
Yes. It saves a short `startup-board-progress.md` file in your folder with your stage, verdict, and numbers, and reads it when you come back. You can open and edit it yourself.

```

- [ ] **Step 6: Full regression — run all 12 scenarios (expect all PASS)**

Run: dispatch advisor Agents for s01–s12 (in batches of up to 6 parallel calls).
Expected: 12/12 PASS. For any FAIL, fix the owning reference file, re-run that scenario plus s12.

- [ ] **Step 7: Commit**

```powershell
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" add skills/startup-board/SKILL.md .claude-plugin/plugin.json README.md
git -C "E:\Projects\Startupboards_SKills\Startup board-v1" commit -m "release: v1.1.0 metadata, README for PMF check-in and scale phases"
```

---

## Follow-ups (not in this plan)

- Update the two promo films in `promo/` for v1.1.0.
- Merge `feature/fit-and-scale` and push to GitHub (only when the user asks).
