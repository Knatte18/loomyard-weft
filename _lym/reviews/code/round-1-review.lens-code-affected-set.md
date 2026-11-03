```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the `WorktreeRoot`/`PaneCwd`/`RepoRoot` change forces outside the touched lines.

**Per change, the set I found and whether it is covered:**

1. **`Geometry.RepoRoot` removed, `PaneCwd` added, `WorktreeRoot` redefined.**
   - Production readers of `Geometry.RepoRoot` were `merriambase.go`, `recoverbatch.go`, `runlevel.go` and `verifyfixer.go`. All four now read `WorktreeRoot`; none is left. Covered.
   - Every other `.RepoRoot` hit in the repo belongs to burler's geometry or to `gatefixer.Input` (`internal/gatefixer/prompt.go:66`, `internal/hubgeom/hubgeom.go:109`, `internal/standalonegeom/burlergeom.go:23`). They are independent structs and are correctly untouched. Covered.
   - Construction sites of `websterengine.Geometry` outside the diff: `internal/loomcli/landingdeps.go:64` only takes it as a parameter and reads `geom.StencilsDir`. `internal/testkit/shedfake/shedfake.go:446` builds it without either root. Neither is affected. Covered.
   - Both constructors, `hubgeom` and `standalonegeom`, are updated and agree: in standalone, `WorktreeRoot` and `PaneCwd` are both `target`, as `RepoRoot` and `WorktreeRoot` were before. Covered.

2. **`trackedRel`, `suspectBlobs`, `writtenWorktreePaths` and `failBatchInput` take both roots.**
   - Callers: `findingtext.go:117`, `reset.go:436`, `batchfail.go:93`, `recoverbatch.go:212`, plus the two `failBatchInput` builders in `recordbatch.go` (lines 604 and 788) and the two in `recoverbatch.go` (lines 694 and 759). All pass both roots. Covered.
   - Every constructor of `failBatchInput` sets `PaneCwd`. Covered.

3. **Moves of `WorktreeRoot` to `PaneCwd`** (audit and write-path resolution, instruction files, `_lym`/`.lym` joins, gate cwds, prompt rendering).
   - The grep shows no remaining use of `WorktreeRoot` for a pane-relative join. What is left on `WorktreeRoot` is git calls, `index.*` card paths, `DiskSizes`, `runOwnPath` (git-status paths), `readModulePath` and `pattern.Directive`.
   - `internal/webstercli/validate.go:108-124`, `cli.go:292` and `rebaseline.go:110` read `WorktreeRoot` for plan paths. This is the #31 fix and is correct. Covered.
   - Stale-comment grep: the comments that still describe the old meaning (`beginbatch.go:125`, `recordbatch.go:222`, `render.go:448` and the `render.go:461` parameter note, `merriambase.go:38`, `geometry.go`) were all updated or are still accurate. No stale claim that `WorktreeRoot` is the anchor remains in the webster packages. Covered.
   - The `hubgeom_test.go:251` mention of `RepoRoot` is about burler, not webster. Not affected.

4. **Rule documents.**
   - `PATTERN-cwd-resolution` and its background file: the diff joins no anchor by hand, and both roots come from `l.WorktreePath()` and `l.AnchorPath()`. No violation.
   - The record and directive declare no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing to land.
   - The commit message carries no `Pattern-Departure:` trailer and the diff needs none.

5. **Stencils.**
   - `contracts/stencils/webster/webster-body-implementer.md:26`, `webster-body-verify-fix.md:20` and `webster-template-plan-fix.md:18` say the session's cwd "is the worktree root, `{{.worktree_root}}`". The diff leaves that placeholder filled from `PaneCwd`, which is the anchor. In a subpath hub the prose "worktree root" therefore names the anchor directory.
   - This is the same fill as before the change, and the record does not ask for stencil edits. See finding 1.

**Findings**

1. **LOW, class `consistency`.** Stencil prose describes `{{.worktree_root}}` as the worktree root, but the diff keeps filling it from `PaneCwd` (the anchor). It was already so, and in a subpath hub the two now diverge in name. The diff did not rename the placeholder or reword the stencils.
   - Evidence: `contracts/stencils/webster/webster-body-implementer.md:26`, `webster-body-verify-fix.md:20`, `webster-template-plan-fix.md:18`; fill at `internal/websterengine/render.go:212` and `render.go:304`.
   - The fix is a wording change and may not be worth making, since the record's `## Both` asks for no stencil change.

I found no other member of the affected set left wrong.

**Tool calls:** 1.
- A repo-wide grep of non-test `.go` and `.md` files for stale root descriptions, plus `worktree_root` in the stencils and `docs/`. My context lacked the stencil text and the non-webster `RepoRoot` hits.
