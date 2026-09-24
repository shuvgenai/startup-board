# Startup Board scenario tests

Skills are tested by role-play, not unit tests. Each file in `tests/scenarios/` holds only a setup and a conversation so far; its pass criteria live in the file of the same name in `tests/criteria/`. A fresh subagent plays the advisor using the skill files exactly as they are on disk and writes the advisor's next message. You then grade that reply.

The advisor must never see the pass criteria, the spec or the plan — a reply shaped by the answer key proves nothing. That's why criteria live in a separate folder and the advisor prompt forbids reading anything but the skill and its one scenario file.

## Running a scenario

1. Dispatch a subagent with the Agent tool: `subagent_type: general-purpose`, `model: sonnet` (a mid-tier model is a stricter test of the written instructions). Use the **Advisor prompt** below, replacing `<SCENARIO FILE>` with the scenario's absolute path.
2. Grade the reply against the matching `tests/criteria/` file and every **Invariant** below. Every item must hold for PASS.
3. Record in chat: scenario id, PASS or FAIL, and for each failed item a short quote from the reply.

Independent scenarios can run in parallel (several Agent calls in one message).

## Advisor prompt

```
You are helping test a Claude skill called Startup Board. The skill lives at:
E:\Projects\Startupboards_SKills\Startup board-v1\skills\startup-board\
Read SKILL.md, then read whichever files in references/ SKILL.md tells you to use for this situation.

Then read the scenario at <SCENARIO FILE>. You may read ONLY files inside the skill folder above and that one scenario file. Do not open, list or search anything else in the repository (no tests/criteria, docs, README.md or other scenarios). Play the advisor exactly as the skill instructs and write ONLY the advisor's next message, replying to the last founder turn. Simulation rules:
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
