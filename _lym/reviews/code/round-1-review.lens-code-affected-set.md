```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, listing what each changed contract forces outside the diff.

**Per-change list**

1. **Rendered-file location moves from root to anchor** (`RenderRequest.Root` to `Anchor`, `Report` paths, `Show`'s argument, `WorktreeEvent.Anchor`).
   - Covered: callers in `heddlecli/verbs.go` (Show and Render), `seam.go`, and the tests in `heddleengine` and `fabricengine`. No `Root:` use of `RenderRequest` is left.
   - Covered: `PATTERN.md` and both `pattern/` backgrounds, in the diff.
   - Not covered: `internal/heddlecli/cli.go:52` and `internal/heddlecli/doc.go:28` still say a render writes at "a worktree's root". See F1.
   - Not covered: the Merriam start base in `internal/websterengine/merriambase.go:38,43` still reads `CLAUDE.md` and `CLAUDE.local.md` from `geom.WorktreeRoot`. See F2.
2. **`warp_exclude` paths become anchor-relative** (`seedWarpExcludeLines`, `SeedWarpExclude`, `junction.go` call, `template.yaml` comment, `doc.go`, `config.go`). All covered, including the new `anchored_hub_seeds_the_lines_under_the_anchor` test. No other caller of `seedWarpExcludeLines` exists (grep).
3. **`ReRender` paths and the take-in selection** (`target.go`, `seam.go`). Covered by `AnchoredAgentsMd`, `AnchoredClaudeMd` and `AgentsMdAboveTheAnchor` in `seam_test.go`.
4. **Commit message and `Pattern-Departure:` trailers**
   - The message claims fabric's event carries the anchor, the render reads the anchor's CLAUDE and AGENTS files, and the take-in and exclude lines follow. The diff bears that out.
   - There is no trailer and none is needed. The `PATTERN-durable-vs-ephemeral-state` and `PATTERN-heddle-instructions` text changes landed.
   - The directive declares no PATTERN changes, and `PATTERN-cwd-resolution` is untouched.
5. **Tests that assert the old root behaviour**. `heddlecli/cli_integration_test.go` (lines 551–633) and `warpexclude_integration_test.go:18` use `hubforge` with Anchor `"."`, where root equals anchor, so they stay valid. Not a finding.

**Findings**

- **F1: LOW, consistency.** `internal/heddlecli/cli.go:52` says heddle renders file targets "at a worktree's root", which is the `lym heddle` help text.
  - `internal/heddlecli/doc.go:28` says "later renders write into a worktree's root".
  - Both are left stale by the anchor move.
  - Fix: say "anchor directory" in both. The `renderCmd` help text was already updated in `verbs.go`.
- **F2: MEDIUM, scope.** `MerriamBase` (`internal/websterengine/merriambase.go:43`) joins each instruction file name from `claudeengine.InstructionFiles()` (`CLAUDE.md`, `CLAUDE.local.md`) onto `geom.WorktreeRoot`.
  - In an anchored hub the rendered `CLAUDE.local.md` now sits at the anchor, so the start base no longer counts it.
  - The repository's own `CLAUDE.md` also sits at the anchor and was already missed. The task adds the rendered file to that gap and leaves the new location unread.
  - Fix: read from the worktree's anchor, if `Geometry` carries one. Otherwise the writer should confirm with the parent that this is out of scope.
  - Evidence: `merriambase.go:38,43` and `capabilities.go:33`. I did not read the `Geometry` definition, so whether an anchor path is available there is unverified.

**Tool calls**
1. `grep` for `InstructionFiles()` callers, plus `sed` of `heddlecli/cli.go` lines 45–58 and `doc.go` lines 24–30. My context lacked the instruction-file consumers and the heddlecli wording.
2. `sed` of `webstercli/cli.go` lines 262–278, plus `grep` in `auditengine.go` and `merriambase.go`. I needed to see how Webster locates instruction files.
3. `grep` of `merriambase.go` for `Root`, `Join` and `instruction`. I needed the base directory used for the read.
