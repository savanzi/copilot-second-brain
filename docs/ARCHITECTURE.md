# Copilot Second Brain — Architecture

**Status: approved design, 2026-08-13.** This document describes the system as designed, before implementation. It is the reference the prompts, the setup guide and the deck are built from. When the architecture changes, this file changes in the same gesture.

---

## 1. What it is

The **Copilot Second Brain (CSB)** is a per-user, per-client working memory built entirely on Microsoft 365 Copilot: **Cowork** maintains it, a **declarative Copilot agent** answers questions from it, and it lives as plain markdown files on the user's **OneDrive**. It requires nothing outside M365: no local tooling, no Claude Code, no third-party services.

Every user runs their own instance, on their own clients. Nothing about any specific client, person or company is hardcoded anywhere: all per-user variables live in one configuration file (`brain-config.md`), created through a guided onboarding interview. The machinery — three operating prompts, two Cowork skills, a one-off backfill tool and a setup guide — is generic and distributed as a self-service kit.


**Division of labour with external task managers.** The CSB is memory and context, not a task manager. Operational task tracking stays where teams already work (ClickUp, monday.com, …); the CSB records the *state* of work (status, waiting-on, next steps) as validated distillate.

## 2. Design principles

Each of these was paid for during a month of running an earlier, hand-built version of this idea:

1. **Distill, never paste.** The brain holds one-line facts with enough metadata to re-find the original. Full emails and transcripts stay in M365, where the reader can always retrieve them live (§6, rule 3).
2. **Autonomous writing is additive only.** The scheduled capture task appends; it never rewrites existing lines. Rewriting is reserved for the validation session, where a human confirms every change.
3. **Humans validate judgment, agents capture facts.** Raw facts flow in autonomously, tagged `[captured, unvalidated]`. Judgment — status changes, decisions, summaries — reaches the curated layer only through the validation session. (Two-level hybrid model, §7.)
4. **Absence must not look like success.** Coverage is declared with zeros, per client, dated. Staleness is detected by the reader, not trusted to the writer. Blind spots are written down, never deduced from silence.
5. **Wrong presence is worse than absence.** Client attribution uses the counterpart's **email domain**, never lists of people (colleagues appear on several clients; a person's name classifies nothing).
6. **Machine and data never share a path.** The repository holds only prompts, docs and the deck. Configuration, state and journals live on the user's OneDrive and have no route into the repo. This is enforced by construction, not by exclusion lists.
7. **Everything is English.** File names, structure, prompts, docs, deck, brain content. This binds every surface that writes: capture translates the Italian email it just read, the two sessions translate what the user says to them, and the conversation's own language changes nothing about the file. Only the query agent's *answers* follow the user's language — what it reads stays English. Proper nouns, product names and quoted subjects are the one exception, kept verbatim: a translated subject line no longer finds the original.

## 3. Layout

Everything lives in Cowork's working directory — the only OneDrive location Cowork can write to:

```
Documents/Cowork/
├── Second Brain/                      ← grounded by the query agent
│   ├── README.md                       ← what this is, usage rules, warnings
│   ├── brain-config.md                 ← per-user configuration (§4)
│   ├── STATE-<client>.md               ← curated state, 1/client — user-confirmed writes only
│   ├── JOURNAL-<client>-YYYY-MM.md     ← append-only journal, 1/client/month — autonomous
│   ├── PENDING.md                      ← proposed STATE updates awaiting confirmation
│   └── _watermark.json                 ← coverage watermark, one `covered_until` per source
└── Second Brain Archive/              ← journals older than 2 months, OUTSIDE grounding
```

File count with 4 clients: ~14 active files. The agent builder's **50-files-per-source cap** is the binding platform constraint; the structure stays under it by design (files grow with clients and months, not with tasks), and old journals move to the Archive folder — still on OneDrive, never deleted, just out of the grounding.

**Backup and recovery: OneDrive native version history.** No snapshot machinery. A botched write to any file is recovered from the file's version history.

### File classes

