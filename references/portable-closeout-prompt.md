# Portable closeout prompt

A copy and paste version of Thought Tracks' closeout, for a chat in any tool where the skill is not installed. The live skill reads the current trace and the backfill skill reads a saved `.jsonl`; both need this skill and its store. This prompt needs neither. Paste it into any LLM chat (ChatGPT, Gemini, claude.ai, anything) at the end of a session and it emits the same portrait as text, using the conversation in the model's own context as the only data.

It produces **Output A only** (the portrait). No diff, by decision: the diff is a forward falsification check that only bites inside the live chain, and a foreign chat has no prior model to anchor it. This is the same "Output A only" shape the backfill subskill writes.

## How to use it

1. At the end of a session in another LLM's chat, paste the block below (everything inside the fence) as your last message.
2. If your name does not appear anywhere in that chat and you want the portrait named, start your paste with `My name is <your name>.` on its own line. Otherwise it writes "the user."
3. The model emits one markdown block: frontmatter, then the portrait. Copy it.
4. To keep it, save it into your store (`~/.thought-tracks/`) as `YYYY-MM-DD-<label>.md` with the real date. It seats in the timeline by date. It carries `extraction: foreign` and does **not** anchor the live diff chain.

**Foreign portraits never anchor the diff.** A portrait written inside another chat feeds the timeline only, never the hard prediction chain, so saving one into the store is a manual step by design. This mirrors the backfill rule: only live models commit predictions forward and get scored against the next session.

**Guard note.** The prompt carries the skill's honesty guards (cite every claim, falsifiable predictions, name the shadow, no invented interiors). The risk unique to this path: the extractor is the same model you were just talking to, so it is portraying its own conversation partner and will tend to flatter. The Honest limit clause says so. Weight a foreign portrait a notch softer than a live one.

---

## The prompt (copy everything inside the fence)

````text
You have just finished a working session with me. Close it out by modeling HOW I reasoned in this conversation, not what it was about. The topic is scaffolding; the method is the signal. Take no new input from me and ask me no questions about myself: this conversation is the only data. Read the whole thing.

Look for the moves I make to produce thought: how I frame a problem before working, what I treat as settled versus what I make prove itself, where I change the whole approach versus patch the surface, what I trust, what I defer, how I open and close. Discover the patterns fresh from THIS conversation. Do not force a fixed number and do not pad.

Voice and format: third person, using my name if it appears in this conversation, otherwise "the user." A standalone portrait a stranger could read cold, so gloss anything specific to our work and quote real moments. Plain prose. No em dashes. No jargon labels for the patterns; each heading is a plain claim that stands on its own.

The guards. These are the point. Without them this is a horoscope that flatters me and can never be wrong:
- Every interpretive claim must point to a real moment in this conversation. Quote or paraphrase the actual exchange. If you cannot cite it, cut it.
- Predictions must be falsifiable. For each, ask "could my next session prove this false?" If not, it is a trait, not a prediction; rewrite it until a future session could break it.
- Do not flatter. Name the cost and the failure mode as readily as the strengths. A portrait that only finds good things is broken. You are describing the person you have been talking with, so watch your own pull toward flattering me.
- Infer how I construct reality from the moves I actually made. Never invent motives, history, feelings, or diagnoses the conversation does not show.

Output only the block below, starting at the frontmatter. No preamble, no closing remarks. Emit raw markdown (do not wrap it in a code fence). Use this exact skeleton:

---
date: YYYY-MM-DD
session_label: "<short label for this session>"
artifact_type: "extract (model_n), foreign port, Output A only"
source: "portable closeout prompt (<which app or model this chat is>)"
extraction: foreign
---

# <short session label> · <which app or model this chat is> · <today's date if you know it, else leave YYYY-MM-DD>

**Context.** <one line: the kind of work this was (build, debug, analysis, planning, writing) and on what, written so a reader who knows nothing about it follows everything below. Infer it; do not ask.>

## Mental model patterns

### <a plain English heading that is a clear claim on its own>
<What it means: unpack the heading into the actual move.> <A concrete example from this conversation, written for a reader with zero context: gloss any specific term and quote the real moment.> Most people <the default this move deviates from>. <What this means: what the move reveals about how I build or think.>

### <next observation, same four beats: what it means, a cited example, "most people do Z", what it means>

### <as many as the session genuinely shows. Do not cap. Do not pad.>

## What it says about <my name, or "the user">
<One short synthesis: what the patterns together say about how I construct reality and what I treat as real. It may make a larger claim, but every claim ties back to a pattern named above.>

## The shadow
<The cost or failure mode of these strengths: where this way of working exposes me. About me, not about your reliability as a model.>

## Predictions
<3 to 5 concrete, observable predictions about my next session, the kind that would embarrass you if wrong. Each must be checkable against a future trace.>
- <observable move>
- <...>

## Honest limit
<What this read cannot support: one mode of mine sampled, hype that is mood and not method, too thin a slice to call structure. Then add this clause verbatim: This portrait was written by the same model that was in the conversation, following a pasted spec, not by a dedicated outside reader, so the extraction is unverified and may flatter me.>
````

---

## Shape reference (do not paste this; it shows what one good observation looks like)

This is one completed `### ` observation in the four beats, so the shape is concrete. It is an example of the *form*, not a pattern to go find. Discover the real ones fresh from the actual conversation.

> ### Devon builds the brake before the engine
> Designing a retry system for a flaky payment webhook, the first thing Devon built was not the retry, it was the kill switch. "Before I make this thing retry, I need to answer one question: if it retries a million times at 3am, how do I stop it without shipping a deploy?" The off switch and a hard ceiling on attempts existed before the retry logic they were meant to contain. Most people build the feature and bolt on the safety afterward. To Devon the failure they are most afraid of is the one they cannot stop by hand, so they build the brake before the engine.
