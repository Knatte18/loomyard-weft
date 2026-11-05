```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens for round 1, listing what each change reaches outside the lines it touches.

**Per change: set found, covered or not**

1. **Scratch link and exclude line move to `<anchor>/.scratch`** (`scratchlink.go`)
   - Set found:
     - Callers `junction.go:107`, `reconcile.go:472` and `anchorchange.go:472-475`.
     - Tests `addparent_integration_test.go`, `anchorchange_integration_test.go` and `remove_junctions_integration_test.go`.
     - Docs `docs/overview.md`, `fabricengine/doc.go` and the comments in `junction.go`, `unwire.go`, `reconcile.go`, `remove.go` and `gitexclude.go`.
   - Covered, apart from findings A and B below.

2. **`withoutRootScratchLink(l, names)` becomes `withoutScratchLink(names)`**
   - Set found: `reconcile.go:859`, `remove.go:307`, `unwire.go:74`. A grep for the old name finds no remaining reference.
   - Covered.

3. **Cleanup segment reads `location.WorktreePath()/AnchorRel/.scratch`** (`segments.go:120`)
   - Set found:
     - `cleanupcli/doc.go` and `scratch.go` comments, updated.
     - `docs/overview.md`, updated.
     - `cleanup_integration_test.go:107`, which still builds `filepath.Join(h.PrimeWorktree(), scratchDirName)` and uses `Shape{Anchor: "."}` (line 59).
   - Covered only at anchor `.`. See finding C.

4. **`Remove` unlinks the anchor link and a legacy root link** (`unlinkScratchLink`)
   - Set found: `fabricengine/doc.go` and `TestRemove_SweepsAnchoredLinksOnSubpathHub`.
   - The `doc.go` remove paragraph is left stale. See finding A.

5. **Anchor change drops the old anchor's exclude line**
   - Set found: `doc.go` lines 566 and 1334, and `requireWiredOnlyAt` in `anchorchange_integration_test.go`.
   - Covered by the diff's doc edits. Whether a test pins the dropped line is for code-tests.

6. **PATTERN declarations**
   - `git diff --stat` over `pattern`, `PATTERN.md` and `contracts` shows no change.
   - The directive declares no PATTERN entries. Nothing is required.
   - `PATTERN-fabric-boundary` and `PATTERN-lym-files` hold no scratch-specific text, so no PATTERN doc goes stale.

7. **Commit messages**
   - There is no `Pattern-Departure:` trailer in any of the four commits, and none is needed.

**Findings**

A. **MEDIUM, consistency.** `internal/fabricengine/doc.go:575-577` is left wrong. The paragraph says `lym fabric remove` "unlinks the root link … at a root and at a subpath anchor alike, and records one `link_removed` at the root" and "any other entry at the root is left to the worktree removal".
   - The diff makes the primary link the anchor link, so `link_removed` is recorded at `<anchor>/.scratch`.
   - A legacy root link is only a second, subpath-only unlink.
   - The paragraph contradicts the new line 568 ("`lym fabric remove` unlinks the link, and a retired root link still standing") added in the same diff.
   - The test `TestRemove_SweepsAnchoredLinksOnSubpathHub` asserts one `link_removed` at the anchor.
   - Evidence: `doc.go:575-577` and `remove_junctions_integration_test.go` (the diff hunk).

B. **LOW, consistency.** `contracts/specs/loom-plan-spec.md:128` still says "the repository root's git-ignored `.scratch/`".
   - In an anchored hub nothing is now wired or excluded at the root.
   - The path a `Create` target may use as a transient draft is then `<anchor>/.scratch/`.
   - The diff does not touch this spec.
   - Evidence: `contracts/specs/loom-plan-spec.md:128`, with `docs/overview.md:184` stating that nothing is wired or excluded at the root.

C. **MEDIUM, design.** Cleanup's anchored branch has no coverage.
   - `cleanup_integration_test.go:59` uses anchor `.`, and line 107 builds the path without `AnchorRel`.
   - The `segments.go:120` change only matters at a subpath anchor, and no test reaches it there. A revert to `location.WorktreePath()` would still pass.
   - The focus directive asks about this claim. The code does resolve the anchored path now, so the summary's claim holds. The gap is only the missing coverage.
   - Evidence: `cleanup_integration_test.go:59,107`; `segments.go:120`.

D. **LOW, consistency.** The `cleanupcli/doc.go` scratch-segment text keeps "Only direct children of the prime's `.scratch/` are touched" (context line after the hunk), while the lines above it now say the directory is at the prime's anchor. Minor wording drift.
   - Evidence: `cleanupcli/doc.go` diff context.

**Tool calls (3 used)**
1. Grep of `.scratch` in non-test `.go`, `.md`, `.yaml` and `.tmpl` files. My context held only the diff and the two fabric files, so this was to find other consumers.
2. `sed` and grep of `cleanup_integration_test.go`, to confirm the anchor and path the cleanup integration test uses.
3. Grep for "root scratch" and "scratch link" prose, to find stale text elsewhere.
4. `sed` of `doc.go` lines 566-580, plus a `git diff --stat` over `pattern`, `PATTERN.md` and `contracts`. The remove paragraph was shown only as unchanged diff context, and I needed to confirm the PATTERN and contracts diff was empty.
