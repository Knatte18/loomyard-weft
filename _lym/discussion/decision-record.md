## Goal

Code review reads the gate's redundancy report, and PATTERN-test-speed names the map, verb and step

Takes: gate-redundancy-report's report reader and HEAD tree match
Delivers: a Go-filled rubric marker in loom-rubric-code-review, filled through shedadapters.ReadRubric for the Code-Burler-Review and Code-Bouncer call sites with the findings or the no-current-report line; the rubric's test-economy text on contained tests; PATTERN-test-speed's index line and its background's gate and test-economy sections
Exports: nothing new

## Scope

The part changes these packages:

- internal/shedadapters (about 60 changed lines)
- internal/shedrecipe (about 60 changed lines)
- internal/loomcli (about 40 changed lines)
- contracts/stencils/loom (about 15 changed lines)
- pattern (about 30 changed lines)

## Decisions

### D8 — Code-Burler-Review reads the report when it matches the reviewed tree

The Code-Burler-Review row's stencil gets a Go-filled marker.
Go fills it with the redundancy report's findings when the reviewed HEAD's tree matches the report:
every judged package's Go files hash the same, and step 1 of D7 run on HEAD gives the recorded tests with the recorded function hashes.
A report written on a red tier records no hashes and never matches.
Otherwise it fills a line saying no current report exists and why, naming the red gate when the report is a red one.
`contracts/stencils/loom/loom-rubric-code-review.md` treats each contained test as a test-economy candidate.
The reviewer checks the test's assertions against the covering tests.
It then files a finding to fold or delete the test, naming the covering tests, or accepts it when it pins what coverage cannot see.
A test it accepts gets a `//testtiming:keep <reason>` finding.

Rationale: block coverage does not see assertions (archive tags, deletes on origin, user content in the study), so the reviewer decides.
A content match keeps the report valid across commits of run records and of code outside the judged packages.
Bound: the match ignores the map, so a map rebuilt since the report can make a contained verdict stale; review reading each finding bounds it.

### D9 — `PATTERN-test-speed` names the map, the verb and the step

The background's "Tests run through the gate" section adds `lym gate testmap` and the gate's `redundancy` step as runs that use a tier's `env` and a gate slot.
Its "Test economy" section states the package-scoped verdict with the module-wide column (D2), the map (D4), the report step and its bounds (D7), and that a gate finding is evidence for Code-Review exactly as a report candidate is.

## Constraints

- Per-test coverage needs each test in its own process run: `runtime/coverage` counters are process-wide, so tests under `t.Parallel` mix their counts.
  The map cannot come from a normal tier run.
- No test of this feature launches a test binary or `go test`: the runner's tests use a stand-in `go` and fake profiles, as `cmd/testtiming`'s prebuild test does.
- `PATTERN-gate-slot`: the verb and the step run tests only while holding gate slots.
- `PATTERN-fabric-boundary`: the verb's tracked-file and status reads (D5) and the step's merge-base reads (D7 step 1) go through `internal/fabricengine`.
- `PATTERN-config-layers`: every new key has a default and loads through `configengine`; no meaning of an existing key changes, so no reinterpretation ships.
- `PATTERN-cli-cobra`: `lym gate testmap` joins the existing `lym gate` cobra subtree.
- `PATTERN-refusal-way-forward`: the verb's refusal outside the prime worktree and the unknown report name refusal each name their way forward, and a test reaches each.
- `PATTERN-code-conventions`: the step's keys come from `gate.yaml`, never a hard-coded key list.
- The prime parent's worktree is updated in place when a run lands, so its HEAD tracks the prime branch.
  `internal/fabricengine/merge.go`'s merge runs `MergeStart` in the parent's warp checkout under the weft write lock.

## Auto-mode assumptions

- The full design is built, not only the tool change: the goal asks that redundant tests stop piling up, which needs the check on new tests.

## Open risks

## Acceptance criteria

- D8: the marker holds the findings when HEAD's judged packages and changed tests match the report,
  including when the diff also changes a `tier2` test and after a commit touching only `_lym/`.
  It holds the no-report line otherwise, naming the red gate for a report written on a red tier.
- Review: D8's rubric text tells the reviewer to check assertions before filing a fold, delete or keep finding.
- Review: D9's background text states the package-scoped verdict, the map, the verb, the step and their bounds.

## PATTERN changes

- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by gate key; one `tiergate.Require` tier, lowest that fits; it pins behavior no other test covers, which the gate's redundancy report checks; skips, clocks, `t.Parallel`, `//lym:seam` and `//lym:watch` per the background.
