---

name: validate-my-brain
description: Run the Copilot Second Brain validation session — walk today's pending proposals one at a time, apply the confirmed ones to the STATE files, fix wrong journal lines, collect what no agent can see, and stamp the brain as validated. Use ONLY when the user's request names the brain explicitly — "validate my brain", "brain validation", "review my brain's pending proposals", "let's validate the brain". A request that does not name the brain is never this skill, however close it sounds: other review, triage or inbox-processing skills may be installed alongside this one, and the word "brain" is the only thing that tells them apart.

---

Version: 1.7 (2026-08-25) — edit in the copilot-second-brain repo first, then copy here.

# Validation session — Copilot Second Brain

You are Copilot Cowork running an interactive session, triggered by the user ("Validate my
brain"). This is a conversation, not a batch job: it takes 2–3 minutes, and every write this
session makes is confirmed by the user first. This document is your complete instruction set
— you need nothing beyond it and the files it tells you to read. Follow the steps in order.
Do not skip steps, do not reorder them, do not improvise beyond what is written.

All paths below are relative to `Documents/Cowork/Second Brain/`. You may write **only**
inside this folder, plus one exception: moving journal files to
`Documents/Cowork/Second Brain Archive/` when Step 1 recommends archiving and the user
agrees. Never write anywhere else in OneDrive.

**How to run this conversation:** one thing at a time. Show what you're proposing to do,
including the evidence behind it, then stop and wait for the user's reply before doing
anything or moving on. Never batch multiple proposals into one message, never apply a
proposal the user has not confirmed, and never guess at a correction the user only implied —
ask.

**Two languages, one rule.** The conversation follows the user's language (reply in whatever
language they write). But **everything you write to a file is in English**, always — entry
titles, next steps, journal lines, stakeholder notes. When the user answers in another
language, translate their words before writing them; keep proper nouns, product names and
quoted titles verbatim. The brain's files are shared artifacts and stay English regardless
of who talks to them.

**File-name key mapping.** `<client>` in file names (`STATE-<client>.md`,
`JOURNAL-<client>-YYYY-MM.md`) is always the client's `- Key:` value from
`brain-config.md` — e.g. client "Acme Corp" with `- Key: acme` gives `STATE-acme.md`,
`JOURNAL-acme-2026-08.md`. `PENDING.md` client sections use the full client name (e.g.
`## Acme Corp`), never the key — map the section's name to its key via `brain-config.md`
before touching any file.

## Step 1 — Open PENDING.md and show coverage

Read `PENDING.md`.

- **If it does not exist, or contains only a cleared header** (`# Pending — cleared
  YYYY-MM-DD`, with no coverage table below it) — before saying anything, **check the
  journals**: for each client in `brain-config.md`, open its journal for the current month
  and look for agent-written lines (`[captured, unvalidated]` or `[enricher]`) below the last
  `· ritual ·` line — `[user]` and `[seeded]` lines never count, they are already validated
  and their STATE edit was applied when they were written. If there are none anywhere, tell the
  user so ("capture hasn't run yet" if the file is missing, "nothing pending since the last
  session" if it's a cleared header) and go straight to Step 4 (the user's turn) — **Steps
  2–3 are skipped entirely**. If there *are* lines below a marker, say that instead: capture
  has not run since the last session, so those facts have not been turned into proposals
  yet. Offer to walk them directly from the journal, in Step 2's format, and do so if the
  user agrees. An empty `PENDING.md` is a statement about capture, never a statement that
  the brain is up to date.
- **Otherwise**, show the user its header and coverage table first, exactly as written,
  zeros included — this is the state of capture, not something you summarize or filter.
  - If the header's date is older than 2 working days (Saturdays and Sundays don't count),
    say so before going further: capture may have stalled and the user should know before
    trusting what follows.
  - **Read out the `Awaiting validation` column too.** It counts, per client, the
    agent-written journal lines below that client's last `ritual` marker — the real backlog. `Items captured` can
    be `0` on every row while this column is not: that means the last run found nothing new,
    not that there is nothing to walk. If the table has only one column, capture is running
    a version older than 1.6 — say so, and derive the backlog yourself from the markers.
  - **Read out the source line too** (`Sources: Outlook covered to … · Teams … · Box …`),
    and if any source is behind the others, say it in words: that source has a window nobody
    has looked at yet, and the facts inside it are not missing from the brain — they were
    never seen. It closes by itself on the next run that finds the source available; what
    you owe the user here is knowing it is open. If the line is absent, capture is running
    a version older than 1.5 — say that too.
  - Count the files directly inside `Second Brain/`. If the count is **greater than 40**,
    recommend archiving: journal files (`JOURNAL-<key>-YYYY-MM.md`) whose `YYYY-MM` is more
    than 2 months before the current month are candidates. Tell the user how many files that
    would move and to where (`Second Brain Archive/`). Do this only if the user agrees — it
    is a write like any other in this session, so it needs confirmation. Moving a file means
    moving it, not deleting it; the archive is still on OneDrive, just outside the folder the
    query agent reads.
  - If `PENDING.md` has a coverage table but no client sections below it (nothing was
    proposed this run), say so and go straight to Step 4.

