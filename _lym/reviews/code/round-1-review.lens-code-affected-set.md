```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens review of the diff.

**Per change, the set found:**

- `landDeps.HeldSlugs` (new field):
  - `landDeps{` is constructed in two places only: `internal/beamcli/deps.go:147` (sets it) and `internal/beamcli/land_test.go:131` (sets it). Covered.
  - No other constructor can leave it nil.
  - The only caller is `mergeParent`, as the `landDeps` comment says. Covered.
- `freeMergeTaskSlug` signature (gained `held`):
  - The only caller is `mergeParent`, in `internal/beamcli/mergeparent.go`. Covered.
  - No other test calls it directly.
- `merge-parent` slug rule text:
  - `cli.go:277` and `doc.go:262-268` are updated and agree with the code.
  - `docs/overview.md:808` describes merge-parent without the slug rule, so nothing is stale. Covered.
  - Stencils: `grep` for `-merge-in` found no stencil stating the numbering rule.
- `shedrun.RunIDs` (new) replacing the inline reader in `fixloop.go`:
  - The only other caller is `hubFixDeps` (`fixloop.go:395`), and `fixloop_test.go:63` fakes the seam, so nothing else asserts the removed inline code. Covered.
  - Imports in `fixloop.go` are still used: `errors` (349), `os` and `filepath` (264). Covered.
  - Other readers of the same directory are `shedrun.List` (`seed.go:243`), which filters on `seed.json`, and `battencli.cleanupSlugs` (`cleanup.go:245`), which unions the records and scratch roots. Both have different semantics, so neither is a drop-in for `RunIDs`. No finding.
  - A `fixloop.go` alignment-only change (gofmt realignment of struct fields) is licensed by the refactor.
- `RunIDs` and `RunDir`/`runSegment`:
  - `RunIDs` lists `<anchor>/.../shed/`, the parent of what `RunDir` joins. `RunDir`'s doc says the legacy `self` directory serves through `runSegment`. A `self` directory therefore appears in the list as the name `self`.
  - A held slug of `self` is not a real beam slug, so there is no practical collision.
  - Not a finding.
- PATTERN entries:
  - `PATTERN-lym-files`: `shedrun` names the directory via `lymdirs.LymDirName` and `shedDirName`. Compliant.
  - `PATTERN-gitrepo-client-boundary`: `HeldSlugs` calls `fabricengine.TaskBranchSlugs`, the existing path. Compliant.
  - `PATTERN-owner-machine`, `PATTERN-landing` and `PATTERN-sole-parsers` are not touched.
  - No declared PATTERN changes appear in the focus or the rubric (the amendment is `none`).
- Commit messages: neither commit carries a `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

Tool calls:
1. `grep` for `landDeps{` and merge-in slug text across the repo: needed every constructor and doc naming the rule.
2. `grep` for `ReadDir` in the shed/batten/darn packages and the `RunRecordSlugs` context in `fixloop.go`: needed other run-directory readers and the removed code's seam.
3. `sed` of `seed.go` `List` and `cleanup.go` `cleanupSlugs`, plus `grep` of `docs` for merge-parent: needed whether those readers should share `RunIDs`, and whether docs state the slug rule.