| File | Class | Who writes it | Discipline |
|---|---|---|---|
| `brain-config.md` | Configuration | Onboarding interview; user edits | Free |
| `STATE-<client>.md` | Curated state | Validation session · quick update (both user-confirmed) | Every line human-confirmed |
| `JOURNAL-*.md` | Raw facts | Capture task (append) · validation (delete errors, under confirmation) | Append-only for autonomous writes |
| `PENDING.md` | Ephemeral derived | Capture task (rewrite daily) · validation (clear) | Disposable: losing it loses a proposal, never a fact |
| `README.md` | Rules | Onboarding; kit updates | Stable |

`PENDING.md` is the one deliberate exception to principle 2 (autonomous rewrite): accepted because it is ephemeral and derived — the facts it points to live in the journals, and validation can always be rebuilt from them.

## 4. Configuration — `brain-config.md`

The heart of scalability: three of the four prompts read from it, and no prompt ever contains a client name. Fixed sections, one per client:

```markdown
# Brain Configuration
Owner: <name> · Timezone: <tz> · Daily capture: 07:00 weekdays
Agency: <agency>         ← the user's own organisation — a config value, never hardcoded
Task system: none        ← reserved; see §8

## Client: <Name>
- Key: <slug>                          ← used in file names
- Counterpart domains: <domain.com>    ← inclusion criterion (a domain, or one full address; never a name)
- Keywords: <…>                        ← classify agency-internal threads about this client
- Streams: <name (aliases); …>         ← the client's workstreams, with the other names each goes by
- Third parties: <vendor, tool, …>     ← senders that don't carry the client domain
- Teams channels: <names, or —>        ← opened on every capture run (domain never fires there)
- Agency team: <names>                 ← classify + sweep 1:1/group chats, never filter
- Stakeholders: <name — role — notes>
```

Adding a client = adding a section (via a re-run of the onboarding interview, or by hand). Capture, validation and the query agent pick it up on their next run; nothing else is touched.

## 5. The five surfaces

Two roles, cleanly split: **Cowork is the hands** (capture, validation, quick update — it can write files), **the declarative agent is the reader** (query + drill-down — it cannot write). Three surfaces are prompts in `prompts/`; the two interactive sessions ship as Cowork **skills** in `skills/` (invoked by name, so a paste-once prompt would be the wrong vehicle). All five are versioned here and installed there ("edit here first, then paste/copy there").

The write invariant, stated precisely: **STATE is written only in user-confirmed interactive Cowork sessions** — the validation session (§5.2) and the quick update (§5.5), which share one write contract — and **autonomous capture never touches STATE**. The guarantee "every STATE line passed through a human" holds because the human is present and confirming in both sessions, not because there is exactly one session.

### 5.1 Capture — scheduled Cowork task (`prompts/capture-task.md`)

Runs weekdays at the configured time. Steps:

1. Read `brain-config.md` and `_watermark.json`; **each source has its own window**, starting at its own `covered_until` − 15 min (consecutive runs never leave gaps, and a source that was down on an earlier run carries a longer window until it catches up).
2. Sweep **Outlook, Teams and Box** in the window. Inclusion: counterpart **domain** in from/to/cc; plus third parties and keywords for vendors and agency-internal threads. **Teams gets its own enumerated pass** — the domain criterion barely fires there: every channel listed in the client's `Teams channels` is opened on every run (a mapped-but-unopened source is a fabricated silence — a lesson paid for on chat DMs), and 1:1/group chats with the client's `Agency team` members are swept for that client's facts.
3. **Append** one line per relevant fact to the client's journal:
   `- 2026-08-13 · email-in · Acme asks to move the SOW review call to Aug 20 — source: mail "SOW review reschedule" from j.smith@acme.com, 2026-08-13 · [captured, unvalidated]`
   Event types: `email-in`, `email-out`, `meeting`, `decision`, `delivery`, `revision`, `blocker`, `unblock`, `note`. Relevance threshold: *"will I need to know this happened, two weeks from now?"* Source metadata must be enough to re-find the original (sender, subject/title, date). Never touch existing lines.
