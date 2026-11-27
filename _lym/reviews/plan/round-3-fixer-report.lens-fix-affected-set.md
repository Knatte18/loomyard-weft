```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's set search and check that the fix covers every member (fix-affected-set lens).

Findings left unchecked: none.

No findings.

- F17: I searched card 14 for every `gitexec`/`gitrepo` error, `GitError`, `ErrPush`, "standing in", "built in process" and transport. Every test use is covered:
  - Line 64 states the rule.
  - Lines 65 to 67 cover transport, rejected and auth-refused.
  - Line 77 is the joined read-failure row, which wraps provoked transport errors.
  - Line 82 is `TestOutcomeClassOf`.
  - Lines 47, 48 and 52 are behaviour text, not test inputs.
  - Neither "standing in" nor "each error built in process" is left. The only "built in process" is line 69, which covers the engine's own types and `errors.New`, as the fix allows.
  - The tier is now tier1 (line 63). The fixer's earlier grep showed the overview's lines 86 and 87 name no tier for either test, so no twin is left stale.
- F18: I searched card 15 for the reason phrasings the review flagged ("earlier subtests", "same assertions", "its sibling", "same loop"). The only one left is line 136, which the fix rewrote to say what it holds and why the dropped check guards no rule. The other two flagged reasons (lines 144 and 149) no longer use those phrasings. Each now names a holder whose calls match the code shown at `destructivegaps_integration_test.go:468`, `:521` and `warpexclude_integration_test.go:149`. I did not reopen the other deletion reasons the review accepted.

Tool calls:
- One grep over `_lym/plan/14-executor-loop.md` and `_lym/plan/15-snapshot-reader.md`, to rerun both findings' searches over the changed cards as one set. My context held the edits but not a full search of either card for remaining members.
