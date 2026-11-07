```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the D3 diff forces outside its own lines.

**Per change, the set found and whether it is covered**

1. **`copyTemplate` checks only `requireUnpoisoned`.**
   - Callers: `CopyHub` and `CopyRegistered`.
     Their doc comments, the `template.go` header and `doc.go` are all updated to say a copy checks only the poisoned flag. Covered.
   - `requireIntact` is now reached only from `SharedHub` (`template.go:490`). Covered.
   - The test pinning "write poisons, later `CopyHub` fails" is `TestSharedHub` (`hub_test.go:651`). Covered.
   - The `SharedHub` doc comment (`template.go:480-485`) still says it fails when the tree "changed since it was built". That is correct, since `SharedHub` keeps the walk.
   - No other package or doc outside `internal/hubforge` mentions a per-copy fingerprint. My grep found none.

2. **`packRepositories` and `gitDirsUnder`, called from `buildTemplateIn` before the fingerprint.**
   - Behaviour readers: `relocateHub`, `findRootReference` and `fingerprint`.
     Pack files are binary (NUL bytes), so relocation skips rewriting them. `packed-refs` holds no path. Covered by `findRootReference` inside `relocateHub`, which fails the copy on any leftover template path.
   - Docs: `doc.go:33-36`, the `template.go` header and the `buildTemplateIn` comment all describe the packing. Covered.
   - Cross-package users of `CopyHub`, `SharedHub` and `CopyRegistered` call only the public API, which is unchanged. Not affected.
   - Git floor: `toolfloor.GitFloor` is 2.43.0 (`internal/toolfloor/toolfloor.go:14`), and `--cruft` needs 2.37. Not a defect.
   - Departure from the record's `git repack -ad`:
     - No `Pattern-Departure:` trailer is on either commit.
     - No `PATTERN` entry forbids `--cruft`. The trailer applies only to a difference from a `PATTERN` entry, so none is required. No finding.
     - The departure is stated in `_lym/landing/summary.md`, and `--cruft` serves the acceptance criterion "no loose objects" better than `-ad`.

3. **`commitRelocatedBinding` drops its `git add`.**
   - The only caller is `copyTemplate` (`template.go:461`).
   - Its readers are the tests, which now check the clean board, HEAD equal to upstream, the records bare tip and the newest tracking-ref reflog entry. `assertBindingFoldedAndPushed` covers them from `TestCopyHub` and `TestCopyRegistered_…`.
   - The doc comment is updated. The `hub_test.go` `TestCopyHub` comment is updated. `doc.go:50-51` is updated.
   - `internal/fabricengine/reflog.go` reads the tracking-ref reflog through `trackingReflog`. The push still writes it, and no other reader of the file shape was found.

4. **The `hub_test.go` assertions on the board changed.**
   - `TestCopyHub` now clean-checks only `PrimeRecords()` directly, and the board through the helper.
   - The old `rev-list @{upstream}..HEAD` check is replaced by the stricter SHA equality check. No gap.

5. **PATTERN documents.**
   - `PATTERN-test-isolation.md` hub-fixtures paragraph (lines 186-208) describes copies, `SharedHub` and registered templates. It says nothing about the fingerprint walk or the binding fold, so nothing is stale.
   - `PATTERN-test-speed.md`: each new test has `tiergate.Require(t, 1)` and `t.Parallel()`. Compliant.
   - `PATTERN-gitrepo-client-boundary.md`: the `git` calls here go through `gitkit.Git` in test-support code, so the boundary is not engaged.
   - Declared PATTERN changes: the record's Constraints list D1, D5 and D7 items, none for D3. The diff touches no `PATTERN.md` or `pattern/` file. Nothing to land.

6. **Comment conventions.**
   - Comments are self-contained and have no end-of-line comments.
   - Nothing in the changed text violates the Go comment skill. Line breaks are not a finding.

**Findings:** none.

Tool calls:
1. Ran a grep for `fingerprint`, `requireIntact` and `commitRelocatedBinding` outside `internal/hubforge`. My context lacked an outside-package search for readers and docs of the changed behaviours.
