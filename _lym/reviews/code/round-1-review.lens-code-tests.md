```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that pins it.

**Behaviours and the tests that pin them**

1. **The check skips an untracked session usage record under `_lym/usage`.** Pinned by `TestCheckResolved_LymOwnStateIsExempt`, for both anchors `.` and `backend` (`internal/preflight/preflight_integration_test.go:476-479`, then `assertCheckSet(t, report)` at about line 488).
   - The record goes through the real writer, `shedrun.WriteSessionUsage`, into the real directory, `shedrun.UsageDir(l)`.
   - The unscoped `fabricengine.Clean(l)` assertion at about line 481 proves the record is visible to git first, so a pass is never a gitignore's doing.
   - If `shedrun.UsageDirRel()` is removed from `runStateExemptRels` (`internal/preflight/preflight.go:31`), `CheckResolved` reports `CheckWorktreeClean` and `assertCheckSet(t, report)` fails.
   - The `backend` subtest pins that the entry is rooted at the anchor, the same as the other exempt entries.
2. **The exemption stops at the exempt paths, so other `_lym` files still refuse.** Pinned by the same test (about lines 490-495). It writes `_lym/plan/plan.md` and asserts `CheckWorktreeClean`.
   - This assertion existed before the diff and still holds with the new entry.
   - It would fail if the exemption grew to cover all of `_lym`.
3. **A usage record that was tracked and then modified or deleted is skipped.** No row for the usage directory itself.
   - The diff adds no separate code path for it. The exclusion is one pathspec through `exemptPathspec` (`internal/fabricengine/warpclean.go:52-61`), and git applies it the same way to every status.
   - The same test already pins tracked-then-deleted content for another exempt entry: the committed halt note that is removed and moved into an archive (lines 448-455).
   - So this is not a behaviour the diff adds without coverage, and I raise no finding.
4. **Doc-comment changes in `doc.go`, `preflight.go` and the test's doc comment.** Static prose, exempt from this lens.

**Findings:** none.

- The redundancy report lists the changed test as unique. The new assertion is one more fixture inside the existing test, not a new test, so it does not duplicate coverage.
- The expected value is an empty failure set, written as a literal. It is not computed by calling the code under test, so the test is not tautological.
- The mutation report shows 0 uncovered changed blocks and 0 survivors.

Tool calls: none. The inherited context already held the diff, the test body, `UsageDir`/`UsageDirRel`, `WriteSessionUsage`, `Clean`/`exemptPathspec`, and the redundancy, reach and mutation reports.
