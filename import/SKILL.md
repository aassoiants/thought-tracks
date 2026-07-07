---
name: thought-tracks-import
description: Pull a foreign portrait (one emitted by the portable closeout prompt in an outside chatbot, e.g. ChatGPT, Gemini, claude.ai) into the thought-tracks store. Takes the pasted portrait block or a path to a file holding it, assigns the next store number and output_at, writes NNN-foreign-<label>.md, appends the index row, and audits the honesty guards. It writes no diff; a foreign portrait never anchors the prediction chain. Use whenever the user wants to "import a portrait", "pull in a foreign closeout", "save this portrait from another chatbot", or pastes a markdown block carrying "extraction: foreign".
---

# Thought Tracks: Import (foreign portraits)

This is the import companion to `thought-tracks`. The live skill models the session you are in; the backfill skill models a saved transcript. This one stores a portrait that was **already written somewhere else**: the user ran the portable closeout prompt (`references/portable-closeout-prompt.md`, next to the main skill) in an outside chatbot, copied the emitted block, and is bringing it home. Your job is seating, not authoring.

The division of labor is deliberate. The outside model cannot know the store's numbering or the moment of import, so the emitted block carries **provenance only** (date, label, source, platform, `extraction: foreign`). **Identity is assigned here**: the running number and `output_at` come from this skill at import time.

The sacred constraint from the main skill holds here too: **no interview.** Do not ask the user about themselves. You may ask which block or file to import, and for the real session date if the outside model left a placeholder; that is choosing the input, not interviewing.

## What you need from the user

The foreign portrait, as either:
- a markdown block pasted into this conversation (frontmatter plus portrait), or
- a path to a file holding that block.

If neither is present, ask for one. Nothing else is required.

## Workflow

### Step 1: Validate the block

Confirm the pasted text is actually a foreign portrait:
- YAML frontmatter with `extraction: foreign`, a `session_label`, a `source`, and a `date` (or `date_start`/`date_end`).
- A body with at least `## Mental model patterns`, `## Predictions`, and `## Honest limit`.

If the frontmatter is missing `extraction: foreign` but the block is unmistakably a portable closeout output (the skeleton matches), add the field and tell the user you did. If the block is not recognizably a portrait, **stop and say so**; do not coerce arbitrary text into the store.

If `date` is still the `YYYY-MM-DD` placeholder (outside models often do not know the date), ask the user what day the session happened (UTC) and fill it in.

`platform` may be missing on blocks emitted by an older prompt; derive it from `source` (the parenthetical names the app or model) and add it.

### Step 2: Resolve the store and the next number

Write into the **same store the other skills use**: `~/.thought-tracks/` (on Windows, `%USERPROFILE%\.thought-tracks\`). If the store directory is missing, confirm with the user before creating it: a foreign portrait can genuinely be someone's first ever entry, but a missing store can also be a path mistake, and the two look identical from the inside.

The next number `NNN` is one more than the **highest number anywhere in the store** (any kind: live, backfill, foreign), zero padded to three digits. If the store holds unnumbered date named files from an earlier version, count them and start numbering after them (see the main skill's legacy note). Never renumber existing files; this run appends.

### Step 3: Write the file

Filename: `NNN-foreign-<label>.md`, where `<label>` is the frontmatter's `session_label`, slugified.

Content: the pasted block, **body verbatim**. Never rewrite, trim, polish, or correct the foreign model's prose; the store keeps what was actually written, flaws included, because the flaws are evidence of how well the guards traveled. Touch only the frontmatter:
- add `output_at`: the UTC timestamp of this import, ISO 8601 with `Z` (e.g. `2026-07-03T18:30:00Z`). This is the permanent ordering key; it never changes afterward.
- add `platform` if you derived it in Step 1, and the real `date` if you filled the placeholder.
- do **not** add an `instance:` field. That field is for live forward bets only; a foreign portrait's place in the store is its filename number.

Never overwrite an existing file. If you are re-running an import of a portrait already in the store, leave its `output_at`, number, and row position alone and set `updated_at` (UTC) instead, same as the other skills.

### Step 4: Audit the guards (report, never rewrite)

Read the portrait once and score it against the guards the prompt carries:
- **Cited:** does each pattern quote or concretely anchor a real moment, or does it assert unanchored traits?
- **Falsifiable:** could a future session prove each prediction false, or are they traits in disguise?
- **Shadow:** is there a real cost or failure mode, specific to this person, or a compliment dressed as one?
- **Barnum:** would the patterns fit most thoughtful professionals, or do they identify someone?

Report the verdict per guard in the conversation, briefly. This audit is the standing validation of the foreign tier: a portrait that comes back flattering and uncited means the pasted guards are not holding in that chatbot, and the user needs to see that trend.

The audit **never blocks a structurally valid import and never edits the portrait**. If two or more guards come back weak, put a short flag in the index row's note column (e.g. `guards weak: uncited patterns, generic`), so any later synthesis across the store can weight or skip it. The user can always delete the file; that is their call, not yours.

### Step 5: Append the index row

Append one row to the store's `index.md`, in the table's column order:

`| NNN | <output_at> |  |  | <session start> | <session end> | foreign (<platform>) | <filename without .md> | <note, blank unless the audit flagged it> |`

The blank cells are `updated_at` and `inst` (a foreign portrait has no instance). The index is a **derived** view of the portraits' frontmatter, ordered by `output_at`, so this row lands at the bottom; fix any later classification in the portrait's frontmatter, never by hand in the table. If the index does not exist yet, create it with the header the main skill's Step 4 describes.

## Why no diff (do not add one)

The live skill's Output B is a falsification check: the prior model committed to predictions before the session existed, and the diff scores which broke. A foreign portrait sits outside that chain twice over: its session was never predicted by a prior model, and its extractor is an unknown model portraying its own conversation partner, the softest tier in the trust model. So:
- Foreign portraits are **standalone snapshots**. No Output B, ever.
- They are **never used as diff anchors** by the live skill (the live skill already skips them when choosing `model_{n-1}`).
- They **feed the timeline**, weighted a notch below live and backfill, and any synthesis across the store should skip or discount a row the audit flagged.

## Privacy

The portrait may quote sessions from anywhere the user works, including ground that is not theirs alone. The user gauges the block's sensitivity before importing; storing it is their decision, already made by the act of pasting. Do not echo the portrait's content back beyond what the audit verdict needs, and never reproduce quoted material outside the stored file.
