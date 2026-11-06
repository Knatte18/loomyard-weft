# Measure what the session scope fix gives in a real launch

`session-scope-fix-measure` · Task · research, claude, shuttle · running · Drift-Verify

## Seen

`c6b5ca66` made the session scope write `disableWorkflows: true`, to drop the `Workflow` tool and its larger schema.
Its change description says the proof asked for was never run: a real launch with the first-request tokens stated against the 37,330 of `docs/research/tokens/session-scope-launch-2026-11-06/report.md`.

## Wanted

Measure what the fix gives, in a real launch, and report it in `docs/research/tokens/session-scope-fix-launch-2026-11-06/report.md`.

Use the method and the role of the launch trial's report, from a `lym` that holds `c6b5ca66`, in the sandbox hub `lyt`.
Run exactly two launches with `lym shuttle open`: one spawned role and one orch, both with the scope on, and no headless launch.

Report:

- The first-request tokens of each, against the trial's 37,330 for the spawned role and 34,383 for the orch.
- Whether the `Workflow` tool is gone from the session.
- Whether a skill the session preloads is still there.
- Which plugin skills stay listed, as the fix now states they do.

## Bounds

No code change: if the fix does not hold, the report states what was seen and the fix becomes a note.
