# Card 10 — spec-requires

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Runner.attach`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shuttleengine#ErrRequirementsUnmet`
- `internal/shuttleengine#knownRequirement`
- `internal/shuttleengine#requiredOf`
- `internal/shuttleengine#unmetRequirements`
- `internal/shuttleengine/requires_test.go`

**Intent:** Add `Spec.Requires`, the closed list of what a run requires of its engine, checked at `Start` and at attach (D8).
It only refuses runs and never lets one skip a check: an empty list and no `ForkSubagents` behave as today, so no current role changes; this task sets no role's `Requires` (D8).

`Spec` gains `Requires []Requirement`, its doc comment naming the closed set, the `ForkSubagents` implication, that duplicates are harmless, and that `validate` does not inspect it.
In a new file `internal/shuttleengine/requires.go`:
- `knownRequirement` reports membership in the three `Requirement` constants.
- `requiredOf` returns the listed requirements plus `RequirementForks` when `forks` is true, as a deduplicated set in first-seen order.
- `unmetRequirements` returns nil when `engine` meets every requirement through `RequirementSupporter`, an engine without the capability meeting none.
  Otherwise it returns an error wrapping `ErrRequirementsUnmet` that names the engine, each unmet requirement, and the registered engines (`engines.Names()`, each resolved) that meet them all, `none` when no engine does.

At `Start`, `start` runs the check right after resolving the spec's engine (card 8) and before the skills check, the resume check, the pause wait, the run directory and any pane:
an unknown listed requirement is refused as `shuttle: start: unknown requirement "<req>"; known requirements: forks, peer-messaging, background-wake`, wrapping `ErrRequirementsUnmet`;
otherwise `unmetRequirements(engines, name, engine, requiredOf(spec.Requires, spec.ForkSubagents))` refuses with the way forward `way forward: give the role a model alias of an engine that meets them`.

At attach, `attach` checks the one attachable candidate before reconstructing it, against the engine its record names (D4): the attaching spec's own `Requires` joined with `forks` when the record carries `ForkSubagents`; a `ForkSubagents` on the attaching spec alone adds nothing.
An unknown listed requirement is refused as at `Start`.
An unmet set is refused with `unmetRequirements`' error and the attach way forward, because the live run keeps its recorded engine: `way forward: end the run's live strand with "lyx reed remove <strand guid>", then re-step the row once the run record in <run dir> is older than twice startup_timeout_s (<minAge>)`, the guid, directory and `minAge` the probe already holds.
A finished run, every declared output file on disk, is harvested without the check, since its harvest drives no further turn; that covers the attachable candidate whose files all exist and the three reed-gate harvests, which never reach the check.
The attach way forward is reachable, since `lyx reed remove` leaves the record reading running and the probe then answers not found once the record passes the age `attach.go`'s "younger than" refusal names.

`internal/shuttleengine/doc.go` gains a paragraph on `Spec.Requires`: checked at `Start` and attach, with the two ways forward.

These refusals are shuttle errors with no loom or webster refusal row in this task: neither is reachable from any on-disk state while Claude is the one registered engine and declares all three requirements, and no role sets `Requires` beyond the `ForkSubagents` implication (D8, R5).

`requires_test.go` adds `TestRunner_Requires`, table-driven over a set of `fakeEngine`s wrapped in a test type adding `Supports` over a scripted requirement set, and one plain `fakeEngine` with no `Supports`:
- `Start` refuses an unknown requirement, an unmet one (naming the engine, the requirement and the meeting engine), one that no registered engine meets (naming `none`), and a `ForkSubagents` spec on the engine without `forks`, each wrapping `ErrRequirementsUnmet` with its way forward and with no `AddStrand` call;
- `Start` on an engine meeting every listed requirement starts, with `Requires` holding a duplicate and `forks` listed beside `ForkSubagents`;
- attach of a running record over a live strand, with a spec whose `Requires` the recorded engine does not meet, is refused with the attach way forward naming the strand guid, the run directory and twice `startup_timeout_s`;
- attach with a spec listing an unknown requirement is refused as at `Start`;
- attach of a record carrying `ForkSubagents` by a spec without it, on a recorded engine without `forks`, is refused the same way;
- attach of a record without `ForkSubagents` by a spec carrying it, on a recorded engine without `forks`, attaches with no refusal;
- the same record with every output file present is harvested and waited on with no refusal.
Raise `internal/shuttleengine` in `test-budget.yaml` from 176 to 177.

**ImpactSummary:** Internal to shuttleengine: one new Spec field and two new refusals in start and attach, both unreachable for any spec or record a current caller produces.
