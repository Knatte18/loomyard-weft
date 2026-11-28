```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, rerunning the set-premise findings' searches against the fix.

**Unchecked findings:** F4, F8, F10, F11, F12 and F13 have no search rerun. Each names a single site or a set already visible in context. F1, F2, F3, F5, F6, F7 and F9 were checked.

**Findings:**

1. **MEDIUM**: F7, batten half still unpinned (`internal/battencli/driverdeps_test.go`).
   - The new assertion passes the literal `"tst:orch"` into `tacklerDepsFor` and reads it back.
   - Nothing asserts that `loadedTacklerDeps` (`drive.go`, `return c.tacklerDepsFor(settings, shuttle, c.promptParentName())`) or `loadedArbiterDeps` (`ParentName: c.promptParentName()`) passes the name.
   - Replacing either with `""` keeps every test green. F7's premise named both batten sites.

2. **LOW**: F2, twin help text left stale (`internal/driftcli/cli.go`, `deferCmd` Long).
   - It still reads "so only the run's parent or the operator's own shell runs it".
   - `defer` shares `callerBy` (`cli.go:357`), so it now admits every orch above the pair. Only `amend`'s Long text was updated.

3. **LOW**: F3, same claim left in sibling comments after the stencil reword.
   - `internal/sheddriver/tacklerprompt.go:27` and `internal/shedarbiter/prompt.go:52` say "ParentName is the orch the run hands its stops back to".
   - `internal/battencli/drive.go:420` says "parentName is the orch the run hands its stops back to".
   - The fix withdrew that claim in the stencil, `parentdirective` and `stencils.go`, because a gone parent pair's orch receives no stop.

4. **LOW**: F1/F9, "nearest live orch" wording left where the routing changed.
   - `docs/overview.md:858` says the dobby "sends the nearest live orch its own notices". Those now go through `NoticeDestination` or `NotifyRunFrom`, which can pick a beam orch that reads as down.
   - The same wording remains in `internal/dobbyengine/doc.go:60,146,208,232` and `loop.go:156`.
   - It also remains in `internal/beamcli/cli.go:739` and the comments in `contracts/stencils/beam/beam-template-describe.md:6` and `beam-template-board-tidy.md:5,40`.

The F1 routing call sites and the F6 caller-check sites all appear covered.

**Tool calls:**
- grep of non-test routing call sites (F1): context lacked a post-edit list of all callers.
- grep of parent-caller checks outside the edited files (F2): context had no search of other verbs.
- grep of docs, contracts and help text for old-rule wording (F9/F1): context showed only a pre-edit, partial search.
- grep of prompt-input wirings and hand-back wording (F3/F5/F7): context lacked the sibling comments.
- read of `drift defer` help and the `loomcli/doc.go` caller rule, plus a grep for orch-role caller checks (F2/F6): context had not shown either text.
