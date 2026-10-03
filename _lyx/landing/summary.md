# Friction reaches GitHub on its own, under step and on halts
Friction from a loom run now reaches GitHub as issues without an operator.
Before this change the reflection ran only under `lyx loom run`, so no driven run (every driver uses `lyx shed step`) ever reflected, and ly-drive's self-report died in its drive report unless someone approved it.
Filtering of what was filed moves to the later board intake, not before filing.

## What changes

- **Friction-Reflect under `step`.**
  `reflectFrictionRow` drops its `run`-only gate and reflects whenever the friction directory is non-empty.
  The row still returns Done, and done persists only after the reflection returns, so batten's teardown still waits for it.
- **Halts reflect under both verbs.**
  A `step` or `run` that leaves the run `blocked` or `failed` writes a Go-authored halt note (producer, state, reason, history position, trace path) and then runs the non-waiting reflection.
  Under `step` this happens in the step process before the envelope is written, and step envelopes now carry the `friction` status key (`shedverbs` gains an `AfterStep` hook).
  `awaiting` and `paused` write and reflect nothing.
  `run` previously reflected on `blocked` only; `failed` is new.
- **Webster refusals leave a note.**
  Every refusal from a `lyx webster` verb except `validate` writes one Go-authored note with the verb, arguments, refusal text and class, when the friction directory resolves.
  A write failure only logs and never changes the verb's exit or envelope.
- **`frictionengine.Reflect` is re-entrant.**
  It records the covered notes in a non-`.md` record file before spawning.
  On entry it attaches to a live reflection agent instead of spawning, or archives the covered notes when a finished report exists.
  Archives move only the covered notes, so a note written mid-reflection stays for the next one.
  Friction-Reflect's interrupt policy stays `reinvoke`, now justified by the attach probe.
- **Reflection stencil.**
  The agent files every distinct lyx problem, merges duplicates, drops only notes about the task's own code or describing nothing wrong, and ends each issue body with a provenance line (task slug and note file names).
  Issue bodies may not quote task source, diffs, plan text, credentials, environment values or absolute home paths.
  The report is now a success marker, written only after filing succeeded; a failed `lyx selfreport create` ends the turn without it, so Reflect returns `failed` and archives nothing.
- **ly-drive.**
  Repair and re-step records go to the envelope's `friction_dir` when non-empty.
  The driver files nothing; the `## Self-report` section is replaced.
  A `done` stop with `friction: failed` notifies the parent.
- **Docs.**
  `CONSTRAINTS.md` gains a Friction Capture invariant; `docs/overview.md`, the `internal/friction` and `internal/frictionengine` docs, the loom recipe comment and the `InterruptPolicies` doc are updated.

## Design decisions to check

- The report is the only signal that filing succeeded, since Go never parses it.
  A reflection killed mid-filing leaves no report, so the next one refiles the issues it had already created.
- A failed terminal reflection still lets `done` persist; the notes then survive only in the run records and the `archive/` tag.
  The parent notification is the alert.
- A halt reflection delays the step envelope by up to `friction_timeout_min`; a held reflection lock skips it rather than waiting.
- No deduplication against open GitHub issues, and no new label.
- Tier 1 anomaly filing (`detectAndFileAnomalies`) stays `run`-only, and the board entry's later note that it should also run under `step` is not implemented here.
- Per-agent end-of-session self-report sessions are out of scope; halt notes, refusal notes and repair records are taken as covering them.

## Not run by hand

- No real reflection agent has filed a real issue.
  The tests use a fake shuttle and a fake issue filer, including a stepped run whose Webster row hits `plan_drifted`, blocks, and reflects over both the refusal note and the halt note.
  Worth one real driven run through a halt to see the issues land on `Knatte18/loomyard`.
- Deploy the stencil with `update-plugins.sh`: the board's copy of `friction-template-reflection` is older than the worktree source.
