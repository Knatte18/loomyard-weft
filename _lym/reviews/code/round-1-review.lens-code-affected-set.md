```yaml
lens: code-affected-set
outcome: reported
```

**Per change: the affected set, covered or not**

**Change 1 — `BriefTask.ParentBeam` now holds `root` for the prime (`internal/boardengine/store.go:768-780`); reaches `list` JSON and also `find` JSON (`Store.Find` calls `ListTasksBrief`).**
- Production Go readers of the field: none (grep for `.ParentBeam`/`parent_beam` outside boardengine/boardcli finds only callers of the `boardengine.ParentBeam` *function*, which is unchanged).
- Tests: `parentbeam_test.go:92,119`, `store_test.go:1039` updated; failing-seam row `parentbeam_test.go:150` still expects empty, consistent with the `parentErr` branch. Covered.
- Docs: field comment `store.go:58-59`, `boardengine/doc.go:170`, `boardcli/doc.go:48`, list help `cli.go:935`. Covered.
- `find` help/doc do not mention `parent_beam`; nothing stale.

**Change 2 — `get` envelope prints `root` for the prime (`internal/boardcli/cli.go:1374-1381`).**
- Caller: `cli.go:907-911` (single). Nil-task path `cli.go:913` still omits the key.
- Tests: `verbs_test.go:600,617`. Covered.
- Docs: `boardcli/doc.go:45-46`, top-level help `cli.go:99-100`, SKILL.md line 27. Covered.
- `stepProjects` doc comment `verbs_test.go:582` still says parent_beam "is empty under a grouping project" — now false; the body two lines later asserts `RootSubject`. **Not covered** (finding 1).

**Change 3 — `list --text` shows a P0 row (`text.go` `withRootProjectRow`, `cli.go:961-964`).**
- Tests: `stepListAndFindText` (`verbs_test.go:347-350`) updated. `stepListShowsASketch` (`verbs_test.go:385-391`) checks only the last two lines; sketches still follow P0, so it holds. `verbs_test.go:491` checks a prefix only; holds. `help_test.go:121,127` check word presence only; hold.
- Docs: `boardcli/doc.go:67-69`, `cli.go:933-934`, SKILL.md line 28. Covered.
- `find --text` goes through `writeListing` (`cli.go:1198-1205`) and gets no root row; consistent with the docs.

**Change 4 — new export `boardengine.RootProjectBrief` (`render.go:185-190`).**
- Caller: `boardcli/text.go:98` (production), so PATTERN-test-only-code holds.
- Documented in `boardengine/doc.go:338`. Covered.
- Board entry's "Exports: nothing new" names no exported symbol to judge against; left to scope lens.

**Change 5 — prose readers of `parent_beam`.**
- `orch-role.md:14,327`, SKILL.md 27-28 updated.
- Repo search for leftover wording ("those with an empty `parent_beam`", "when `parent_beam` is empty"): only `docs/research/loom/adopt-build-revision-2026-11-16/report.md`, a dated research record, not a reader — no finding.
- No stencil or spec mentions `parent_beam`; no "Stencil wording" exposure.

**Repo rules / commit checks.**
- Declared PATTERN changes: the directive declares none; the diff touches neither `PATTERN.md` nor `pattern/`. No finding.
- Commit messages: no `Pattern-Departure:` trailers in range; `ebb1e87c` message matches the diff.
- PATTERN-code-conventions: `root` referenced via `boardengine.RootSubject` everywhere in production code, no repeated literal.

Findings:

1. **LOW, consistency** — location `internal/boardcli/verbs_test.go:582` — `stepProjects`' doc comment still says parent_beam "is empty under a grouping project", but the assertions it describes now expect `boardengine.RootSubject`. Evidence: comment at `verbs_test.go:582` vs. assertions at `verbs_test.go:600-605`.

Tool calls:
1. Grep for leftover "empty … prime"/`parent_beam` wording across Go and markdown — to find prose readers outside the manifest.
2. Grep for Go code reading `ParentBeam` or shelling `board list` in orch/dobby/fronttask/batten packages — to rule out hidden consumers of the printed value.
3. Grep for copies of the old orch-role wording, plus range check for `Pattern-Departure` trailers and `PATTERN.md`/`pattern/` changes — manifest did not cover them.
