# Worked Example

A completed model, so the shape is concrete before you write one. The person here is fictional, a composite built to show the format. The session modeled is a debugging and design session on a billing service: chase down a flaky payment webhook, then design a retry system so it stops happening. The diff at the bottom is illustrative, showing how a later session would compare against this one.

This is illustrative, not a rule. Match the user's actual moves; don't force this one's patterns onto your subject. Write in the third person using the user's name. Plain prose, no em dashes. Discover the patterns fresh, and surface as many as the session genuinely shows rather than hitting a number.

The model below uses real heading levels (H1 title, H2 sections, H3 observations), so it renders as the actual file would.

---

# Payment webhook debug + retry redesign · billing service · Mar 3-9, 2026

**Context.** A working session in a billing service's code repo: a payment provider's webhook (the callback the provider fires to say a charge succeeded or failed) was firing unreliably, sometimes double charging, sometimes going silent, and Devon had to find the cause and design a retry system so it stops. Modeling how Devon worked the problem, not what got shipped: the procedure he ran, not the patch.

## Mental model patterns

### Devon maps how it breaks before he reads how it works
Devon lays out the failure surface before the happy path. Handed the flaky webhook, his first question was not "what does the handler do" but "what are all the ways a webhook arrives wrong: twice, never, out of order, or hours late?" He listed those four before reading a line of the handler. Most people trace the working path first and treat failures as exceptions to it. To Devon the failure modes are the real specification, and the happy path is just the one case that happens to be going right at the moment, so the work starts by naming every way the thing can betray you.

### Devon won't stop at the first cause
Devon treats the first explanation as another symptom and keeps descending. The double charge traced to a bad idempotency key (the token that is supposed to make a repeated request do nothing the second time), and he refused to fix it there. He asked what let a bad key through, found the retry wrapper rebuilt the request from scratch, asked why that dropped the original key, found a shared helper that defaulted a missing key to the current timestamp, and only stopped when he hit the one condition that produced every layer above it. "Every why has a why under it. The bug lives at the bottom, not where it bit you." Most people stop at the first cause that explains the symptom and ship the fix. Devon assumes the first cause is just the lowest one he has reached so far, and climbs down until he hits something nothing else produced.

### Devon won't believe a story until it fails in front of him
Devon refuses to act on an explanation he has not watched happen. When the assistant offered a plausible reason for the double fire, he waved it off: "I don't want a theory about why it's flaky, I want to watch it flake." He wrote a script that replayed the webhook two hundred times against staging until the duplicate charge reproduced on command. Most people read the logs, form a hypothesis, and start fixing. Devon distrusts any explanation that has not been forced to happen on demand, because an unreproduced bug is a guess wearing a lab coat.

### Devon fixes the class, not the case
Devon won't patch the one bug in front of him when he can kill the kind. Once he found the shared helper that defaulted the idempotency key, he did not just fix the webhook that paged him. He asked which other handlers called the same helper, found three more that could double fire the exact same way, and fixed the mechanism so none of them could. "If I only patch the one that paged me, the next one is already written and just hasn't fired yet." Most people fix the failure they were handed. Devon treats one failure as a single instance of a class, and goes after the class.

### Devon builds the brake before the engine
Devon's first move on the retry system was not the retry. It was the stop. "Before I make this thing retry, I answer one question: if it retries a million times at 3am, how do I shut it off without shipping a deploy?" A kill switch and a hard ceiling on attempts existed before the retry logic they were meant to contain. Most people build the feature and add the safety afterward, if there is time. Devon builds the brake before the engine, because the failure he is most afraid of is the one he cannot stop by hand.

### Devon turns his own fix into the next suspect
Devon aims the same distrust at his own solution that he aimed at the bug. After the retry system shipped, he did not call it a win. He asked what the fix itself now made possible, and named it: "This adds a queue, and a queue is a new thing that can back up. I traded a sharp failure for a slow one, a double charge for a charge that lands an hour late. Better, but not free." Most people present a fix and move on. Devon runs his own work through the same acid he ran the problem through, and books the new liability before someone else finds it.

