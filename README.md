# Thought Tracks

**Every AI memory tool learns what you prefer. None learn how you think. Nor how your thinking evolves.**

Your AI remembers you like dark mode and TypeScript. It has no idea that you go after the cause, not the symptom, that you get suspicious the moment an answer looks too clean, that you won't trust your gut until something backs it up. Your reasoning is the most valuable thing you bring, and the one thing nothing keeps.

And it does not hold still. The way you reasoned a year ago is not the way you reason now. Somewhere in between you picked up a move and dropped a habit. You'd never have caught it happen. Your change in reasoning is the most interesting thing about you and the only one your AI doesn't keep a record of.

So the change goes by unseen. You can reopen last month's code. You cannot reopen last month's thinking. The session ends, the moves evaporate, the next one starts from zero, and the drift of how you reason is recorded nowhere.

## Ask your chatbot "how do I think?" and watch it flatter you

You have probably tried it. The answer comes back fluent, generous, and impossible to check. It calls you a creative systems thinker and you nod, because who wouldn't. That is a horoscope. It reads warm, says nothing, and can never be wrong.

Thought Tracks is the opposite, on purpose. It earns every line three ways:

- **It cites.** Every claim points to a real moment in your session. No moment, no claim.
- **It commits.** Each portrait ends with falsifiable predictions about your *next* session, the kind that can be embarrassingly wrong. "Will reject the first clean proposal and rebuild the frame." Then it checks them.
- **It tracks change.** Every new portrait is diffed against the last, and real movement gets separated from noise. You see when your thinking actually shifts, not when the wording did.

A flattering summary cannot do any of those. That is the whole point.

## What it does

It works on any chatbot conversation. ChatGPT, Claude, Gemini, whatever you talk to. At the end of a chat it reads what happened and writes a portrait of **how you reasoned**, your method and not your topic. The moves you made to produce thought, in plain language, with the receipts.

It takes nothing from you. No form, no interview, no journal prompt. The conversation already happened, and that is the data. You never describe yourself, because the version of you that fills out a form is the performed one. The trace is the real one.

Then it does the part nobody else does. It bets on your next session. The time after that, it grades the bet and tells you what changed.

## Why nothing else does this

We checked. Across 88,000+ published skills and the user modeling research, the tools that claim to know you all stop at the same wall:

| What gets modeled | Thought Tracks |
|---|---|
| What you **prefer** (dark mode, Python, short answers) | not this |
| Who you **are** (Big Five, MBTI, a personality score) | not this |
| How you **feel** (mood, sentiment, tone) | not this |
| **How you reason** (the moves, the method, the structure) | **this** |

Memory tools store facts. Journaling apps track feelings. Digital twins copy your writing voice. Not one of them models the reasoning itself, commits to a prediction about it, or versions it over time. Thought Tracks is the first that does.

## What you get

- **A portrait of your reasoning that stands on its own.** Readable cold by a stranger, or by another model. It names what you do, shows the move, and says what it means about how you build.
- **A record of how your thinking changes.** Session by session, the real shifts surface and the noise gets quarantined. You can watch your method move.
- **A brief you can hand to another AI.** Drop the portrait into a fresh chat and the model reasons more like you do, because for once it has your method instead of just your preferences.

See a full portrait before you install: [references/worked-example.md](references/worked-example.md).

## How it works

Two ways to run it, the same portrait either way.

**Installed as a skill** (in Claude Code, Cursor, or any agent that supports skills), say "run thought tracks" at the end of a session. It reads the whole transcript, writes the portrait, and diffs it against your last one. Automatic, every time, and your record builds itself.

**In any other chatbot,** paste one prompt as your final message. It reads the conversation you just had and emits the same portrait. Back home, one command ("import this portrait") files it into your record and the picture keeps building.

Both produce the same thing: a cited portrait of how you reasoned, with falsifiable predictions for next time. You are always adding to one record of how you think, and the installed skill diffs each new run against that record and shows you what held, what moved, and what is just noise.

## Honest by design

It will not only find good things. A portrait that flatters is broken, so it names your blind spot as readily as your strengths, and it tells you when the trace is too thin to be sure. On your first run there is nothing to diff against, and it says so instead of inventing a trend.

## Use it

Install it as a skill:

```bash
npx skills add aassoiants/thought-tracks
```

Then end any session with "run thought tracks."

In any other chatbot, there is nothing to install. Paste the portable prompt as your last message and copy back the portrait it writes. The prompt is in [references/portable-closeout-prompt.md](references/portable-closeout-prompt.md). Then say "import this portrait" where the skill is installed and it files itself into your record, checked against the honesty guards on the way in.

And to fill in your history, **backfill** a past session straight from its saved transcript.

## License

MIT
