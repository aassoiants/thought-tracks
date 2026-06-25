# Output Templates

Use these verbatim as the skeleton. Keep entries concrete and short, this is a changelog, not an essay. Cite moments in the session rather than asserting traits.

---

## Stored file frontmatter

Every file in the store (`~/.thought-tracks/`) opens with a YAML block. The dates are what Step 1 sorts the store by, so they carry the provenance, not just the Output A header.

```yaml
---
instance: 6                   # live runs only; a backfill omits this (its date is the identity)
date: 2026-06-01              # a single day session
# For a session that genuinely spanned days, drop `date` and give the real span instead:
# date_start: 2026-05-03
# date_end: 2026-05-17
session_label: "short label"
artifact_type: "extract (model_n) + diff (Output A + B)"   # backfill: "extract (model_n), backfill, Output A only"
source: "thought-tracks skill run"           # backfill: "thought-tracks-past-session (saved transcript)"
extraction: live              # or: backfill
---
```

The filename starts with the date (`YYYY-MM-DD-<label>.md`, using `date_start`); add a two digit sequence for that day (`-02-`) only when a start date already has a file. The store is ordered by the frontmatter dates, not by filename. See SKILL.md Steps 1 and 4.

---

## Output A: `extract(session) → model_n`

Third person, using the user's name when the trace shows it, otherwise "the user." A standalone portrait, readable cold. Plain prose, no em dashes. Headings exactly as below.

```markdown
# {Session label} · {surface or project} · {date or date span}

**Context.** {Project or repo, surface (which agent, chatbot, or API), and kind of work (build, debug, analysis, planning), written so a reader who knows nothing about the project follows everything below. One line. Inferred, never asked.}

## Mental model patterns

### {A plain English heading that stands on its own: a clear claim, no jargon}
{What it means: unpack the heading into the actual move.} {A concrete example of what they were working on or solving, written for a reader with zero context: gloss any project term, quote real moments.} Most people {the default this move deviates from}. {What this means: what the move reveals about how they build or think.}

### {Next observation, same four beats}
{...}

### {...as many as the session genuinely exercised. Do not cap. Do not pad.}

## What it says about {Name}
{One short synthesis: what the patterns together say about how they construct reality and what they treat as real. The shareable centerpiece. It may make a larger claim, but every claim ties back to a pattern above.}

## Product building choices
{The forks where two or more options were genuinely live and they committed to one. As many `### ` entries as the session shows, and none at all if it shows none. Discovered fresh from this session, attributed to their own move and not the assistant's default they accepted.}

### {Plain English heading naming the choice}
- **What it builds:** {the kind of thing this choice produces in the product, for a cold reader, quoting the moment.}
- **What they chose against:** {the option they killed and what it had going for it. Reconstruct the rejected branch even when they never said it aloud.}
- **What it implies (conjecture):** {what it says about how they think a product should be. A guess from this session only, marked as such, never an established trait.}

### {next fork, same three beats, or omit the whole section on a thin session}

*{Optional closing line: the session's product philosophy read off the forks together. Distinct from "What it says about {Name}", which is how they reason toward truth; this is what they treat as a good product. Drop it if it would only restate that.}*

## The shadow
{The cost or failure mode of the strengths, about the person: where this way of working exposes them. Not the model's reliability, which goes under Honest limit.}

## Predictions
{3 to 5 concrete, observable predictions about the next session, the kind that would embarrass you if wrong. The one hard, falsifiable part. On a backfill these are retrodictive, written after the session ended; keep this heading and record the provenance in frontmatter (`extraction: backfill`).}
- {Observable move 1}
- {...up to 5}
- {taste bet, when the session had product building choices: the strongest fork criterion written as a falsifiable prediction, e.g. "faced with store versus derive, they derive." Mark it as a taste bet so the diff groups it.}

## Honest limit
{What this read cannot support: one mode sampled, hype that is mood not method, too thin a slice to call structure. The caveat about the data, separate from the shadow.}
```

**Prediction quality test:** before writing each prediction, ask "could the next session prove this false?" If not, it's a trait, not a prediction, so rewrite it.

- GOOD: "Will reject a clean or flattering output and demand the symmetry pass."
- BAD: "Thinks in systems." (unfalsifiable, always true, anchors nothing)

---

## Output B: `diff(model_n, model_{n-1}) → deltas`

```markdown
**Reasoning diff · instance {n-1} → {n}**

**Contexts.** {n-1}: {project · surface · kind of work}. → {n}: {project · surface · kind of work}. {If they differ materially, the method differences below are suspect as driven by context; weight them toward WOBBLE.}

*The predictions are the hard anchor; the recurring pattern ledger is a soft, weaker signal. Keep them apart.*

**HELD (predictions that fired, stable structure):**
- {Prediction from model_{n-1} that came true this session. Naming these matters as much as naming change.}

**DRIFT (real movement, backed by the trace):**
- {What moved}. *Evidence:* {moment in this session}. {Strongest form, name it: "model_{n-1} predicted X; session did not X." A clearly different move visible in the trace also counts, but a violated prediction is the gold standard.}

**Untested predictions:** {Predictions the session never created an occasion to confirm or refute, common when modes differ. Listed, scored neither way.}

**Recurring patterns (soft signal, NOT hard evidence):**
- {Pattern from a prior instance that reappears here, matched by meaning not by name. Recurrence across several sessions is mounting soft evidence it is real structure. An absent prior pattern is noted too, but is as likely an untested mode as a real change.}
- {Recurring fork criterion from a prior instance's product building choices, matched by meaning. A criterion that holds across different kinds of product is soft evidence of a stable product taste. Prior taste bets are checked above among the predictions, not here.}

**WOBBLE (quarantined, probable extractor noise, NOT reported as change):**
- {Difference with no trace backing: rephrasing, assistant vocabulary bleed, topic shift mislabeled as method shift, two patterns that are the same move under different names.}

**Explanation.** {2 to 4 sentences. Did the predictions hold or break, and is any drift consistent (same underlying method extending to new ground) or scattered? What does the balance say about whether there's enough trace yet to trust the trend? Lean on the predictions, not the pattern ledger, for any confident claim.}
```

For **instance 1**: emit Output A only. End with: "No prior model exists, the diff begins next session."

For **instance 2 to 3**: expect mostly WOBBLE. Say so honestly rather than manufacturing a trend.
