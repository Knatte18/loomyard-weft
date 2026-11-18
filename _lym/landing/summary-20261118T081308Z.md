# Shell guard refusals say nothing ran and name the refused part

A shell guard deny refuses the whole Bash call.
Agents that put a probe or scratch step in front of read-only work, such as a version check before a grep, read the old refusal as being about the probe alone.
They then worked out by hand which of the other parts had run.
None of the old steers said that nothing had run.

Every steer in `internal/shellguard`'s table, plain and per engine, now opens with one shared clause:

> Nothing in this command ran: the guard refuses the whole command for one part, so run the parts you still need again without it. The refused part: …

The row's own text follows the clause, so the refusal goes on to name the refused part, for example `python and python3 are refused in every lym session…` or `rm on a computed path… is refused here`.

## Design decisions

- **The clause is added once, in `Rows`.**
  `openSteersWithWholeCommandRefusal` prefixes each plain steer and each engine steer, so Claude and Codex get the same text with no change to either engine's renderer.
- **Notices are unchanged.**
  A notice announces a guard in the system prompt before any command runs, so the clause does not belong there.
  The raw go row's notice is still its steer text plus a full stop, minus the clause.
- **The refused part is named by the row, not quoted from the command.**
  The guard is a static `grep -E` over the payload JSON.
  A match runs from the start of the `command` value to the refused word, so it holds the earlier segments too and can end partway through a JSON escape.
  Quoting it back would need per-engine extraction and escaping in the hook command.
  Each row's steer already says what it refuses, so the shared clause points the agent to that text.
- **Out of scope:** claudeengine's own Bash deny of `lym webster` in a fork subagent is not part of the shell guard table and keeps its text.

## Tests

- `TestRows_SteersOpenWithWholeCommandRefusal` asserts that every row's steer, on Claude and on Codex, opens with the clause and then names a refused part.
  It also asserts that no notice carries the clause.
- `TestCheckTexts` and `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` now expect the clause before the python refusal and when they compare the raw go notice with its steer.
- `internal/shuttleengine/claudeengine/testdata/settings-golden.json` holds the new deny reasons.