4. Rewrite `PENDING.md`: proposed STATE changes (status change, new decision, new waiting-on, new stakeholder), each referencing the journal lines that justify it. **Derived from every agent-written journal line below that client's last `ritual` marker — never from this run's own facts alone.** That is what lets a proposal survive the daily rebuild: a run that finds nothing new still re-proposes the whole unwalked backlog, and only a human walking it empties the section (§5.2).
5. Declare coverage at the top of `PENDING.md`: dated table *client × items captured × awaiting validation*, **clients enumerated from the config, zeros included**, plus a line naming how far each source is covered and what an unavailable source left uncovered. The second column counts the agent-written journal lines below each client's last `ritual` marker: capture found nothing new is a different statement from there is nothing to walk. Above the table, when the most recent `ritual` line anywhere is 3+ days old and at least one client is carrying unvalidated facts, a one-line **validation-staleness banner** says how long nobody has walked the brain and how many clients that leaves stale — the table says how much is waiting, the banner says how long it has been waiting.
6. Update `_watermark.json` — **advancing only the sources actually read**.

### 5.2 Validation — interactive Cowork session (`skills/validate-my-brain/SKILL.md`, a Cowork skill)

User-triggered ("Validate my brain"), 2–3 minutes, recommended daily or every other day. **Skipping it breaks nothing**: capture keeps accumulating, and because proposals are derived from the `ritual` markers rather than from the current run, they survive every rebuild of `PENDING.md` until a human walks them. The reader just reports more facts as unvalidated in the meantime.