## Step 2 — Walk the proposals, one at a time

For each client section in `PENDING.md`, for each numbered proposal in it, in order:

1. Show the proposal exactly as written, plus its journal evidence — open the cited
   journal file(s) and show the actual line(s), not just the citation. The user should never
   have to take your word for what the journal says.
2. Ask the user: accept / amend / reject. Wait for the answer.
3. Apply the outcome:
   - **Accept** → write the change to `STATE-<client>.md` exactly as proposed. Exception: if
     applying it means creating a new Active work entry, or filling in a field the proposal
     itself didn't carry (see the sourcing rule below), show the complete entry block to the
     user and wait for their OK before writing it — a plain accept of a single-field edit the
     proposal already shows in full (e.g. "set to waiting-client") can be written directly,
     without this extra check.
   - **Amend** → the user gives you a correction; apply the proposal *with* that correction,
     not the original. Confirm briefly what you wrote before moving to the next proposal.
   - **Reject** → do nothing. No trace is needed anywhere: Step 5 appends a `ritual` line to
     the journal, which moves the validation marker below every line you walked this session,
     rejected ones included. A rejected proposal does not resurface because the evidence
     behind it is now above the marker — not because anyone recorded the rejection.

**Where a proposal lands in `STATE-<client>.md`** depends on what it is:

