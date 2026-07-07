---
name: thought-tracks-past-session
description: Backfill the thought-tracks store with a model of a PAST session. Reads a specific saved transcript (.jsonl) you point it at, seats it at its real dates, and writes an Output A model (mental model patterns, a synthesis, and predictions) into the same model store the live skill uses. Use this whenever the user wants to "backfill", "model an old or past session", "run thought tracks on a previous session", "seat my history", or "fill in the timeline" with sessions that already happened. It models a transcript on disk, NOT the current conversation, and it writes no diff. Like the main skill, it takes no interview input, the transcript is the only data.
---

# Thought Tracks: Past Session (backfill)

This is the backfill companion to `thought-tracks`. The live skill models the session you are in right now. This one models a session that already happened: you point it at a saved transcript, and it seats that session in the store at its real dates, so the reasoning trajectory is built from when sessions actually happened rather than from the day the user started keeping models.

It produces **Output A only**, the model of how the user reasoned in that past session. It deliberately writes **no diff (no Output B)**. See "Why no diff" below; that is the whole reason this is a separate skill.

The sacred constraint from the main skill still holds: **no interview.** The transcript is the only data. Do not ask the user about themselves. The only thing you need from them is *which* transcript.

## What you need from the user

A pointer to one past session's transcript. Accept any of:
- a full path to a `.jsonl` file,
- a session id (the filename stem), or
- enough of a description that you can find it.

If you were given a full path, use it. If you were given an id or a description, look wherever your agent saves its session transcripts (often `.jsonl` files) and find the matching one with Glob first. If several plausibly match, list them and let the user pick the right one. That is choosing the input, not interviewing them about themselves.

## Workflow

### Step 1: Read the session's real dates from the transcript

A saved transcript is usually JSON Lines: one JSON object per line, and most lines carry a `timestamp`. Two common gotchas:
- The **first line is often a header** (metadata like a title and a session id) with **no timestamp**. Skip it when looking for the start.
- So `date_start` = the date on the **first line that has a timestamp**, and `date_end` = the date on the **last line**.

Pull these without loading the whole file just for dates. Read only the head and tail: in PowerShell, `Get-Content '<path>' -TotalCount 3` shows the head (confirm which early line first carries a timestamp) and `Get-Content '<path>' -Tail 1` shows the end; on Unix, `head -n 3` and `tail -n 1`. Parse the `timestamp` out of those lines. Do not echo transcript content while doing this, read only the timestamp field.

If `date_start` and `date_end` fall on the **same calendar day**, the session was a single day; record one `date`. If they span days (started the 3rd, last activity on the 17th), record both `date_start` and `date_end`. That span is the session's real duration, and capturing it is a stated reason this skill exists.

Also grab a label: a title in the header is a good basis. Slugify it for the filename.

### Step 2: Read the transcript as the trace, and extract Output A

Now read the transcript itself as the session data. **This file is the data, not the conversation you are currently in.** Attend to reasoning *method*, not topic, exactly as the main skill does.

Each line is a JSON object; the signal is in the user and assistant message turns. Tool call lines and system reminders are mostly noise, skim them. If the file is larger than one read, read it in chunks by offset; do not skip the user's turns.

Write **Output A** using the shape defined in these two files (read them first if you have not this session):
- `references/output-templates.md` (the **Output A** block), and
- `references/worked-example.md` (a completed example).

Same structure and voice as the live skill, defined in those two files. Write it in the **third person, using the user's name** when the transcript reveals it, otherwise "the user," as a standalone portrait a stranger or a future model could read cold. Plain prose, no em dashes. The skeleton:
- **`H1` title:** `# <session label> · <project> · <date, or date_start to date_end>`.
- **Context:** one line, written so a reader who knows nothing about the project follows everything below.
- **`## Mental model patterns`:** as many `H3` observations as the session genuinely shows (do not cap, do not pad). Each is a plain English heading that stands on its own, then four beats in order: what it means, a concrete example written for a reader with zero context, "most people do Z," and what this means.
- **`## What it says about <Name>`:** one short synthesis of how they construct reality, every claim tied back to a pattern above.
- **`## The shadow`:** the cost or failure mode of the strengths, about the person.
- **`## Predictions`:** 3 to 5 observable predictions. On a backfill these are retrodictive (you read this session after it ended and write them with the answer already in hand), so keep them honest and record the provenance in frontmatter (`extraction: backfill`), not in the heading.
- **`## Honest limit`:** what this read cannot support (one mode, hype as mood, a thin slice).

