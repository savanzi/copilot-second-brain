Version: 1.3 (2026-08-19) — edit in the copilot-second-brain repo first, then paste here.

# Channel backfill — Copilot Second Brain

A **one-off** job for one situation: you just added a Teams channel to a client's
`Teams channels` in `brain-config.md`, and the channel has history the brain has never seen.
The scheduled capture only looks forward (from the watermark); this job looks backward, once.

**How to use:** replace the two placeholders below, then paste the whole thing into a normal
Cowork chat — never into a scheduled task, and never touch the watermark to "look back"
(that would make the next scheduled run re-sweep mail it already captured, duplicating it).

---

One-off backfill task — Copilot Second Brain. This is a single-run job, not the scheduled
capture. Work ONLY on the Teams channel named below. Do NOT touch `_watermark.json`, do NOT
sweep Outlook or Box, do NOT touch any STATE file.

Everything you write is in English, whatever language the channel is in: translate each fact
as you distill it, and keep proper nouns, product names and quoted titles verbatim.

Channel: "<CHANNEL NAME>"
Client: <CLIENT NAME> (config section `## Client: <CLIENT NAME>` in
`Documents/Cowork/Second Brain/brain-config.md` — read it for the client's `Key`, Keywords
and context).

1. Open the channel and read its ENTIRE history, from the very first message to now. If you
   cannot find or access the channel, stop and say so explicitly — that is the most
   important possible outcome of this run.

2. For each message or thread, apply this threshold: does it still matter for understanding
   where the project stands today, or is it a durable fact (a decision, a commitment, a
   date, a risk, a sensitivity)? Routine chatter and logistics that no longer matter are
   skipped.

3. For each fact that passes, append one line to the client's journal for the **current**
   month, `Documents/Cowork/Second Brain/JOURNAL-<key>-YYYY-MM.md`, exact grammar:
   `- YYYY-MM-DD · <type> · <one-line fact> — source: Teams message "<channel/thread>" from <sender>, YYYY-MM-DD · [captured, unvalidated]`
   The date is the message's own date, even if months old. Types: email-in, email-out,
   meeting, decision, delivery, revision, blocker, unblock, note. Never modify existing
   lines — append at the end only. Distill, never paste message bodies. If a fact records a
   secret shared in the channel (a password, a token), record THAT it was shared and that it
   should be rotated — never the secret itself.

4. Do NOT touch `PENDING.md`. The lines you just appended sit below that client's last
   `ritual` marker, which is exactly where the scheduled capture looks for its proposals —
   so the next run picks them up on its own, however old the messages are, and keeps
   re-proposing them until a human walks them. Writing a section here would only be
   overwritten by that run.

5. Close by reporting in chat: how many messages you read, the date of the oldest one, how
   many journal lines you appended, and anything you could not access.

The backfilled facts reach the next validation session by themselves, as proposals from the
scheduled capture, alongside everything else awaiting validation. Nothing is lost if you do
not validate immediately.
