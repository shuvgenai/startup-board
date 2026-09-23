# Startup Board — AI Startup Advisor for Idea Validation, Lean Canvas & Pitch Decks

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.1-green.svg)](.claude-plugin/plugin.json)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-D97757.svg)](https://github.com/shuvgenai/startup-board)

**Startup Board is a free, open-source Claude Code plugin that helps founders validate a startup idea before they build it.** One AI advisor — trained on eight proven startup frameworks like *The Mom Test*, *Lean Canvas*, and *Jobs-to-be-Done* — interviews you one question at a time, tells you honestly where your idea stands, and then helps you plan the MVP, launch, brand, and pitch.

---

## Contents

- [Why Startup Board](#why-startup-board)
- [How it works](#how-it-works)
- [The eight startup frameworks](#the-eight-startup-frameworks)
- [What a session looks like](#what-a-session-looks-like)
- [Install](#install)
- [How to use it](#how-to-use-it)
- [FAQ](#faq)
- [License](#license)

---

## Why Startup Board

Most founders skip validation and go straight to building. Startup Board is designed to stop that — kindly, but firmly.

- **A real conversation, not a form.** You get one clear question at a time, and the advisor reacts to your answer before asking the next.
- **Eight expert frameworks, one voice.** It reasons like a board of startup experts behind the scenes, but you only ever talk to one advisor.
- **Honest verdicts.** It will tell you to proceed, pivot, go talk to more customers — or drop the idea. No cheerleading.
- **Built for no-code founders.** Next steps are things you can do yourself: a landing-page test, ten customer calls, a quick survey — not "hire a growth team."
- **From idea to pitch in one place.** Validation, market research, Lean Canvas, MVP plan, go-to-market, naming, and pitch deck.

## How it works

### Validate the idea (every session)

| Phase | What happens |
|---|---|
| **0. Intake** | What you're building, who it's for, the problem it solves, and your stage |
| **1. Interrogation** | Sharp questions that pressure-test your riskiest assumptions |
| **2. Fit assessment** | Where you really are: Problem → Research → Product → Customer → Market → Sales fit |
| **3. Market research** | Your choice of depth: quick competitor scan, or full market sizing (TAM / SAM / SOM) |
| **4. Documents** | Lean Canvas, Mom Test customer interview questions, research brief |
| **5. Verdict** | Proceed, pivot, talk to more customers, or kill it — plus your single riskiest assumption |

### Build, launch, and raise (when you're ready or when you ask)

| Phase | What you get |
|---|---|
| **6. MVP build blueprint** | A no-code / prompt-first stack, a tight MVP feature cut, a data model sketch, and ready-to-use build prompts |
| **7. Go-to-market & launch** | Channels, positioning and messaging, pricing, and a launch-week checklist |
| **8. Naming, branding & pitch** | Name and tagline ideas, a pitch deck outline, a one-pager, and likely investor objections |

## The eight startup frameworks

Startup Board draws on the work of these authors. You never see them named in the conversation — their thinking shapes the questions you're asked.

| Framework | Author | Used for |
|---|---|---|
| Business Model Canvas | Alexander Osterwalder | Is the business model coherent? |
| Lean Canvas (*Running Lean*) | Ash Maurya | Mapping problem, solution, and untested assumptions |
| *The Mom Test* | Rob Fitzpatrick | Customer interviews that get honest answers |
| Lean Customer Development | Cindy Alvarez | Validating with real people, not founder vision |
| *Talking to Humans* | Giff Constable | Running customer conversations well |
| *Lean Analytics* | Alistair Croll & Benjamin Yoskovitz | The One Metric That Matters |
| AARRR (Pirate Metrics) | Dave McClure | Acquisition, activation, retention, revenue, referral |
| Jobs-to-be-Done | Clayton Christensen | What job the customer is really "hiring" you for |

## What a session looks like

*Illustrative example.*

> **You:** I want to validate an idea for an app that helps freelancers chase late invoices.
>
> **Startup Board:** Good problem space — late payment is a real pain for freelancers. Let's make sure we're solving the right slice of it.
>
> > **Who exactly is this for — which kind of freelancer, doing what work?**
>
> Your answer:

It keeps going like this, one question at a time, until it has enough to give you a fit assessment and a verdict.

## Install

**Requires [Claude Code](https://docs.claude.com/en/docs/claude-code/overview).**

Inside Claude Code, run:

```
/plugin marketplace add shuvgenai/startup-board
/plugin install startup-board@startup-board
```

Or from a terminal:

```
claude plugin marketplace add shuvgenai/startup-board
claude plugin install startup-board@startup-board
```

Then restart Claude Code.

## How to use it

Just describe your idea. You don't need to mention the plugin — it starts on its own when you say things like:

- "Validate my startup idea: …"
- "Is this a good business?"
- "Write Mom Test interview questions for …"
- "Build me a Lean Canvas for …"
- "Do competitor research / market sizing for …"
- "How should I build the MVP?"
- "Help me launch this" / "Name this product" / "Build me a pitch deck"

## FAQ

**How do I validate a startup idea with AI?**
Install Startup Board and describe your idea. It interviews you, checks your evidence, assesses where you are on the path to product–market fit, and gives you an honest verdict with a clear next step.

**Does it create a Lean Canvas?**
Yes. It fills in a Lean Canvas from what you've told it and clearly marks anything that's still unknown instead of inventing numbers.

**Can it write customer interview questions?**
Yes — a question set that follows *The Mom Test*, tailored to your idea, so customers give you real answers instead of polite compliments.

**Does it do market research?**
Yes. It asks how deep you want to go first: a quick competitor scan, a positioning map with competitor deep-dives, or full market sizing with sources.

**Do I need to know how to code?**
No. It's designed for no-code and prompt-first founders. Build plans use no-code tools and include ready-made prompts.

**Will it just tell me my idea is great?**
No. It's built to give honest verdicts, including "talk to 10 more customers first" or "kill it."

**Is it free?**
Yes. Startup Board is open source under the MIT license. You need Claude Code to run it.

## License

[MIT](LICENSE) © 2026 shuvgenai
