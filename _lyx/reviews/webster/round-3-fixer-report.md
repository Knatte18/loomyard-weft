# Round 3 fixer-report — webster

## Fixed

### F1 — merged PATTERN.md over the overview size cap

Validated: `PATTERN.md` was 16576 bytes against `MaxOverviewBytes` (16384) at `internal/pattern/check.go:21`.
Main's side of merge `523ec97ad` added `PATTERN-hub-store-gc`, `PATTERN-gogit-read-helper` and `PATTERN-gate-slot`, and the branch adds the `## Daemons and waits` section and `fswatch` in `PATTERN-leaf-packages`.
Neither side was over alone; the merge result was.

Commit `62bc9881e` cuts 239 bytes, leaving the file at 16337 bytes.
Each cut drops an enumeration or a clause the entry's background file already states, so no entry loses its trigger or requirement:

- `PATTERN-push-both-sides`: drops the parenthetical list of the prime's push verbs; "the prime's pushes" covers them all.
- `PATTERN-parent-directive`: drops "not stencil text", which "come from the marker" already says, and shortens "outside the rule" to "exempt".
- `PATTERN-edit-directive`: shortens "outside the rule" to "exempt".
- `PATTERN-quarry-cgo`: replaces the rationale clause with "for quarry's tree-sitter grammars"; `pattern/PATTERN-quarry-cgo.md` states the cgo guard.
- `PATTERN-hub-store-gc`: drops "and it is no destructive primitive", which `pattern/PATTERN-hub-store-gc.md` states.
- `PATTERN-fabric-vocabulary`: drops the owner-set scope clause, which `pattern/PATTERN-fabric-vocabulary.md` states.
- `PATTERN-documentation-lifecycle`: shortens the link text to `overview`; the target is unchanged.

The branch's own entries stay as they were.
No test pins the wording of any edited entry.
No test was added: the finding was a failing existing test, not a coverage gap.

### Gate repair — comment lines broken mid-sentence

The card gate's `lyx loom lint-comments` flagged twelve Go comment lines, all brought in by `main` through merge `523ec97ad`, that end in a comma with the sentence continuing on the next line.
Commit `5a569cb3a` joins each such sentence onto one line, in `internal/boardengine/layer.go`, `internal/proc/procgroup.go`, `internal/seatengine/shuttle.go`, `internal/shedadapters/multillm.go`, `internal/shedrecipe/entries_multillm.go`, `internal/shuttleengine/doc.go`, `internal/stencil/stencil.go`, `tools/tokencount/cost.go` and `tools/tokencount/main.go`.
Comment text is unchanged.
`lyx loom lint-comments` reports ok on the working tree and with `--base` at `523ec97ad`, its first parent and `abd062f46`.

### Gate repair — verify flake in `internal/loomcli`

The verify run's integration tier failed `TestLoomResumeReconcilesTheHubConfigBeforeArming` on cleanup alone: `TempDir RemoveAll cleanup: unlinkat .../weft-bare/objects/tmp_objdir-incoming-...: directory not empty`.
Every assertion had passed, and the test passed eight of eight isolated reruns.
Cause: the final `loom resume` reconciles the pair's config and commits it through `Fabric.Commit`, which fires `fabricengine.SpawnDetachedPush`, a fire-and-forget `lyx fabric push` child.
The test returned while that child was still receiving into the fixture's bare weft remote, so `TempDir`'s removal raced it.

The test asserts the reconcile commit and a clean records tree, never the push.
Commit `126a1edfb` runs both `loom resume` calls with `FABRIC_SKIP_PUSH=1`, which `SpawnDetachedPush` honours by spawning nothing.
The variable is set on the subprocess alone through a new `runLoomCLIWithEnvNoFatal` in `smoke_helpers_test.go`, which `runLoomCLINoFatal` now delegates to with no extra env, so the test keeps `t.Parallel` and the 25 existing callers are unchanged.
No test was added: the finding was a race in an existing test, not a coverage gap.

## Verification

`lyx gate test ./internal/pattern ./cmd/lyx ./contracts/stencils` passes, including `TestOverview_RealFilePassesCheck`, and after the comment repair `lyx gate test` passes on every touched package: `internal/boardengine`, `internal/proc`, `internal/seatengine`, `internal/shedadapters`, `internal/shedrecipe`, `internal/shuttleengine`, `internal/stencil`, `tools/tokencount` and `internal/pattern`.
After the flake fix, `internal/loomcli` passes untagged and under `--tags integration`, `go vet` passes under `integration` and `tmux`, and `lyx loom lint-comments` reports ok.

## Disputed

None.

## Deferred

None.
