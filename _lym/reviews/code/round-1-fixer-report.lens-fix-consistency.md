```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: checking that the change set agrees with its surroundings.

1. **LOW** · `_lym/landing/summary.md`, `## Tests`: the `TestTargetOf_AWorktreeNoLongerStandingIsAPairThatStartsNoPusher` bullet is stale.
   - It still says the test "drives the real reads over both removals".
   - The fix added a third `removals` row, `ThePathIsARegularFile` (`internal/runpusher/seams_test.go`), which this test also ranges over. The fixer rewrote the neighbouring `TestWorktreeGone` bullet to "each removal shape" but left this one.

2. **LOW** · `internal/runpusher/run.go:422`: the twin of the F4 doc was left as it was.
   - `canPush` still reads "reports whether the worktree exists and the start conditions but the lock hold".
   - It gates on `deps.WorktreeGone()`, whose field doc now says "no longer stands, holding no `.git` entry". A directory that exists without `.git` makes `canPush` answer false.

3. **LOW** · `_lym/landing/summary.md`, `## What changes`, first bullet: the added wording is inaccurate.
   - "Its directory-only check now also reads these cases as gone" follows a list that now includes "A missing path".
   - The old directory check already read a missing path as gone. Only the regular file and the leftover directory are new.

4. **LOW** · `internal/runpusher/seams_test.go`, `removals`: the new row does not fit the table's doc.
   - The doc reads "removals change a pair's worktree as a removal leaves it".
   - `ThePathIsARegularFile` is not a state the code or the summary attributes to a removal. `worktreeGone`'s new doc lists "a file at the path" apart from a removal's leftovers.

The other siblings I checked still agree: the `ParentDeps` docs in `parent.go` ("pair is gone", "as long as its pair stands"), the `doc.go`, `target.go`, `start.go` and `PruneLedger` wording, and the summary's merge-danger text.

Tool calls (3 of 6):
- Grep for "gone / no longer stands" wording across the packages and `docs`: my context lacked the sites outside the files the fixer read.
- Read of the `ParentDeps` field docs and `HoldPair` use in `parent.go`: the file was not in context.
- Grep for `PruneLedger` and worktree-exists wording in `docs` and the package docs: needed to find twins elsewhere.
