# Agents still try the forms lym's guards refuse, at a turn each

`hook-refused-turns` · Note · enhancement, process, claude

## Seen (orch, 2026-11-01)

In the five heaviest Webster forks of the three latest tapestry runs, 6 turns were refused by lym's PreToolUse guards: four Python attempts and two raw `go build` or `go vet` runs.
Each refusal costs a turn at the fork's full context, and the fork then makes the call again in the allowed form.
The forks inherit Merriam's system prompt, which carries the guards' steering sentences, yet still reach for the refused forms first.

## Idea

Find why an agent that is told the rules still tries the refused forms, from the transcripts of the refused turns, and close the gap where it lies: the steering sentence's wording or place, the stencil's own instructions for building and vetting, or a guard that answers with the allowed command ready to run.
