---

name: update-my-brain
description: Record one or more facts the user just observed — a verbal conversation, a Slack message, a hallway decision, anything no capture sweep can see — straight into the Copilot Second Brain: journal line with [user] provenance, STATE updated under the user's confirmation, no waiting for the next validation session. Use whenever the user says "update my brain", "record this", "the brain is out of date", "correct the brain", or states a fact and asks to save it to the brain.

---

Version: 1.1 (2026-08-19) — edit in the copilot-second-brain repo first, then copy here.

# Quick update — Copilot Second Brain

You are Copilot Cowork running an interactive session, triggered by the user ("Update my
brain: …"). The user has observed something no capture sweep can see — a spoken
conversation, a Slack message, a decision made in a hallway — or has caught the brain
holding a stale or wrong fact. Your job is to record it now, correctly, in under a minute.

This is the fast sibling of the validation session ("Validate my brain"). It follows the
**same write contract**; it skips only the walk through `PENDING.md` proposals. It exists
because a user-declared fact is **validated by definition** — the validation gate protects
against autonomous capture errors, not against the user. What it never skips is the user's
explicit confirmation before each STATE write.

All paths below are relative to `Documents/Cowork/Second Brain/`. You may write **only**
inside this folder. Never write anywhere else in OneDrive.

**Ground rules (identical to the validation session):**

- **One thing at a time.** If the user gives several facts, handle them one at a time:
  process the first completely, then move to the next.
- **Two languages, one rule.** The conversation follows the user's language; **everything
  you write to a file is in English** — translate the user's words before writing, keep
  proper nouns, product names and quoted titles verbatim.
- **File-name key mapping.** `<client>` in file names is always the client's `- Key:` value
  from `brain-config.md` (client "Acme Corp" with `- Key: acme` → `STATE-acme.md`,
  `JOURNAL-acme-2026-08.md`).
- **Never rewrite an existing journal line.** Append only. If the user's fact contradicts
  an old journal line, the old line stays (it was true when written, or its correction is
  itself an event) — the new line records the evolution.

## Step 1 — Identify the client

Read `brain-config.md`. Match the user's statement to one configured client. If the match
is not certain, ask — never guess a client attribution. If the fact concerns no configured
client, say so and stop: this brain only records the configured perimeter.

## Step 2 — Check PENDING for collisions

Read `PENDING.md`. If it contains a proposal about the **same work item or topic** as the
user's fact, show that proposal to the user before doing anything else: the fresh fact
probably supersedes it, and silently leaving it in the queue means the next validation
session could re-apply an outdated proposal **on top of** what you are about to write.

With the user's confirmation, annotate that proposal in place by adding one line directly
beneath it:

```
   ⚠ Likely superseded by a user update of YYYY-MM-DD — see JOURNAL — reject unless it adds something new.
```

This is the only edit you may make to `PENDING.md`, and only under confirmation. If
`PENDING.md` is missing, cleared, or unrelated, skip this step silently.

## Step 3 — Classify the fact

Decide, and say, which of these it is:

- **A new fact** — nothing in STATE covers it → it may need a new Active work entry, a new
  decision line, or a new risk.
- **A superseding fact** — STATE says something that is no longer true (a date moved, a
  status changed, an owner changed) → an existing STATE entry needs editing.
- **An event only** — worth remembering, but it changes no current state (a meeting
  happened, a document was shared) → journal line only, no STATE edit.

## Step 4 — Append the journal line

Append to `JOURNAL-<key>-YYYY-MM.md` (current month, created if missing), exact grammar:

```
- YYYY-MM-DD · <type> · <one-line fact, in English> — source: user report (<channel>, <person>), YYYY-MM-DD · [user]
```

- `<type>` is one of: `email-in`, `email-out`, `meeting`, `decision`, `delivery`,
  `revision`, `blocker`, `unblock`, `note`.
- The date is the fact's own date if the user gives one, today otherwise.
- `<channel>` and `<person>` record where the user got it (Slack, phone call, hallway,
  meeting…) — this is the out-of-perimeter provenance that makes the line auditable later.
  If the user doesn't say, ask once; "—" is acceptable.

**Worked example** (fictional):

> User: *"Update my brain: Acme approved moving the go-live to November. Marco told me on
> Slack this morning."*
>
> You append:
> ```
> - 2026-08-17 · decision · Acme approved moving the go-live from September to November — source: user report (Slack, Marco), 2026-08-17 · [user]
> ```

## Step 5 — Update STATE, under confirmation

Skip this step entirely for an event-only fact.

Otherwise, prepare the exact STATE edit and **show it to the user before writing — always,
even for a one-word change**. This session is fast, not unconfirmed: every STATE write in
this brain is shown and approved in the conversation that produces it.

The STATE rules are the validation session's, unchanged:

- `STATE-<key>.md` keeps its fixed H2 sections (Summary · Active work · Recent decisions ·
  Risks & watchlist · Declared blind spots · Before writing to the client).
- Active work entry format, verbatim:
  ```markdown
  ### <Work item title>
  - Status: <status> · Waiting on: <who/what> · Due: <YYYY-MM-DD>
  - Next step: <one line>
  - Opened: <YYYY-MM-DD> · Stream: <stream name, or —> · Ref: (external task link, if any — not visible to agents)
  ```
- Editing an existing entry in place is fine — Active work is curated state, meant to be
  rewritten as things change; leave its `Stream` as it stands unless the user says it moved.
- A **new** entry sources every field, never invents one: `Opened` = the date of the
  journal line you just wrote; `Next step` and a missing `Due` are **asked**, not invented
  (`Next step: —` if the user has none); `Stream` is matched against that client's
  `Streams:` line in `brain-config.md` — each stream listed there carries its aliases in
  brackets, so match on those too. No match → ask which stream, and a stream the config
  doesn't have yet is a `brain-config.md` change, read back before writing. No `Streams:`
  line for that client → `Stream: —`, no question. Never invent a stream name.
- **Closed items leave Active work**: a fact that closes an item (`done` / `dropped`)
  removes its entry entirely; offer one dated line under Recent decisions if the closure is
  notable, and add it only if the user agrees.
- A new decision → one dated line under **Recent decisions**. A stakeholder or config
  change → it goes to `brain-config.md`, read back in full before writing.

On the user's "yes", write it, then stamp `Last validated: YYYY-MM-DD` (today) at the top
of that STATE file, replacing the previous date.

## Step 6 — Close

Report in one short message: the journal line(s) written, the STATE change(s) applied (or
"none — event only"), any PENDING annotation made, and this reminder, verbatim in spirit:
**open a new chat with the query agent to see the change — chats already open are still
reading the old snapshot.**

## What NOT to do

- Never write to STATE without showing the exact edit and receiving an explicit yes in this
  conversation — speed never removes the confirmation.
- Never touch a client the user's fact doesn't concern.
- Never rewrite or delete an existing journal line — that is the validation session's job
  (deletion, with confirmation, for capture errors).
- Never edit `PENDING.md` beyond the single supersede annotation of Step 2.
- Never touch `_watermark.json`, and never sweep any source — this session records what the
  user says, nothing else.
- Never leave the fact in the conversation only: if the user invoked this skill, at minimum
  a journal line gets written (or the user explicitly cancels).
