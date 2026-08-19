# Spike: append discipline

## Verdict

> Date: 2026-08-16
> Result: **PASS**
> Anomaly (if any): none. Line counts exact after every run (3 / 6 / 9), run number `n`
> computed correctly each time, and both diff checks produced no output — the lines from
> runs 1 and 2 were byte-identical after run 3. Verified by hand the same day.
> (Observation, not an anomaly: one read during run 2 briefly showed the pre-run content
> while OneDrive was still syncing — re-reading a few seconds later resolved it. Worth
> remembering when checking capture output right after a run.)

---

## What this tests

The architecture (`docs/ARCHITECTURE.md` §2 principle 2, §5.1) assumes the scheduled
Cowork capture task can append a line to a journal file, run after run, **without ever
rewriting, reformatting or reordering the lines already there.** That assumption has
never been verified against the real product. §11.2 names the failure mode this spike
tests directly: *"Cowork breaks append discipline (rewrites a journal instead of
appending)"* — and the countermeasure it lists is exactly this spike, run before anything
else is built.

The result gates Task 2 (journal file design): **PASS** → journals stay one file per
client per month, as designed; **FAIL** → fall back to one journal file per capture run
(`JOURNAL-<client>-YYYY-MM-DD.md`), trading more files for zero rewrite risk.

This protocol takes about 15 minutes and produces a binary PASS/FAIL verdict, recorded
above.

## Steps

### 1. Create the spike folder

In OneDrive, inside Cowork's working directory, create a scratch folder:

```
Documents/Cowork/Second Brain Spike/
```

(Locally this is under your OneDrive sync root, e.g.
`~/Library/CloudStorage/OneDrive-<tenant>/Documents/Cowork/Second Brain Spike/` — adjust
to your own sync path.)

Everything created in this folder is deleted at cleanup (step 5). Nothing in it is part
of the kit.

### 2. Run the spike prompt three times, saving a copy after each run

Open `spike-capture-prompt.md` (same folder as this file) and copy its prompt block into
Cowork **verbatim**, unmodified between runs.

Run it three times. After **each** run, before running it again, save a local copy of the
target file (`JOURNAL-spiketest-2026-08.md`) into the spike folder under a new name:

- After run 1 → save a copy as `spike-run1.md` (should contain 3 lines)
- After run 2 → save a copy as `spike-run2.md` (should contain 6 lines)
- After run 3 → save a copy as `spike-run3.md` (should contain 9 lines)

If any of these line counts don't match, note it as an anomaly in the verdict — Cowork
added the wrong number of lines, which is itself a finding worth recording even before
the pass/fail check below.

### 3. Pass criterion

Cowork passes the spike if, after run 3, the lines written by runs 1 and 2 are still
exactly what they were when first written — byte for byte, nothing rewritten, reformatted
or reordered.

Two checks, both against the run-3 copy, both expected to produce **no output** (no
output = files identical = pass):

```bash
cd "$HOME/Library/CloudStorage/OneDrive-<tenant>/Documents/Cowork/Second Brain Spike"
# ↑ replace <tenant> with your own OneDrive sync folder suffix (see step 1)

# Check A — run 1's 3 lines untouched after run 3
diff <(head -3 spike-run3.md) spike-run1.md

# Check B — run 1's and run 2's 6 lines untouched after run 3
diff <(head -6 spike-run3.md) spike-run2.md
```

**Pass/fail is binary**: both checks produce no output → **PASS**. Either check produces
any output (a line added, changed, or reordered) → **FAIL**. There is no partial pass —
one changed character in an old line is a FAIL, because that is exactly the failure mode
this spike exists to catch.

### 4. Record the verdict

Fill in the **Verdict** block at the top of this file: today's date, PASS or FAIL, and any
anomaly observed (including line-count mismatches from step 2, even if the diff checks
themselves passed).

### 5. Cleanup

Delete the whole spike folder from OneDrive:

```
Documents/Cowork/Second Brain Spike/
```

This removes the spike journal and all three saved copies. Nothing from this spike is
meant to persist — the verdict recorded in step 4 is the only lasting output.
