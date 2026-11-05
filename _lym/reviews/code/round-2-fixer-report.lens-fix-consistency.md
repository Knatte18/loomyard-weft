```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for consistency with its surroundings (lens fix-consistency).

**Findings**

1. **LOW** — `internal/fabricengine/shortnamebinding_clone_integration_test.go`, the new comment above `SecondCloneDerivesShortname`.
   - The comment says the subtests "all read the one weft fixture and existing hub (firstParent and first)".
   - `SecondCloneDerivesShortname` and `DisagreeingShortnameRefuses` use only `weftURL` and a fresh `t.TempDir()`. They never touch `firstParent` or `first`.
   - Only `ResetWithDisagreeingShortnameKeepsHub` and `ResetKeepsRecord` touch the hub. The comment states the hub dependency for subtests that don't have it, which is the sibling-mismatch pattern the review's F3 raised.
   - A reader will notice that the reason given doesn't hold for the first two subtests.
   - The fix is to name the weft fixture as the shared state for all four and the hub for the last two only.

2. **LOW** — `internal/fabricengine/livestate_doc_test.go`, row (8) of the neutering log.
   - The fixer changed the two attributions the review named.
   - The same entry still says "a gate refusal, not the pre-flight's" and "proving the pre-flight itself is load-bearing", which are twins of the changed text.
   - They now read as an unnamed guard, though the context makes it recoverable as `checkResetTarget`.
   - The quoted `expected pre-flight refusal` is a test message from the past run, so it correctly stays.

**Checked and consistent**
- `_lym/landing/summary.md` now matches `resetsubpathcheck.go`:
  - The staged check runs before the reset.
  - It refuses a disagreeing anchor, or a `--subpath` absent from the warp when no anchor is recorded.
  - It uses `fetchWeftBranchTip`.
- The summary's test list matches the four tests present, including the new `TestCloneHub_ResetSubpathCheckReadsRecordedAnchor`.
- The new test's three rows map one-to-one onto the three untested branches in F2.
- The doc comment of `TestCloneHub_ResetRefusalKeepsExistingHub` (four refusals) agrees with the summary's list.

Tool calls made: none. My context already held every changed file and the review.
