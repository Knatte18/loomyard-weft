# Card 14 — Adopt reports one line per run

**Create:**
- `internal/adoptcli#runReport`
- `internal/adoptcli#runReports`
- `internal/adoptcli#reportInput`
- `internal/adoptcli#TestRunReports`

**Edit:**
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#outcome`
- `internal/adoptcli#adoptFields`
- `internal/adoptcli#Command`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#world`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#TestAdoptFields`
- `internal/adoptcli/doc.go`

**Uses:**
- `internal/adoptcli#fixRunHalted`
- `internal/adoptcli#subjectDir`
- `internal/fronttask#OpenFront`
- `internal/fronttask#ActionAdoptCheck`
- `internal/adoptcli#ReadCheck`
- `internal/adoptcli#checkAlive`
- `internal/adoptcli#newWorld`

**Intent:**
D7: adopt's envelope carries `runs`, one item per run under the named subjects, sorted by slug, beside the per-slug outcomes it keeps.

`runReport` carries `slug`, `state`, `row` (the batten run record's current producer, omitted for an entry with no record), `owned_by` when owned elsewhere, `caller` for `paused`, and a plain `line` such as `dobby-land-notice-needs-flag: waiting for the adopt check at Code-Bouncer`.
`reportInput` carries this machine's name, the runs before the reconcile, the moved runs after it, the held slugs per subject, the subjects whose check this call started, and the owners and board entries read after the owner write.

The runs are the union of the moved runs before the reconcile, the moved runs after it, the released tasks card 11 held and each subject's open front task (`fronttask.OpenFront`).
`runReports` gives each the first state that applies, in this order:
- `owned_elsewhere`: its entry's effective owner after the owner write is another machine, with `owned_by`.
- `landed`: it was a moved run before the reconcile and its board entry is absent or done after it, an entry removed by hand on the other machine included.
- `held_failed_check`: a `HoldCaller` entry of its own scope stands and its subject's check record reads failed, or running with its process gone (ended without a verdict), and this call started no check of the subject.
  Card 11 starts a check of every subject with a standing hold, so in adopt's own envelope this state is reached when that start failed, such as a subject worktree that does not resolve (card 5); the record it reads is the previous check's.
- `waiting_for_check`: a `HoldCaller` entry of its own scope stands otherwise; a held released task has no row.
- `paused`: a live entry of its own scope of another caller stands, naming that caller.
- `halted`: this machine owns it and `fixRunHalted` reads its run record halted, the escalation read through the new `EscalationEligible` seam; the line names the record's halt reason, and for the subject's fix task (front, `on_land` `adopt-check`) the re-run of `lym adopt <subject>`.
- `ended`: it was a moved run before the reconcile, its entry after it reads a status that is neither empty, a run status nor done, such as the draft a cancel on the other machine writes, and its run record is gone; the line names the status.
- `running_here`: any other run this machine owns, the fix task included; an unstarted fix task has no row.
A read that fails returns its error, which adopt adds to the envelope's `warnings` naming a re-run of the same `lym adopt`, leaving `runs` empty; the steps already taken stand.

New seams `CheckState(subject)` (the subject's record through `ReadCheck` and `checkAlive`: running, green, failed, ended without a verdict, or none) and `EscalationEligible(slug)` are filled by `hubDepsAs`; `outcome.Runs` is emitted by `adoptFields` as `runs`, always present, an empty array for none.
`Command`'s help names `runs` and its states.

Tests:
- `TestRunReports` over the fake `world`: one row per state, `owned_elsewhere` with `owned_by`, `landed` for an entry done and for an entry pruned, `held_failed_check` for a failed record and for a running record whose process is gone, each with the subject's check start failed in this call, and `waiting_for_check` for the same records when this call started the subject's check, `waiting_for_check` for a held run with its row and for a held released task with none, `paused` naming the caller, `halted` for a record in `blocked`, `failed`, and `awaiting` with no admitted escalation, and for a fix task stopped on a machine failure naming `lym adopt <subject>`, `awaiting` with an admitted escalation reading `running_here`, `ended` for a run cancelled on the other machine naming its status, `running_here` for an unstarted fix task with no row; the precedence when a held run is also landed; the sort by slug; a run record read that fails leaves `runs` empty and adds a `warnings` line naming the re-run of the same `lym adopt`, the other steps' outcomes standing.
- `TestAdopt` gains: the envelope's `runs` holds every moved and held run; a second adopt right after the first reports the same states for the runs not done.
- `TestAdoptFields` asserts `runs`.

`adoptcli/doc.go`'s envelope section names `runs` and its states.

**ImpactSummary:** adopt's envelope gains `runs` built from reads it already makes plus two new seams; the per-slug outcomes stay.

**Commit:** 14: adopt reports one line per run under its subjects
