# Card 4 — The missing-seed and `run` status-file refusals name the driverless seed

**Edit:**
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#loomCLI.loomPreRun`
- `contracts/specs/refusal-spec.md`
- `internal/loomcli/arm_seed_test.go`
- `internal/loomcli/cli_test.go`

**Uses:**
- `internal/shedrun#MissingSeedMessage`
- `internal/shedrun#SelfRunID`

**Intent:**
Decision D4.
The missing-seed refusal and `run`'s status-file refusal name only `lyx loom start`, which spawns an LLM driver; a caller that wants a driverless run finds no route on the refusal path.

`resolveRunID` picks the remedy it passes to `shedrun.MissingSeedMessage` by verb and run-id.
For `run` and `step` addressing `self` (`shedrun.SelfRunID`), the remedy names both routes from D3: `run "lyx loom start" first to bootstrap this task`, or a driverless seed with `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>` (`<parent-branch>` being the pair's recorded parent branch, `parent_branch` in fabric's origin record) followed by `lyx loom step`.
For any other run-id, and for `status`, `pause` and `goto`, the remedy stays `run "lyx loom start" first to bootstrap this task`: a driverless seed gives those verbs nothing to act on, and the bootstrap's seed-agreement check in `seedAndCommitBootstrap` reads and writes only the `self` seed, so a parent typed into another run-id's seed would never be checked against the recorded parent.
The message still renders through `shedrun.MissingSeedMessage` and the refusal stays a `KindlessRefusal`.
The input that reaches each branch after every upstream check passes is a generic verb addressing a run-id with no `seed.json`: `run` or `step` on `self` for the two-route remedy, and `status`, `pause`, `goto`, or `run` or `step` on another run-id, for the `lyx loom start` remedy; `TestResolveRunID` reaches both.

`loomPreRun`'s status-file refusal, which a driverless seed reaches when `run` is called before any `step`, names `lyx loom step` beside `lyx loom start` as the way to seed the status file, for example `run "lyx loom start", or "lyx loom step" after a driverless seed, first to seed the status file`.
Its input is a seeded run-id with no status file; `TestVerbRefusals`'s `Run_SeedMissing` row reaches it.
Its doc comment's claim that only `lyx loom start` may seed is corrected: the shared bootstrap of `start` and `step` seeds the status file.

What the change lets through and its bound: it only adds a named route to two refusals' text; `run` still never seeds and `step` still refuses without a seed.
The parent branch stays a placeholder because the refusal path reads only the seed and the run list; a wrong value is refused at the next bootstrap's seed stage (card 5).
Both remedies name only `lyx loom start`, `lyx loom step` and `lyx shed seed`, none denied (`PATTERN-no-denied-recovery`).

In the same commit (`PATTERN-refusal-way-forward`), the loom section of `contracts/specs/refusal-spec.md` changes:
- its lead sentence, which says loom's own seed is written by `lyx loom start`, also names the driverless `lyx shed seed` route the `run` and `step` rows name;
- the `seed missing` row becomes two rows, one for `run` and `step` addressing `self` quoting the new two-route remedy, and one for `status`, `pause` and `goto`, and for `run` and `step` addressing another run-id, keeping `run "lyx loom start" first to bootstrap this task`;
- a new `run: status file missing` row, trigger `lyx loom run` finds a seed and no status file (as after a driverless seed before any `step`), class correctness halt, quoting the new remedy;
- the `status file missing` row for `status` and `approve` stays as it is.

Coverage extends existing tests, with no new top-level test:
- `TestResolveRunID` in `arm_seed_test.go` gains a `goto` refusal row, a `step` row addressing a non-`self` run-id, and a want-remedy column: the `run` and `step` rows on `self` assert the message holds `lyx loom start`, `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>` and `lyx loom step`; the `status`, `pause` and `goto` rows and the non-`self` `step` row assert it holds `lyx loom start` and not `lyx shed seed`.
- `TestVerbRefusals` in `cli_test.go` asserts the `Run_SeedMissing` row's output holds `lyx loom step` beside `lyx loom start`.
Grepping for the old remedy text finds the binary-driven assertions in `smoke_prebootstrap_integration_test.go`, which check only `loom start`, still held by both messages.

**ImpactSummary:** the text of two loom refusals and their spec rows; no verb's control flow changes.

**Commit:** 4: loom: missing-seed and run status-file refusals name the driverless seed
