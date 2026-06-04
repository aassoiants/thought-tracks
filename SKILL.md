---
name: thought-tracks
description: At the end of a working session, passively analyze the conversation trace to extract a model of HOW the user reasoned (their method, not the topic), then diff it against the previous stored model to surface deltas. Use this whenever the user asks to "map my thinking", "model how I think", "run thought tracks", "close out this session", do a "reasoning retrospective", "capture this session", "update my model", or wants to track how their thinking is changing over time. Trigger at session end even when phrased casually ("what did you learn about how I think", "diff me against last time"). Do NOT require the user to answer questions or enter data, the session itself is the only input.
---

# Thought Tracks

Reads a finished session and produces two things: a model of how the user reasoned *this* time, and a diff against the last model. The job is to map reasoning **method**, the moves the user makes to produce thought, not the subject matter they happened to think about.

The whole design rests on one decision: **the skill takes no input from the user.** No interview, no journaling, no questionnaire. The session trace is the data. The user runs the skill; the skill reads what already happened and emits the artifacts. Asking the user to fill in a form defeats the entire point, so read this again if you are tempted to prompt them for clarification about themselves.

## Why this is built the way it is

A passive reader of a conversation will *always* produce a plausible model and a set of changes that sounds plausible, because language models are fluent at coherence after the fact. That fluency is the central risk: without a guard, this skill becomes a horoscope generator that feels accurate every time and can never be wrong. Two design constraints exist solely to fight that:

1. **Every model commits to falsifiable predictions** about how the user will behave in the *next* session. Vague traits ("thinks in systems") cannot be checked and are forbidden as anchors. Observable moves ("will demote a factual claim to its generating cause before evaluating it") can be checked against the next trace.

2. **The diff separates DRIFT from WOBBLE.** A difference between two models is only trusted as real change (DRIFT) if it is *backed by the trace*, ideally by a prediction the prior model made that this session violated. Differences that are just two summaries phrased differently are WOBBLE and must be quarantined, not reported as the user evolving.

If you drop either constraint, the skill still produces output, it just produces output that cannot be trusted. Don't.

## The store

