# Drift diffs captures in code against a sealed expectation and a judge

Drift no longer depends on an agent reading an opaque compare.
lym diffs the baseline capture against the current capture into changed values, checks them against an expectation an agent wrote and sealed before the writer started, and has a separate judge rule on each expected value against its exact old and new value.
An acceptance covers one exact value, never a whole test.
Drift stays a general module: no hub's harness or output format is assumed.

## What changes

- **Diff in Go** (`internal/driftengine`): every regular file under the capture directory is a case.
  Lines are identified by a key from `drift.key_patterns` (named groups `key` and `value`, with an occurrence `n` for a repeated key), or by position among the file's unkeyed lines.
  Cells are `added case`, `removed case`, `changed`, `added line` and `removed line`.
- **Two modes**, `drift.diff`: `lines` (default, the Go diff) and `command` (today's opaque compare with the judge ruling per drifted case, for a hub whose capture cannot meet the contract).
- **Scope by config**: `Drift-Baseline` records `off` or `full` from `drift.scope`, plus `diff` and `key_patterns`, in `scope.json`; no agent chooses it.
  `partial`, `skip` and `{skip_file}` are gone; a stored `partial` loads as `full`.
  The `docs` profile's `darn` block sets `drift.scope: off`.
- **New rows**: `Drift-Expect` (after `Drift-Await`) spawns the expect agent, validates `_lym/drift/expected.yaml` through its sole parser and seals it in one weft commit with a `Drift-Seal` trailer.
  `Drift-Verify` and `Drift-Recheck` now share one engine, capture into a scratch directory, check containment by code, and spawn the judge only for cells with no stored ruling.
  Both the tapestry and darn recipes are updated; `Drift-Recheck` loses its `on_stuck` route.
- **Containment**: an unexpected cell stops the row, naming case, identity, old and new value, `lym drift amend` and the route that fixes code.
- **Writer gate entry and verb**: a `drift` gate entry on `Darn-Write` and the Webster row, and a new `lym drift check`, run the same check mid-work and write no tracked record.
- **Judge**: rulings are stored as exact tuples in `_lym/drift/judged.json`; a reproduced tuple reuses its ruling, a changed one is judged again.
  Open ranges such as `12-` are allowed and each is listed in the judge's output and the verdict.
- **Parent verbs**: `lym drift amend` adds expectation entries, or `--restore`s or `--discard`s after a failed seal or records check; `lym drift defer` is widened to the cases where a baseline is missing or changed.
  `lym drift waive` is removed.
- **Verdict**: `driftengine.Verdict` renders the `## Regression check` section from the committed records without capturing: scope and its source, diff mode, cell counts, uncovered entries, open ranges, amendments and deferral.
  Old compare, acceptance and waiver fields render under a line saying the check ran under the old Drift.
- **Config** (`drift.yaml`): `diff` and `key_patterns` added; `baseline_model`, `verify_model` and `verify_interactive` migrate to `expect_model`, `judge_model` and `judge_interactive`; `skip` and `verify_max_runs` are retired keys.
  `diff` and `key_patterns` are refused in `run_settings`.
  A `configreg.Reinterpretation` flags a base with `compare` set and `diff` unset, a command holding `{skip_file}`, and a base with scope `partial`.
- **Stencils**: new `drift-template-expect` and `drift-template-judge`; the darn writer's, Merriam's and the card fork's stencils gain a drift paragraph.
- **Removed**: the `scope`, `accept`, `withdraw`, `waive` and `compare` verbs, `capture --side`, acceptances, the accepted profile and its seams, the run budget and `runs.json`, the baseline and verify agents with their stencils and `baseline.md`/`verify-report.md`, the `DriftRecheck` engine, and the `drift-verify` gate key.

## Design decisions to check

- **Tamper guards through git.**
  The seal and the drift records (`scope.json`, `judged.json`) are guarded by weft commits carrying `Drift-Seal` and `Drift-Records` trailers; the baseline directory's digest is recorded as a `Drift-Baseline` trailer.
  `internal/fabricengine` gains drift-agnostic seams for this: commit paths with a trailer, find the oldest or newest commit carrying a trailer key, read a file at a commit.
  A forged commit carrying the trailer is not detected.
- **Judge guard.**
  Drift-Verify snapshots both checkouts and the scratch capture directory before the judge runs and stops on any change other than `judgement.yaml`; the snapshot lives under the hub's board-level `.lym`, named in `internal/lymdirs`.
- **Old chairs ended through the strand table.**
  Live chairs of the removed `drift-baseline` and `drift-verify` roles are ended through a `Deps.EndRoles` seam over reed's strand table, not `runproc.EndRole`, since no seat chair is registered there.
- **Captures take their gate slot before the capture lock** and wait for the lock instead of refusing, so concurrent checks serialise without deadlock.
- **`judgement.yaml` is removed** once Drift-Verify stores the rulings, on a Done and on a does-not-match Stuck alike.
- **`PATTERN-sole-parsers`** gets the short declared text; the list of formats, now with the drift expectation and judgement files, lives in `pattern/PATTERN-sole-parsers.md`.
  The line assumes `raddle-recipe`'s change lands first; check the merged list.
- **Fabric vocabulary.**
  Side-named identifiers were renamed (`TrailerCommit.CodeHead`, `RecordsChangedSince`, `LoomRecordsDirs`, `RecordsPathsChangedSince`) to keep `PATTERN-fabric-vocabulary`.

## To check by hand

- Finding `drift-gate-passes-on-fault`, in the drift gate entry (`internal/shedrecipe/entries_drift.go`, `internal/driftengine`): a review left it open; confirm the entry reports a fault rather than passing silently.
- Finding `card-22-deletes-live-compare-lock-retry`, in the capture lock's bounded retry (`internal/driftengine/capture.go`): confirm the removal of the old compare path did not delete a retry still in use, and that a check waits for the lock.
- The verify gate's first attempt failed on `cmd/lym` and two `hubreconcile` reinterpretation-marker subtests; a fix commit (`34fa6087`) changed the fixtures to set `diff` and corrected the `amend` and `defer` Use lines.
  Confirm the three identities pass.
- A hub's `capture` command holding `--skip-file {skip_file}`, such as the `cs` hub's, is refused until the operator edits it; confirm the reinterpretation notice names the edit.
- Runs in flight when this lands: past `Drift-Expect` with no seal they are checked against an empty expectation and stop on any drift (`lym drift amend` clears it); a `partial` scope is checked as `full` with its skipped cases left out.
- Parent work merged before a compare (tapestry's Parent-Align before Drift-Verify; darn's Finalize after Drift-Recheck) can move outputs relative to the baseline.
  Re-capturing the baseline at the merged parent is a follow-up.
- `sampler-drift-trial` assumes the verbs removed here and is revisited after landing.
