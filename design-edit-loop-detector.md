# Flag an agent that edits the same file again and again

`edit-loop-detector` · Task · feature, shuttle, friction · running · Code-Bouncer

## Seen (orch, 2026-11-17)

Nothing stops an agent that edits the same file again and again without progress; only the run's time budget ends it.
Weaver's cold attempts and longer unattended runs make such a loop costlier.
Shuttle already gets every `PostToolUse` hook event (`shuttle-hook-signals`).
Source: the survey F17 (`orchestrator-survey-ideas`), from Bernstein, which kills an agent that edits the same file four or more times within five minutes.

## Idea

- Shuttle counts edits per file per agent session from its `PostToolUse` events.
- Past a set count in a set window, it raises a friction note to the run's parent, without killing the session.