- A status change, a new waiting-on, or a due-date change → an **Active work** entry.
  `STATE-<client>.md` has fixed H2 sections, always in this order — write them verbatim:

  ```markdown
  # STATE — <Client>            Last validated: <YYYY-MM-DD — today's date>
  ## Summary
  ## Active work
  ## Recent decisions
  ## Risks & watchlist
  ## Declared blind spots
  ## Before writing to the client
  ```

  The **Active work** entry format, verbatim:

  ```markdown
  ### <Work item title>
  - Status: <status> · Waiting on: <who/what> · Due: <YYYY-MM-DD>
  - Next step: <one line>
  - Opened: <YYYY-MM-DD> · Stream: <stream name, or —> · Ref: (external task link, if any — not visible to agents)
  ```

  If the work item already exists in Active work, edit its entry in place (Active work is
  curated state, not a journal — unlike journal lines, it is meant to be rewritten as things
  change); leave its `Stream` as it stands unless the user says it has moved. If it doesn't
  exist yet, add a new entry — and source every field, never invent one:
  - `Opened` = the date of the **earliest journal evidence cited by the proposal**. Never
    today's date, never guessed — read it off the cited journal line(s).
  - `Next step` (and `Due`, if the proposal didn't carry one) is **asked**, not invented:
    "What's the next step on this?" If the user has none to give, write `Next step: —`.
  - `Status` and `Waiting on` come from the proposal (amended if the user corrected them).
  - `Stream` = the stream this item belongs to, taken from that client's `Streams:` line in
    `brain-config.md` (each stream is listed with its aliases in brackets — match on those
    too: a stream's items rarely all carry its name). Three cases, no others:
    **a match** → write it verbatim as the config spells it; **no match** → ask which stream
    it belongs to, and if the answer is a stream the config doesn't have yet, that is a
    `brain-config.md` change — read the edit back before writing it, exactly as for a new
    stakeholder; **the client has no `Streams:` line at all** → write `Stream: —` and do not
    ask. Never invent a stream name, and never file an item under a stream on a guess.
  - `Ref` is left as the literal placeholder text above unless the user gives you an actual
    link.

  **Closed items leave Active work.** If accepting or amending a proposal sets an item's
  status to `done`, `completed`, or `dropped`, do not write that status into Active work —
  **remove the item's entry from Active work entirely**, under the same accept/amend
  confirmation (no separate ask). Its history already lives in the journal; Active work only
  holds what's still open. If the closure is a notable decision (a scope drop, a final
  approval, a cancellation), offer to add one dated line about it under **Recent decisions**
  and add it only if the user agrees.

- A new decision → one dated line under **Recent decisions**.
- A new stakeholder → this is a `brain-config.md` change, not a STATE change. Read the exact
  edit back to the user before writing it (see "What NOT to do" below); on confirmation,
  add/update the stakeholder in that client's `## Client: <Name>` section.
- A **new stream** (an item that belongs to no stream on the client's `Streams:` line) → same
  thing: a `brain-config.md` change, read back before writing. Append it to that client's
  `Streams:` line as `<Name> (<aliases the user gives you>)`, keeping `; ` between streams.
  A stream the user says is a blind spot — work running at this client that they have no
  visibility on — belongs on the line too, marked as such: enumerating it is what keeps its
  silence from reading as "nothing is happening there".

**Worked example** (fictional client, do not copy the content — copy the shape):

> `PENDING.md` shows:
> ```
> ## Acme Corp
> 1. Set "Website relaunch — SOW approval" to waiting-client (waiting on: Acme procurement)
>    — evidence: JOURNAL-acme-2026-08.md lines of 2026-08-14 (email-in "SOW received", email-out "SOW sent")
> ```
> You show this proposal and the two cited journal lines, then ask: accept / amend / reject.
>
> User replies: **"amend: due date is Sep 5."**
>
> This creates a new Active work entry, so before writing anything you source the fields the
> proposal doesn't carry. `Opened` comes from the earliest cited evidence — both journal
> lines are dated 2026-08-14, so `Opened: 2026-08-14`. `Next step` isn't in the proposal
> either, so you ask: **"What's the next step on this?"**
>
> User replies: **"follow up if no reply by Aug 20."**
>
> You now show the complete entry back to the user before writing it, since this is a new
> entry with fields the proposal didn't supply:
> ```markdown
> ### Website relaunch — SOW approval
> - Status: waiting-client · Waiting on: Acme procurement · Due: 2026-09-05
> - Next step: follow up if no reply by Aug 20
> - Opened: 2026-08-14 · Stream: Website programme · Ref: (external task link, if any — not visible to agents)
> ```
>
> `Stream` needed no question: Acme's `Streams:` line reads
> `Website programme (relaunch, SOW, CMS migration); …`, and "relaunch" is one of its
> aliases — so the entry files under it as the config spells it.
> User confirms: **"yes."** You write it to `STATE-acme.md`, confirm it's written, then move
> to the next proposal.

## Step 3 — Wrong captured facts: delete vs. append

Separately from accepting or rejecting a *proposal*, the user may flag that a *journal line*
itself — something you showed as evidence, or something they recall — is wrong. This is not
the same thing: a proposal is a suggested STATE change; a journal line is a claimed fact.
Handle it with this test, verbatim:

**"did it actually happen?"**

- **No** (capture error — the agent misread or misattributed something) → **delete the
  journal line**, but only after the user explicitly confirms the deletion (e.g., you ask
  "delete this line?" and the user says yes). Never delete on inference or assumption.
- **Yes, but the situation has since changed** (a true fact that evolved — e.g., a date that
  really was Aug 20 and is now Sep 5) → **append a new journal line** recording the
  evolution. The old line stays; it was true when it was written.

**Never rewrite a journal line in place, in either case.** Delete-with-confirmation and
append are the only two operations you may perform on an existing journal file.

## Step 4 — The user's turn

Ask: **"anything the agents can't see?"** — meetings, hallway conversations, phone calls,
anything that happened in the spoken-word channel no capture sweep covers.

For each thing the user tells you:

1. Confirm which client it's about.
2. Append a journal line with `[user]` provenance to that client's
   `JOURNAL-<key>-YYYY-MM.md` for the current month — using the client's key from
   `brain-config.md`, created if it does not exist yet:
   `- YYYY-MM-DD · <type> · <one-line fact> — source: validation session · [user]`
   (`<type>` is the capture vocabulary minus `ritual`, which only the closing step
   writes: `email-in`, `email-out`, `meeting`, `decision`, `delivery`, `revision`,
   `blocker`, `unblock`, `note`.)
3. Apply it directly to the relevant `STATE-<client>.md` section — no proposal step, no
   accept/reject: what the user says in this session is validated by definition. Pick
   whichever of the six fixed sections fits (Summary, Active work, Recent decisions,
   Risks & watchlist, Declared blind spots, Before writing to the client).

Keep asking until the user says there's nothing more.

## Step 5 — Close

Do this last, in this order, once every proposal has been walked and the user's turn is
done:

1. For **every client in `brain-config.md`** — not only the ones touched this session — do,
   in this order:
   - Stamp `Last validated: YYYY-MM-DD` (today) at the top of its `STATE-<client>.md`,
     replacing the previous date. A client with nothing to walk was still reviewed, and this
     date is what the query agent reads as the brain's freshness.
   - Append to its journal for the current month, exact format:
     `- YYYY-MM-DD · ritual · validation session — N proposals: A accepted, M amended, R rejected · [user]`
     where N/A/M/R are that client's own counts from this session (N = A + M + R; a client
     touched only via Step 4, or not touched at all, gets `N proposals: 0 accepted, 0 amended, 0 rejected`).
     Create the journal file if it does not exist yet.

   **Write this line for every client, every session, even at zero.** It is not a log entry:
   it is the marker that tells capture where validated history ends and the backlog begins.
   A client you skip keeps its old marker, so every journal line since then gets re-proposed
   on every future run — and the facts you read and decided needed no proposal come back
   forever. Write it only once the walk is genuinely finished, though: the line claims those
   lines have been seen by a human, and stamping it over facts nobody walked buries them for
   good.
2. Clear `PENDING.md` last, after every count above has been read from it: overwrite it so
   the only content is a dated header, exact format (with today's date):
   ```
   # Pending — cleared <YYYY-MM-DD — today's date>
   ```
   No coverage table, no client sections — the next capture run rewrites the whole file.
   This header date is what tells the query agent how stale `PENDING.md` is when nothing
   else does, so never leave it undated.

## What NOT to do

- Never apply a proposal, an amendment, or a deletion that the user has not explicitly
  confirmed in this conversation — silence is not confirmation.
- Never edit `brain-config.md` silently. A new-stakeholder proposal, or any other config
  change, gets read back to the user in full before you write it.
- Never rewrite an existing journal line. Delete (with confirmation) or append — nothing
  else.
- Never touch the *content* of a `STATE-<client>.md` file for a client that had no proposals
  and no Step 4 entries this session — the `Last validated` stamp in Step 5 is the one
  exception, and it is the only line you may change there.
- Never append the `ritual` line before the walk is finished, and never skip it for a client
  because nothing happened there. It is the validation marker, not a record of activity.
- Never move a journal file to the Archive without the user's agreement, and never delete
  one — moving out of the grounding path is enough, it stays recoverable on OneDrive.
- Never leave `PENDING.md` without a dated header after clearing it.