Thought Tracks keeps its models in a single global store at `~/.thought-tracks/` (a folder in the user's home directory, independent of any one agent; on Windows, `%USERPROFILE%\.thought-tracks\`). One store across all of the user's projects, because the skill models the person, not the project. Create the directory if it is missing. The reference files this skill ships with (the output templates and the worked example) live alongside this `SKILL.md`, in the skill's own `references/` folder.

## Workflow

### Step 1: Locate the prior model

Prior instances live in the store, one file per session, named `YYYY-MM-DD-<label>.md`. **Order the store by date, never by filename:** sort the files by frontmatter `date_start` (fall back to `date` for a single day session), then `date_end`, then `instance` (a backfill carries no `instance`; treat that as lowest, so a same day live run sorts after a same day backfill).

`model_{n-1}` is the **most recent prior model in that order**. It is usually the previous live session, but it may be a backfilled snapshot if that is what sits most recently before now. If the chosen prior is tagged `extraction: backfill`, you may still diff against it, but flag in Output B that its predictions are retrodictive in origin (written after that past session had already ended) even though they were fixed before this live session, so treat them as a softer anchor. Read the chosen model's Output A block as `model_{n-1}`. Set `n` to one more than the highest `instance` anywhere in the store (backfills carry none and do not count), so the live instance sequence stays monotonic no matter how much history gets backfilled. If the user pasted a model into the session, prefer that as `model_{n-1}`. If the store has no model at all, this is **instance 1** (`n` = 1), run extraction only, skip the diff, and say so plainly. Do not fabricate a baseline to diff against.

### Step 2: Extract `model_n` from the session trace

Read the *entire* session, attending to reasoning moves, not topics. The topic is scaffolding; the method is the signal.

Write the model in the **third person, using the user's name** when the trace reveals it, otherwise "the user." It is a portrait that has to stand on its own, readable by a stranger or by a future model asked to reason the way this person does. Plain prose, no em dashes.

The model has a fixed skeleton. Use the exact headings in `references/output-templates.md`, and see `references/worked-example.md` for a completed one.

**Title (`H1`).** The session label, the surface or project, and the date or date span. Example: `# API rate limiter design + rollout · payments service · Mar 3-9, 2026`. So every stored file is identifiable at a glance.

**Context (one line).** Project or repo, the surface (which agent, chatbot, or API), and the kind of work (build, debug, analysis, planning), written so a reader who knows nothing about the project still follows everything below. Infer it from the environment and trace, never ask. It also situates the diff in Step 3.

**`## Mental model patterns`.** The core of the model: a run of `H3` observations, **as many as the session genuinely exercised. Do not cap the count and do not pad it.** Discover them fresh from THIS session; an analysis session and a build session surface different patterns, and forcing a prior session's patterns onto this one is the failure to avoid. Each observation is complete in itself: a plain English heading that stands on its own, then four beats in order.
- **The heading** is a clear claim in ordinary words, understood from the heading alone (`### Changes the frame instead of patching the surface`). Not a jargon label like "frame control move."
- **What it means:** unpack the heading into the actual move.
- **A concrete example:** what they were working on or solving, written for a reader with zero context. Gloss any term specific to the project, ask what a stranger or a future model would need to know, then supply it. Quote real moments.
- **Most people do Z:** name the default the move deviates from, because a move is only legible as distinctive against a baseline.
- **What this means:** what the move reveals about how they build or think.

**`## What it says about {Name}`.** One short synthesis: what the patterns together say about how they construct reality and what they treat as real. This is the shareable centerpiece, and it may make a larger interpretive claim on one condition: every claim traces back to a pattern named above. This is the climb from "what they do" to "what it means," the step that separates a portrait from an inventory.

**`## The shadow`.** The cost or failure mode of the strengths, about the person and not about the model's reliability (that goes under Honest limit). Where does this way of working expose them? A model that only finds good things is broken.

**`## Predictions`.** 3 to 5 concrete, observable predictions about the next session, the kind that would embarrass you if wrong. **This is the one hard, falsifiable part of the model**, and the anchor the next diff bites on. Test each: *could the next trace prove this false?* If not, rewrite it until it can.

**`## Honest limit`.** What this read cannot support: a single mode sampled, hype that is mood and not method, too thin a slice to call structure. This is the caveat about the *data*, kept separate from the shadow.

The interpretive sections (patterns, what it says, the shadow) are the portrait, and they may interpret freely **as long as each claim is pinned to a cited moment in the trace.** That anchor is the whole defense against a horoscope. The predictions stay the falsifiable spine. Keep the two jobs distinct.

### Step 3: Diff `model_n` against `model_{n-1}`

The diff has two anchors, and they carry different weight. Keep them separate.

**The hard anchor: predictions (this is where the diff actually bites).** Take each prediction `model_{n-1}` made and check it against this session's trace:
- **HELD**: the predicted move happened. That is stable structure, and naming it is as valuable as naming change.
- **DRIFT (trusted)**: the prior model predicted X and the session did not X. This is the strongest, most trustworthy signal in the whole skill, because it was committed to in advance and the trace overruled it. A clearly different move visible in the trace itself (not just in word choice) also counts, but a violated prediction is the gold standard.
- **Untested**: the session never created the occasion to confirm or refute the prediction (common when the modes differ). Say so; do not score it either way.

**The soft anchor: recurring patterns (a ledger, explicitly weaker).** Because the patterns are discovered fresh each session, they will not line up slot for slot, so match them by *meaning*, not by name. Note which patterns from prior instances reappear here (a pattern recurring across several sessions is mounting soft evidence it is real structure) and which are absent. Treat this as suggestive only: an absent pattern is just as likely an untested mode as a real change, and pattern recurrence is **never** hard evidence on its own. The predictions police; the pattern ledger only hints.

**WOBBLE (quarantined, applies to both anchors).** A difference with no trace backing (rephrasing, the assistant's vocabulary bleeding into the model, a topic shift dressed up as a method shift, two patterns that are the same move under different names) is noise. List it separately, marked as probable noise. Never report wobble as growth.

**Context check, before trusting any drift.** Compare the two instances' contexts. If they differ (different project, surface, or kind of work), a method difference is most likely driven by context: the same user in a different mode, not a changed user. Lean toward WOBBLE and overfit, and say so, unless a violated prediction shows the method itself changed within comparable contexts.

### Step 4: Emit and store

Output the artifacts in the conversation, Output A then Output B when a prior model exists, using the templates. Then persist this run: create the store directory (`~/.thought-tracks/`) if it is missing, and write a new file whose name starts with the date, `YYYY-MM-DD-<label>.md` (the date is this session's `date_start`; `<label>` = the short session label, slugified). If a file with that start date already exists, add a two digit sequence for that day (`YYYY-MM-DD-02-<label>.md`) so names stay unique. The date prefix makes the directory read chronologically and lets a backfilled older session slot in by date without renaming anything that already exists.

Open the file with a YAML frontmatter block carrying `instance` (= `n`), `date` for a single day session (or `date_start` and `date_end` when the session genuinely spanned more than one day), `session_label`, `artifact_type`, `source`, and `extraction: live`. The dates give every stored model explicit, machine readable provenance and let Step 1 order the store by time. A live closeout is almost always one day, so `date` is today; if you want the exact start, it is the first timestamped line of the current transcript. Put both Output A and (when there was a prior) Output B in the file.

Never overwrite an existing file. Each session is its own file, so the full trajectory stays recoverable and Step 1 can always find the most recent prior model by date.

## Honesty rules

- **Early instances are mostly WOBBLE.** One or two prior sessions make a terrible baseline, and the assistant's own phrasing contaminates the model. Say this honestly when it is true, "not enough trace yet to separate signal from noise" is the correct output for instance 2, not a confident trend. Faking early confidence is the failure mode.
- **Bias toward the performed self.** A passive reader only sees sessions where the user showed up. The model will overfit to whatever mode they were in. When a later session violates that, read it as "model was overfit," not "user changed", and say so.
- **Don't flatter.** This maps reasoning; it is not a compliment generator. Note sloppy moves, dropped threads, and unexamined priors as readily as strengths. A model that only finds good things is broken.
- **Infer worldview, not interiors.** Reading how the user constructs reality from the moves they actually made is the job, and it is what makes the model worth sharing. But never invent motives, history, diagnoses, or feelings the trace does not show. The test is one line: every interpretive claim, including the synthesis under "What it says about {Name}", must point back to a cited moment. Inference from evidence is the work; projection past it is the failure.

## Output

Two artifacts, in order, using `references/output-templates.md`:
- **Output A**: `model_n`, the portrait: an `H1` title, a one line Context, `## Mental model patterns` (the `H3` observations), `## What it says about {Name}`, `## The shadow`, `## Predictions`, `## Honest limit`.
- **Output B**: the diff (held + drift on the predictions, the soft recurring pattern ledger, quarantined wobble, explanation)

For instance 1, emit Output A only and state that the diff begins next session.

See `references/worked-example.md` for a completed pair, so the shape is concrete before you write.