1. Show the day's coverage table (zeros included). Warn if the brain folder is approaching the 50-file cap (>40 files → archive old journals).
2. Walk `PENDING.md` **one proposal at a time**: accept / amend / reject. Accepted or amended → applied to `STATE-<client>.md`. Rejected → dies there (the closing step's `ritual` line moves the marker past the evidence, so it does not resurface).
3. Wrong captured facts: **capture error → delete the journal line** (under user confirmation — the append-only rule constrains the unsupervised writer, not the supervised session); **true fact that evolved → append a new line** recording the evolution. The test: *"did it actually happen?"* If not, it does not deserve to exist in the brain.
4. The user's turn: *"anything the agents can't see?"* — the spoken-word channel no agent covers. Entries go to the journal with `[user]` provenance and to STATE directly (validated by definition).
5. Close: stamp `Last validated: <date>` on **every** client's STATE and append a `ritual` line to **every** client's journal — the line is the validation marker capture reads, so a client skipped here has its whole history re-proposed forever — then clear `PENDING.md` last.

### 5.3 Query — declarative Copilot agent (`prompts/agent-instructions.md`)

One agent per user, created in agent builder, knowledge source = the user's `Second Brain/` folder. Behaviour rules:

1. **Answer from the brain; cite the file** (`STATE-acme.md`, `JOURNAL-acme-2026-08.md`) and the fact's date.
2. **Declare the trust level**: a fact in STATE is validated; a journal-only fact is unvalidated **if it sits below its client's last `ritual` marker**, and the agent says so. The tag records the writer, not the review.
3. **Drill down on cited sources.** When a relevant brain line cites an email or meeting and the distilled line is not enough (exact wording, tone, detail, or the question is about that conversation), retrieve the full original from Outlook/Teams — proactively, not only on request — and separate the layers in the answer: *"the brain records X (Aug 13); from the full email: …"*. Resolving a source the brain cites is always allowed (attribution was already done by the brain); **free-searching the mailbox is not** — see rule 4 for its only use.
4. **Silences are falsified, never inherited.** Before asserting an absence ("the client hasn't replied"), search M365 live in the window between the last capture and now; findings are declared *unrecorded* — candidates for the next capture, not corrections.
5. **An open chat is an undeclared snapshot.** The agent states the as-of date of what it read; after a brain update, start a new chat (also written in the brain's `README.md`).
6. **Reply in the user's language.** Brain content is English; answers follow the question's language.
7. **Never write.** Correction requests are redirected to the validation session in Cowork.
8. **Declare staleness, on both clocks.** If the latest coverage table is older than 2 working days, say so in every answer ("capture may have stopped"). Separately, repeat `PENDING.md`'s validation-staleness banner if present, and give a client's `Last validated` date whenever it is more than 5 days old — capture running is not the same as anyone having checked what it captured.

### 5.4 Onboarding — guided interview (`prompts/onboarding-interview.md`)

A Cowork prompt, "Set up my Second Brain":

1. Interview, **one question at a time**: owner and agency, clients, counterpart domains, third parties, agency team, key stakeholders, capture time.
2. Cowork writes `brain-config.md`, creates the folder structure and `README.md`, and drafts the **initial STATE files from the user's own account**: *"where do things stand with each client today?"* — the brain starts with a validated-by-definition snapshot, not empty. It closes by **naming that client's streams**, grouping the items it has just drafted rather than asking cold (§6).
3. Two guided manual gestures close the setup (~10 minutes total, self-service): create the **scheduled task** in Cowork (paste the capture prompt), create the **agent** in agent builder (paste the instructions, point the knowledge source at the folder).

### 5.5 Quick update — interactive Cowork skill (`skills/update-my-brain/SKILL.md`)

User-triggered ("Update my brain: …"), ~30 seconds. It records a fact the user just
observed in a channel no capture sweep can see — a spoken conversation, a Slack message, a
hallway decision — or corrects a stale fact the user caught while talking to the query
agent. It is the **spoken-word channel of §4 principle, made invocable on its own**: the
validation session's Step 4 ("anything the agents can't see?") factored out, so a single
fresh fact doesn't have to wait for the next session or queue behind capture's proposals.

Flow: identify the client (ask if ambiguous) → check `PENDING.md` for proposals the fact
supersedes (annotate them, under confirmation, so the next validation doesn't re-apply an
outdated proposal on top of the fresh fact) → append the journal line with `[user]`
provenance and the out-of-perimeter source spelled out (`source: user report (Slack,
Marco), …`) → if the fact changes current state, show the exact STATE edit and write it
only on the user's yes → stamp `Last validated` → remind the user to open a new chat with
the query agent (snapshot rule).

It shares the validation session's write contract verbatim (STATE sections, Active work
format, field sourcing, closed-items rule, English-in-files, key mapping). What it never
skips is the show-back: **speed removes the queue, never the confirmation.** A user-declared
fact bypasses `PENDING.md` by design, not by exception — the validation gate exists to
protect against autonomous capture errors, and the user is not an autonomous capture.

## 6. STATE file — the curated layer

```markdown
# STATE — <Client>            Last validated: 2026-08-13
## Summary                    ← 5–10 lines: where things stand with this client
## Active work                ← the CSB's task registry (see below)
## Recent decisions           ← dated, one line each (older ones live in the journal)
## Risks & watchlist
## Declared blind spots       ← what this brain does NOT see (e.g. Slack, external task systems)
## Before writing to the client  ← tone, sensitivities, things not to say
```

The property that holds the design together: **STATE is by construction the only file where every line passed through a human.** That is what makes reader rule 2 meaningful.

**Active work is the task registry.** Entry format:

```markdown
### Website relaunch — SOW approval
- Status: waiting-client · Waiting on: Acme procurement · Due: 2026-08-31
- Next step: follow up if no reply by Aug 20
- Opened: 2026-07-12 · Stream: Website programme · Ref: (external task link, if any — not visible to agents)
```

Lifecycle: capture *proposes* changes it infers from mail and meetings; validation *applies* them; closed items leave Active work and remain as journal history.

**Streams are one optional field, not a structure.** A client's work splits into workstreams (a programme, a product, a retainer); each is declared once on the client's `Streams:` line in `brain-config.md` **with its aliases** — the other names the same work goes by — and each Active work entry names its own in `Stream:`. The aliases are the working part: a stream's items rarely all carry the stream's name (a "GEO Readiness" stream holds items titled after the tool it runs on), and a reader searching one name would otherwise retrieve half of them. Onboarding fills the list at the end of §5.4 Step 4, by grouping the items it just drafted — never as a cold question — and validation stamps the field when it creates an entry, adding a new stream to the config under the same read-back rule as a new stakeholder. A client with no `Streams:` line loses nothing: the field is `—` and nobody asks.

Who touches it: **validation and quick update write it, the reader resolves it, capture ignores it entirely** (it keeps including by domain and `Keywords`) — so adding streams costs no change to the scheduled task, and no re-paste of it.

Considered and rejected: **a file per stream** (files grow with clients and months, never with work — that is what keeps the 50-file cap non-binding); **nesting Active work under stream headings** (changes the write contract of two skills for no retrieval gain — a declarative agent matches text, it does not navigate heading hierarchy); **a stream tag on journal lines** (it would put a judgment call inside autonomous capture, the one writer no human validates, and a mis-filed line is a wrong presence, not an absence).

## 7. Validation model — two-level hybrid

Chosen explicitly over the alternatives:

- *Fully assisted* (every write human-approved): highest quality, but 5+ min/day per user — does not scale to less motivated colleagues.
- *Fully autonomous*: zero friction, but that month measured real, silent errors (4 of 6 factual errors were caught by the human, not by agents).
- **Hybrid (chosen): facts flow autonomously with provenance; judgment is confirmed by a human in a short session.** The gate sits exactly where errors are expensive.

Provenance tags: `[captured, unvalidated]` (autonomous capture) · `[user]` (spoken-word channel) · `[seeded]` (one-time migration) · `[enricher]` (optional, §9). STATE lines carry no tag: the file's write discipline *is* the tag.

**Provenance is not validation state.** A tag says who wrote a line and never changes, because journal lines are never rewritten — so `[captured, unvalidated]` on a fact validated months ago is not a bug, it is the tag doing its only job. Validation state is **positional**: the `ritual` line each session appends to every client's journal is the watermark. The backlog is the intersection of the two — **agent-written lines below the last marker**: `[user]` and `[seeded]` lines are validated wherever they sit, because the person who wrote them applied the STATE edit in the same gesture (§5.2 step 4, §5.5). Both capture (§5.1) and the reader (§5.3) derive it that way. Reading the tag alone as a to-do list overcounts the backlog by the entire validated history; reading the marker alone re-proposes the user's own words back to them.

## 8. External task systems — suspended

Cowork has **no access to Microsoft Planner** (verified 2026-08-13). It does have a **monday.com** connector, not enabled in the tenant this design was validated against — so task-system integration is **suspended, not dropped**. The design keeps the slot open at zero cost:

- `brain-config.md` carries `Task system: none` (reserved field).
- When monday.com (or another Cowork-reachable system) becomes available: the capture task adds it as a sweep source (task completed / created / due-date moved → journal + PENDING proposals), and the reader gains drill-down on task references. Nothing else changes.
- Until then, the external task system is a **declared blind spot** in every STATE, and Active work entries carry plain reference links that agents cannot resolve.

## 9. Optional enricher — Claude Code (nice to have)

If the user has Claude Code, it can feed the CSB from sources Cowork cannot see (Slack, a time-and-project system, generic MCP): it **appends journal lines** with `[enricher]` provenance and **adds proposals to `PENDING.md`**, through the same two doors as everything else — never a required component, never a writer of STATE. OneDrive is reached as a locally synced folder only — a deliberate constraint, so the enricher needs no Microsoft API or MCP access.

## 10. Seeding vs onboarding

Two entry paths, one arrival format:

- **Seeding, if you already keep client state somewhere structured:** a one-time pass can generate `brain-config.md` from that source and distil STATE files (translated to English) and back-dated journal lines carrying `[seeded]` provenance, all reviewed before go-live. Such a migration is specific to the system it reads, so **no seeding tool ships with this kit**; the source system is left untouched.
- **Everyone else — the supported path:** the onboarding interview (§5.4). No migration, no history — the brain starts from the interview snapshot and grows from capture.

## 11. How this system fails

The central lesson of that month: **an absence that looks like success.** Known failure modes and their countermeasures, all of which are rules in the prompts, not good intentions:

1. **Capture silently stops** (scheduled task disabled, quota). → Coverage table is dated and inside the grounding; reader rule 8 declares staleness in every answer. Detection sits with the reader, which is the component that keeps running.
2. **Cowork breaks append discipline** (rewrites a journal instead of appending). → First spike test before anything else is built (three consecutive runs, old lines byte-identical). Fallback: daily journal files instead of monthly (more files, zero rewrites). Recovery: OneDrive version history. Monthly files bound the blast radius either way.
3. **Wrong-client attribution** — a wrong presence, not an absence: it speaks, and says something false. → Domain-based inclusion in config; `[captured, unvalidated]` tags; human validation on judgment; deletion of capture errors in validation.
4. **A client silently drops out of coverage** (config typo, lost section). → The coverage table enumerates clients *from the config*, zeros included: a missing row is visible, a zero-for-days pattern is visible.
5. **The 50-file cap creeps closer.** → Validation session counts files and warns above 40; old journals move to Archive (out of grounding, never deleted).
6. **Someone edits STATE outside validation.** → Header notice ("changes outside validation sessions are not tracked"); low risk — it is the user's own file.
7. **The user skips validation for weeks.** → Not a failure but a **declared mode**: capture continues, the reader reports the unvalidated share and the `Last validated` date.
8. **A stale prompt keeps running in Cowork** after the kit evolves. → Each prompt carries a version line; capture echoes it in the coverage table header, so a divergence is visible at every validation.
9. **A source drops out mid-run** — Box's connector is withdrawn, Teams errors — and its emptiness is indistinguishable from "nothing happened there". Observed on 2026-08-19, three consecutive runs. → Coverage is per source: a source that could not be read keeps its own `covered_until`, so the next run that finds it available sweeps the window it missed, and the source line in `PENDING.md` names the gap on every run until it closes. The declaration alone was not enough — `PENDING.md` is rewritten every run and cleared at validation, so a note about an outage outlives it by minutes. **What is ephemeral cannot be the record**: the file that says how far coverage extends has to be the file that remembers the gap.
10. **A quick update and a queued proposal disagree** — the user records a fresh fact via "Update my brain" while `PENDING.md` still holds capture's older proposal on the same item; the next validation session re-applies the outdated proposal on top of the fresh fact. → The quick update checks `PENDING.md` first and annotates the superseded proposal (under confirmation) so validation sees the warning next to it.

When adding a piece to this architecture, the question is not "does it work?" but **"how do I notice when it has stopped working?"**

## 12. Distribution kit

This repository is the kit:

```
copilot-second-brain/
├── README.md                        ← the product: what, why, how
├── SETUP.md                         ← self-service setup guide (~10 min, 4 steps)
├── LICENSE                          ← Apache-2.0
├── prompts/
│   ├── onboarding-interview.md      ← pasted into Cowork once (Step 1)
│   ├── capture-task.md              ← pasted into a Cowork scheduled task (Step 2)
│   ├── agent-instructions.md        ← paste block for agent builder (Step 3)
│   └── channel-backfill.md          ← one-off: pull a newly added channel's history
├── skills/                          ← Cowork skills, copied to Documents/Cowork/skills/ (Step 4)
│   ├── validate-my-brain/SKILL.md   ← periodic validation session
│   └── update-my-brain/SKILL.md     ← quick user-observed-fact update
├── docs/ARCHITECTURE.md             ← this document
└── deck/                            ← presentation sources
```

Rules: prompts and the skill are edited **here first, then pasted/copied** into Cowork / agent builder; every prompt carries a version line; **no user data, configuration or state ever enters this repo** (principle 6).

## 13. Open items

| Item | Type |
|---|---|
| Verify agent-builder grounding includes/excludes subfolders as assumed | Feasibility |
| monday.com connector enablement → reactivate §8 | Suspended feature |
