# Spike capture prompt

**Throwaway spike material — not part of the kit.**

This is not one of the four kit prompts (`prompts/`). It exists only to run the
append-discipline spike described in `2026-08-append-discipline.md` and is deleted with
the rest of the spike material once the verdict is recorded.

Paste the block below into Cowork **verbatim and unmodified** for each of the three test
runs. Do not edit it between runs — a fair test means the same prompt, run three times.

---

## Prompt (paste everything between the lines)

```
Target file: Documents/Cowork/Second Brain Spike/JOURNAL-spiketest-2026-08.md

Do not read any other file.

1. If the target file does not exist yet, create it empty.

2. Read the target file's current content and count how many lines it already contains.
   Call that number `existing_lines`. Set `n = (existing_lines / 3) + 1`, using integer
   division (an empty file has existing_lines = 0, so n = 1; after one prior run it has 3
   lines, so n = 2; after two prior runs it has 6 lines, so n = 3).

3. Append exactly three new lines to the end of the file, in this exact form, replacing
   only `<n>` and `<k>`:
   `- 2026-08-13 · note · spike run <n> line <k> — source: spike "append test" from spike-runner, 2026-08-13 · [captured, unvalidated]`
   Use `<k>` = 1, then 2, then 3, in that order, so the three new lines read
   "spike run <n> line 1", "spike run <n> line 2", "spike run <n> line 3".

4. Never modify, rewrite, reformat or reorder existing lines; only append at the end of the file.

Do nothing else: do not summarize the file, do not reformat it, do not touch any other
file or folder.
```

---

## Why the lines are fixed

Every value in the appended lines is fixed except `<n>` and `<k>`, and both of those are
computed from a simple line count rather than from anything Cowork has to interpret or
judge. The type (`note`), tag (`captured, unvalidated`), and source
(`spike "append test" from spike-runner, 2026-08-13`) never change between runs. This
means the expected content of every line is known in advance, which is what makes the
byte-identical diff in the protocol a real test of append discipline rather than a test
of whether Cowork wrote plausible-sounding text.

The line format follows the journal grammar from the architecture (`docs/ARCHITECTURE.md`
§5.1):

```
- YYYY-MM-DD · <type> · <one-line fact> — source: <kind> "<subject/title>" from <sender>, YYYY-MM-DD · [<tag>]
```
