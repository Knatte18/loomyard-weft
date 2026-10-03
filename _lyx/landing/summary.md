# Friction reaches GitHub on its own, under step as under run

Every loom run's friction now reaches GitHub as issues with no operator in the loop, under `lyx shed step` and `lyx loom step` as well as `lyx loom run`.
Before this change the reflection ran only under `run`, but every driven run uses `step`, so no driven run ever reflected.
The hub filed #350 and #352 by hand at a Webster block; with this change the halt files them.
Filtering moves to the later board intake, so the reflection no longer triages.

## What changes

- **Reflection under step.**
  `reflectFrictionRow` drops its `armedVerb == "run"` gate.
  The row still returns Done, and done persists only after the reflection returns, so batten's teardown never runs before it.
- **Halt notes.**
  A `step` or `run` that leaves the run `blocked` or `failed` first writes a Go-authored halt note (producer, state, reason, history position, trace path), then runs the non-waiting reflection.
  Under `step` this happens before the envelope is written, and the step envelope gains the `friction` status key `run` already carries.
  `awaiting` and `paused` write nothing.
  `run` previously reflected on `blocked` only.
- **Webster refusal notes.**
  Every refusal from a `lyx webster` verb except `validate` writes one Go-authored note (verb, arguments, refusal text, class) when the friction directory resolves.
  A write failure only logs.
- **Tier 1 anomaly filing under step.**
  `detectAndFileAnomalies` also runs under `step`, with its existing skips, and does not double-file the halt note.
  A failed `FileIssue` is kept as a pending title and body in the filed-title marker and retried on every later pass, so a resumed halt is still filed.
- **Re-entrant `frictionengine.Reflect`.**
  Reflect records the covered notes in a non-`.md` record file before spawning, probes for a live reflection agent through shuttle's attach seam, and archives covered notes without spawning when a finished report exists.
  A note written during a reflection stays for the next one.
  A killed step process no longer leads to a second agent refiling the same notes.
- **Reflection stencil.**
  The agent files every distinct problem in lyx tooling, merges duplicates, drops only items not about lyx, and ends each issue body with a provenance line (task slug, note file names).
  A content rule keeps task source, diffs, plan text, credentials, environment values and absolute home paths out of issue bodies, since `Knatte18/loomyard` is public.
  The report is now a success marker: the agent writes it only after filing succeeded.
- **ly-drive.**
  Repair and re-step records go into the envelope's `friction_dir` when it is set.
  The driver files nothing itself, and at a `done` stop with `friction: failed` it notifies the parent.
- **Docs.**
  New Friction Capture invariant in `CONSTRAINTS.md`; `docs/overview.md` and the `internal/friction` and `internal/frictionengine` docs updated.

## Design decisions to check

- Go never parses the report, so its existence is the only proof filing succeeded.
  A failed `lyx selfreport create` ends the turn without a report: Reflect returns `failed`, nothing is archived, and a done stop carries `friction: failed`.
- A reflection killed mid-filing leaves no report, so the next one refiles issues it had already created.
- A halt reflection delays the halting step's envelope by up to `friction_timeout_min`; a held reflection lock skips it rather than waiting.
- Friction-Reflect's interrupt policy stays `reinvoke`, now justified by the attach probe.
- `internal/friction` stays a leaf: halt and refusal note writing lives in `loomcli` and `webstercli`/`websterengine`.
- Filing goes through `lyx selfreport create` only; there is no dedupe against open issues and no new label.
- A terminal reflection failure still lets `done` persist; refiling then takes a person, alerted by the driver's parent notification.

## Not run by hand

- The integration suite reported OK on the final commit; no real reflection was driven end to end against GitHub.
  Check once that a stepped run with a Webster `plan_drifted` block files issues through `lyx selfreport create` and archives its notes.
- The board copies of the stencils have drifted from the worktree sources; a production deploy (`update-plugins.sh`, operator-run) is needed before the new reflection stencil is live.
