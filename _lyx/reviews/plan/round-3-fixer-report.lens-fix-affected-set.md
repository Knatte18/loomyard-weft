```yaml
lens: fix-affected-set
outcome: reported
```

The fix's sets cover every member the five findings name, except that the new `gitexec` entry point isn't in the test tables that list every `gitexec` entry point.

Findings left unchecked: none. All five set searches were rerun:
- **F1:** card 14 is the only card that builds commits over a temporary index.
- **F2:** card 8's two marker files.
- **F3:** card 17's failure branches.
- **F4:** the plan's `lyx adopt` way-forward sites.
- **F5:** the callers of `RemoveMoveRecord`.

1. LOW — card 14, the new Create line `plan:internal/gitexec#RunWithEnv` and the Intent sentence "`RunWithEnv` is `Run` with `extraEnv` appended … reporting every failure exactly as `Run` does".
   Summary: `internal/gitexec`'s tests list each entry point in tables, and the fix adds a new entry point without extending them or naming a `gitexec` test target. So nothing tests the claim that it reports failures exactly as `Run` does, or that `extraEnv` is appended.
   Evidence:
   - `internal/gitexec/gitexec_test.go:228-231` holds a `forms` map over `Run` and `RunStdin` that asserts a non-zero exit is recovered as `*gitexec.GitError`.
   - `internal/gitexec/gitexec_test.go:563-574` has a row titled "a below-floor git refuses every entry point" that lists `Run`, `RunStdin`, `RunGit` and `CheckFloor`.
   - Card 14's only test target is `fabricengine_test#TestIntegrateWeft`, which reaches `RunWithEnv` only on the success path.
   - The fix would add a `RunWithEnv` entry to both tables and a row checking that the child sees `extraEnv`, which means naming the existing `gitexec` test functions as Edit targets of card 14.

Outside my lens, in text the fix did not touch: card 15:64 removes `_launchers/<A>/<slug>` without naming its removal primitive, the same shape as F2.

Tool calls:
1. Grepped the plan for temporary-index and plumbing commands, to rerun F1's set; my context held only cards 14 and 8.
2. Grepped the plan for removal sentences, to rerun F2's set across cards my context did not hold.
3. Grepped the plan for `lyx adopt`, to rerun F4's set across cards 10 and 19 and the overview.
4. Grepped the tests for lists of `gitexec` entry points, to check whether the new `RunWithEnv` belongs to a set of entry points.
5. Read `internal/gitexec/gitexec_test.go:215-240` and `556-580`, to see what those entry-point tables assert.
