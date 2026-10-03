# Card 9 — drive-skill-filing

**Prosa:**
- `plugins/ly/skills/ly-drive/SKILL.md`

**Edit:**
- `internal/loomcli/driveskill_test.go`

**Uses:**
- `internal/loomshed#InterruptPolicies`

**Intent:**
ly-drive writes its friction into the run's friction directory and files nothing itself; the run's reflection is the one filer (D4).

Skill text changes, branching only on `friction_dir` and the envelope's `friction` key, with no row or recipe name:

- `## Repair records and the stop report`: repair records and automatic re-step records go into the envelope's `friction_dir` when it is non-empty, and under `<scratch_dir>/repairs/` only when it is empty; the stop report lists whichever of the two was used.
- A new sentence there: the driver may write one optional friction note of its own into `friction_dir`, only when something went wrong in driving, under the same rule as the other agents' friction directives.
- `## Self-report` is replaced by a short `## Filing` section: the driver drafts and files no issue in any mode, because the run's reflection reads `friction_dir`, repair records included, and files what it finds.
- `## Stopping on a non-running state` and `## Notifying the parent`: a `done` stop whose envelope reports `friction: failed` notifies the parent as an escalation does, naming the run-id and the stop report; every other `done` stop still sends nothing.
  The stop report of every stop records the envelope's `friction` value.
- The two loom row names the skill still carries go: the `## Baseline` example label becomes `before step: blocked at <current_producer>`, and the stranded-branch paragraph's "while its Webster phase is in flight" names the in-flight producer generically rather than by row.

What this removes, and the bound: the operator-approval step before a driver's `lyx selfreport create` goes away together with the driver's filing itself, so no driver-side filing path is left to bypass; the parent notification on `friction: failed` is the only new notification.

`driveskill_test.go` gains tests reading the skill file as `TestDriveSkill_NamesParkMarker` does, asserting the text no longer contains `lyx selfreport create`, does contain `friction: failed`, and names no loom row as a whole word (the row names are the keys of `loomshed.InterruptPolicies`, which maps every durable row).

**ImpactSummary:** the ly-drive skill text and one test file pinning it; no Go behaviour changes.