Privacy: model the reasoning, never reproduce secrets, credentials, tokens, or personal data that happen to be in the transcript. If the session pasted a key or private content, describe the *move* ("pasted a token to debug auth"), not the content.

### Step 3: Write the model file (no diff)

Write into the **same store the live skill uses**: `~/.thought-tracks/` (on Windows, `%USERPROFILE%\.thought-tracks\`). If the store is missing entirely, confirm with the user before creating it; a missing store can be a path mistake rather than a true first entry.

The filename is **number prefixed and marked as a backfill**: `NNN-backfill-YYYY-MM-DD-<label>.md`, where `NNN` is the next running number in the store (one more than the highest number anywhere; if the store holds unnumbered date named files from an earlier version, count them and start after them, per the main skill's legacy note) and the date is `date_start`. The number is production order, when you backfill it now, not the session's date; the `backfill` marker and `extraction: backfill` say it is not a live forward bet. Never overwrite an existing file, and never renumber the files already in the store; this run just appends the next number.

Frontmatter:
```yaml
---
output_at: 2026-06-15T18:30:00Z   # UTC, ISO 8601 with Z: when first written; permanent ordering key, never changes
# updated_at: 2026-06-20T10:00:00Z # add ONLY if you re-run this backfill in place; leave output_at and the number alone
date_start: YYYY-MM-DD     # session dates, from the transcript; or a single `date:` when start and end are the same day
date_end: YYYY-MM-DD       # omit when a single day
session_label: "<label>"
artifact_type: "extract (model_n), backfill, Output A only"
source: "thought-tracks-past-session (saved transcript)"
source_session_id: "<8 character transcript id>"
project: "<project>"
extraction: backfill
---
```
Do not assign a live `instance:` number, that field is for live forward bets only. The backfill's place in the store is its filename number (production order, when you backfill it); its session date lives in `date_start`. It takes the next running number but never anchors the live diff. Emit Output A in the conversation as well, so the user sees what was stored. **Then append a row to the store's `index.md`**, in the table's column order: `| NNN | <output_at> |  |  | <session start> | <session end> | backfill | <filename without .md> |  |` (the blank cells are `updated_at`, `inst`, and `note`; a backfill has no instance). The index is a derived view of the portraits' frontmatter, ordered by `output_at`, so the row lands at the bottom; if the index does not exist yet, create it with the header the main skill's Step 4 describes.

## Why no diff (do not add one)

The live skill's Output B is a falsification check: the prior model committed to predictions *before* this session existed, and the diff scores which of them broke. That guard only means something forward in time. A past session is read *after* you already know how it ended, so any "prediction" about it is a retrodiction, written with the answer in hand, by the same single extractor voice. Diffing one backfilled model against another would manufacture exactly the post hoc, always plausible story the main skill exists to prevent. So:
- Backfilled models are **standalone snapshots.** Output A only.
- They are **never used as diff anchors** by the live skill. The live prediction chain runs only through live models.
- Their predictions are kept for shape and for possibly chaining them again later, but they are explicitly retrodictive and soft. Say so in the file.

They still integrate into the **timeline** by date: once seated, a backfilled May session sorts before the June models, so a later chronological read of the store shows the real trajectory. That ordering comes from the frontmatter dates, not from the filename.

## Honesty rules

All of the main skill's honesty rules apply (bias toward the performed self, don't flatter, no psychoanalysis, describe observable moves only). Two more, specific to backfill:
- **Retrodiction is soft.** You know how the session ended before you "predict" it. Never present a backfilled prediction as if it had been committed in advance.
- **One voice across many sessions.** Every backfilled model is written by the same extractor in one sitting, so consistency across sessions is partly your own style, not proof of stable structure in the user. Flag that the backfilled stretch shares an authorial voice.
