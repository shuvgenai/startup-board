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
