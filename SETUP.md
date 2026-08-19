# Setup — Copilot Second Brain

Self-service, about 10 minutes, four steps. No installation, no admin access beyond what
you already have in Microsoft 365. Follow the steps in order — each one tells you exactly
what to open, what to type and what to paste.

## Prerequisites

- Microsoft 365 Copilot with **Cowork**.
- Access to **Copilot agent builder** (to create a declarative agent).
- OneDrive syncing normally on your machine.

That's everything. There is no other tooling involved.

## Step 1 — Run the onboarding interview

1. Open Cowork.
2. Open `prompts/onboarding-interview.md` in this repository and paste its **full content**
   into Cowork as your message.
3. Answer the questions as they come — Cowork asks **one at a time**, waits for your reply,
   then asks the next one. You'll be asked for your name, your agency, timezone, preferred
   daily capture time, and then, per client: name, counterpart email domain(s), any third
   parties, any internal keywords, the Teams channels the work lives in, your agency's team,
   and the client's stakeholders. At the end you'll be asked where things stand today with
   each client — that becomes their starting state.

When it finishes, Cowork will have created `Documents/Cowork/Second Brain/` on your OneDrive,
with `brain-config.md`, one `STATE-<client>.md` file per client, and a `README.md` explaining
the folder. Nothing is captured automatically yet — that's the next step.

## Step 2 — Create the scheduled capture task

1. In Cowork, create a new **scheduled task**.
2. Name it `CSB capture` (any name works — this one makes it easy to find in the task list).
3. Set it to run **weekdays**, at the time you chose during the interview.
4. Open `prompts/capture-task.md` in this repository and paste its **full content** — from
   the `Version:` line at the top to the last line — as the task's prompt.

From its next scheduled run onward, this task reads your Outlook, Teams and Box, and appends
one-line facts to each client's journal — nothing is summarized or rewritten, only added.

## Step 3 — Create the query agent

1. In Microsoft Copilot, open **Agent builder** and start a new agent.
2. **Name**: `Copilot Second Brain` (add ` — <your name>` if your tenant may end up with
   more than one).
3. **Description** (it's a required field) — paste this:

   > Answers questions about your clients from your Copilot Second Brain — your validated,
   > per-client working memory on OneDrive. Cites file and date for every fact, declares
   > what has not been human-validated yet, and retrieves the original email or meeting
   > when the one-line record is not enough.

4. **Knowledge source**: add exactly one folder — your `Documents/Cowork/Second Brain/`
   folder on OneDrive. Do not add `Second Brain Archive/` (archived journals are *meant*
   to fall out of the agent's view) or any other folder or file.
5. **Instructions**: open `prompts/agent-instructions.md` in this repository. It starts with
   a short preamble, then a block wrapped in two markers, `INSTRUCTIONS:BEGIN` and
   `INSTRUCTIONS:END`. **Copy only what's between those two markers** — not the markers
   themselves, not the preamble above them — and paste that into the agent's instructions
   field.

   > This is the one step where copying too much breaks things: the preamble is written for
   > *you*, setting the agent up. The pasted block is written for *the agent*, and is what it
   > follows on every answer. Paste the wrong part and the agent either won't fit under the
   > instructions field's character limit, or won't behave as designed.

6. Any other field agent builder offers (starter prompts, capabilities, extra tools): leave
   the defaults. The agent needs nothing beyond the knowledge source and the instructions.
7. Save, then test it with a question you already know the answer to (e.g. "what's the status
   of <one of your clients>?") before relying on it day to day.

## Step 4 — Install the two Cowork skills

Two interactive tools ship as Cowork **skills** you trigger by name instead of pasting a
text each time:

- **`validate-my-brain`** — the periodic validation session (every day or two): walk what
  capture proposed, accept/amend/reject, add what no agent saw.
- **`update-my-brain`** — the quick update (whenever, 30 seconds): record one fact you just
  observed — a call, a Slack message, a hallway decision — without waiting for a session.

Install both the same way:

1. In this repository, find the folders `skills/validate-my-brain/` and
   `skills/update-my-brain/` (each contains one file, `SKILL.md`).
2. Copy **both whole folders** into `Documents/Cowork/skills/` on your OneDrive, so you end
   up with `Documents/Cowork/skills/validate-my-brain/SKILL.md` and
   `Documents/Cowork/skills/update-my-brain/SKILL.md`. Create the `skills/` folder if it
   doesn't exist yet.
3. In a new Cowork chat, say **"Validate my brain"** — Cowork picks the skill up from there.
   The first time, it will most likely tell you capture hasn't run yet: that's correct, not
   an error. Try the other one with **"Update my brain: …"** followed by any real fact you
   want recorded.

## Your first week

Facts only start flowing once the scheduled task has run, and STATE only reflects them once
you've validated. For the first few days:

- Run a **validation session** every day or two: say **"Validate my brain"** in a new Cowork
  chat (the skill you installed in Step 4). Each session takes 2–3 minutes: you walk through
  what capture proposed, accept, amend or reject it, and add anything the agents couldn't see.
- When you learn something outside the brain's reach — a call, a Slack message, a hallway
  decision — say **"Update my brain: <the fact>"** the moment you have it. Thirty seconds,
  and the brain stops being stale on that point without waiting for the next session.
- Expect **thin coverage on day one** — the first capture run has no watermark to start from,
  so it looks back a fixed **24 hours**, not your whole history with the client. If setup
  happens more than a day before the first scheduled run, the gap in between isn't captured —
  that's fine, since the onboarding interview already asked "where do things stand today" and
  put that answer straight into STATE, validated by definition. Coverage fills in from there
  as capture keeps running.
- Before that first run, if you ask the agent something, it will tell you capture hasn't run
  yet rather than warning about staleness. Once captures begin, it flags staleness only if
  they later stop — that's the staleness rule working as intended, not a bug.

## Adding a Teams channel later

When a client's work moves into a Teams channel the brain wasn't watching: add the channel
to that client's `Teams channels` line in `brain-config.md` (the scheduled capture picks it
up from the next run), then run `prompts/channel-backfill.md` **once** in a normal Cowork
chat to pull in the channel's history — the scheduled capture only looks forward. External
and shared channels work too, as long as you can open them in your own Teams.

## Troubleshooting

| Symptom | Check |
|---|---|
| No capture output (journals not growing) | Confirm the scheduled task actually ran in Cowork's task history; check `_watermark.json` in `Second Brain/` — if it hasn't moved, the task isn't completing. |
| Agent doesn't see facts you know are there | Open a **new chat** with the agent — an open chat is a snapshot of the brain at the time it started, not a live view. Also confirm OneDrive has finished syncing the updated files. |
| Agent cites nothing / says the brain is empty | Check the agent's knowledge source in agent builder: it must point at `Documents/Cowork/Second Brain/` on your own OneDrive, not a different or empty folder. |

## Next

- How the whole system is designed and why: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)
- What this is, for anyone you're introducing it to: this repository's [`README.md`](README.md)
