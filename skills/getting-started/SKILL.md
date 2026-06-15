---
name: getting-started
description: >-
  A friendly, no-jargon tour of Cortex Code Desktop for first-time, non-technical
  partners. Explains what the assistant can do, how to ask for things in plain
  language, and a few safe things to try right now. Triggers: what can I do here,
  getting started, give me a tour, how do I use this, help me get oriented, I'm new.
---

# Getting Started — a friendly tour

The user is new and possibly non-technical. Give a short, encouraging orientation. No
jargon. Use plain language and concrete examples. Keep it scannable.

When invoked, do the following:

## 1. One-line reassurance

Start with something like:
> You don't need to know how to code. You talk to me in plain English, I do the work,
> and I always ask before changing anything.

## 2. Explain the three things they can do

Present these as three simple cards/bullets with a real example each:

- **Ask questions about their data** — e.g. *"How many customers signed up last month?"*
  (If a Snowflake connection is set up, offer to run a simple, safe example query.)
- **Build something** — e.g. *"Make me a simple dashboard of monthly sales."*
- **Get help / learn** — e.g. *"Explain what a Cortex Agent is in simple terms."*

## 3. How approvals keep them safe

Explain in one or two sentences that nothing runs without their approval: they'll see an
**Approve / Reject** prompt, and they can always say no. Mention they can switch to **Plan
Mode** if they just want to see the plan first without anything happening.

## 4. Offer one safe thing to do right now

Ask the user which they'd like to try (use the question tool):
- A quick data question (only if a connection exists — check with `snow connection list`).
- A tiny "hello" build (e.g. a one-page summary or a simple chart from sample numbers).
- Just keep exploring on their own.

Then do whatever they pick, narrating each step simply.

## 5. Where to go next

Close by pointing them at:
- Re-run **/setup-my-perspective** anytime to adjust their setup.
- **Agent Settings → Skills** to see everything the assistant knows how to do.

Keep the whole thing warm and under ~250 words of assistant output.
