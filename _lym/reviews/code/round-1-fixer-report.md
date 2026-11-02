# Fixer-report, round 1

## Changes

- **F2** fixed as proposed: `internal/fabricengine/warpexclude_integration_test.go` gains the row `anchored_hub_wiring_seeds_the_lines_under_the_anchor`. On `Shape{Anchor: "backend"}` it wires a pair through `AddPair` and `WireJunctions`, then asserts each `/backend/<path>` line appears once and no root line appears, reusing `countLine`. Commit `d868c7a4`.
- **F3** fixed as proposed: `internal/heddlecli/cli.go` help and `internal/heddlecli/doc.go` now say "anchor directory". Commit `3fba620b`.
- **F4** fixed as proposed: `internal/heddlecli/cli_integration_test.go` gains `RenderWritesFileTargetsAtTheAnchor` on `Shape{Anchor: "backend"}`. It asserts both file targets exist at the anchor and not at the worktree root. Commit `cd8e7653`.
  - Its `show` assertion first checked only the stored text, which does not depend on the anchor, as two self-check lenses found.
  - It now seeds an `AGENTS.md` at the anchor and asserts `show codex-file` contains it. Follow-up commit on the same finding.
- Gate: `lym gate test ./internal/heddlecli --tier 1` passes. The new `fabricengine` row passes in a narrowed run (`-run TestSeedWarpExclude_SeedsWarpOnlyOnce`). The full `fabricengine` tier1 package run hit its 10-minute package timeout under load, outside any test.

## Self-check

- fix-code-correctness, LOW: the `show` assertion passes with the defect in place; fixed as proposed by the second change under F4.
- fix-proposal, LOW: same finding, merged with the above.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- **F1**, ground `premise false`: `MerriamBase` already reads the instruction files at the anchor in hub mode. `internal/hubgeom/webstergeom.go:19-31` sets `Geometry.WorktreeRoot` to `l.AnchorPath()` and `RepoRoot` to `l.WorktreePath()`, and `internal/websterengine/geometry.go:31-33` says so. A rendered `CLAUDE.local.md` at the anchor is therefore counted. The `merriambase.go:38` doc comment's "worktree root" names that field, so changing it to "anchor" would misname the field.

## Gate finding

The verify run failed on `internal/fabricengine` with `panic: test timed out after 10m0s`, Go's default package timeout, with no test failing.
Run with `-timeout 30m`, the whole `fabricengine` tier1 package passes: 1602 passed, 0 failed, 159 skipped, wall 739s.
The package takes longer than 10 minutes under the machine's load, so no code change in this worktree repairs it; this round's fixes add one parallel row of a few seconds to it.

## Deferred

- The `fabricengine` package timeout: the fix is a longer test timeout in the verify command or a split of the package's tests, which lies in the hub's gate config in the Bolt checkout, outside this worktree. For `ly:orch` to settle.