## What it says about Devon
Devon starts from the assumption that the system is broken in a way he has not found yet, and he does not stop at the first thing he turns up. He refuses to believe any story until it reproduces, and climbs past the first explanation to the one that produced it. Then he fixes the whole class rather than the case, and turns the same suspicion on his own patch. Underneath all of it is a flat assumption that the failure has a real bottom: one true generating cause, reachable if you keep descending, and the job is to reach it rather than to ship the first plausible mitigation.

## The shadow
The descent that makes Devon thorough also makes him slow, and occasionally bottomless. Not every bug has a deep generating cause worth chasing to the floor; some are a typo, and the right move is to fix it and leave. The instinct to reach the true root and kill the whole class can gold plate a one line change, and "keep going until you hit the bottom" has no built in place to stop. Devon is exposed where the depth outruns the stakes, where chasing the real cause costs more than the symptom ever would have, and he is least likely to notice it there, because going deeper always feels like the rigorous thing to do.

## Predictions
Falsifiable against the next session:
- Will refuse the first root cause and keep asking what produced it until he reaches a condition nothing else explains.
- Will refuse to act on a bug report until it reproduces on command, and will build a way to reproduce it if one does not exist.
- Will fix the class of failure rather than the single instance that fired.
- Will build the kill switch, ceiling, or rollback before the thing it is meant to contain.
- Will turn his own fix into the next suspect and name the liability it introduces, unprompted.

## Honest limit
This is one incident driven debugging and design session, the exact mode that rewards descending to the root, on a high stakes payment path where the depth is warranted. It says little about how Devon reasons in greenfield design with no failure to anchor on, or when the job is to explore and expand rather than harden, or under a deadline that punishes the slow deep pass. Read it as how Devon works a system that is actively betraying him, not how he works when the stakes are low or the canvas is blank.

---

**Output B, the diff (illustrative).** Shown against a hypothetical earlier session, to make the diff shape concrete. The predictions are the hard anchor; the recurring pattern ledger is a soft, weaker signal.

**Reasoning diff · instance {n-1} → {n}**

**Contexts.** {n-1}: an earlier session on the same service, designing a brand new billing feature from scratch, a build with no bug to chase. → {n}: this session, an incident, debugging the flaky webhook and designing the retry. The register is different (building something new versus hunting a failure), which makes a method *difference* suspect as context driven, but makes a held *prediction* across that gap stronger evidence of real structure.

**HELD (predictions that fired, stable structure):**
- The prior model predicted Devon would reject the first version of a thing and climb to what sits under it. In the build session that was rejecting the first feature spec to reach the actual job it served; here it was descending from the double charge to the shared helper that produced it. The same move, climb past the first answer to the thing that generated it, held across a build and a debug. That it survived the change of register is the strongest signal in this diff.
- The prior model predicted Devon would build the stop before the thing it contains. Fired again: the kill switch before the retry.

**DRIFT (real movement, backed by the trace):**
- None trusted. "Fix the class, not the case" reads new, but it is the same descend to the generating cause pointed one step outward: once he reached the shared helper, fixing the class is just naming everything that helper produces.

**Untested predictions:**
- The prior model predicted Devon would turn the same scrutiny on his own work. Partly tested: he named the queue's new failure mode, but there is no later session yet to see whether that gets revisited or quietly filed. Score it weakly.

**Recurring patterns (soft signal, NOT hard evidence):**
- Reproduce before believing recurs (the replay script). In the greenfield build there was nothing to reproduce, so its absence there was an untested mode, not a real change. Soft evidence it is real structure.
- Fix the class recurs in spirit: he generalized the feature for every case in the build, and generalized the fix for every handler here.

**WOBBLE (quarantined, probable noise):**
- The shift from building a feature to debugging a webhook is a change of task, not of method.
- "Failure surface" and "generating cause" are the extractor's words, not necessarily Devon's.

**Explanation.** The core move, climb past the first answer to the cause that produced it, was predicted from a build session and fired in a debug session, a genuinely different register. A prediction that holds across that gap is the gold standard: it says the move is structure, not a reaction to the situation. Nothing drifted, the pattern ledger softly agrees, and the one open question is whether Devon turns the same descent on his own fixes over time.
