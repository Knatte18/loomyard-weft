```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what else calls, reads, asserts or describes it, and judge whether the diff leaves any of it wrong.

**What each change reaches, and whether it is covered**

1. **`recordsSideNotLanded` asks origin before reporting ended, and wraps `errRecordsUnpushedPastPush`** (`internal/fabricengine/pushanchored.go:313-340`).
   - Its only caller is `pushRecordsSide` (`pushanchored.go:229`), which the diff updates. Covered.
   - The "ended on origin" message is asserted in `pushclasses_integration_test.go:710` and `:734`.
     Both tests delete the branch from origin first, so `branchGoneOnOrigin` reports it gone and the message stays the same.
     The `sleeps` counts (0 and 1) are unchanged, because a gone branch never takes the new retry. Covered.
   - Other places that say "ended on origin" are their own messages, not readers of this one: `hibernateengine/wake.go:121,139`, `hibernateengine/watch_test.go:956`, `cleanupcli/cleanup_integration_test.go:339`. Not affected.
   - The new sentinel is unexported and only used inside `pushanchored.go`. Covered.
2. **The records-side retry loop in `PushPairAnchored`, which sleeps `SyncOptions.BackoffSleep`.**
   - The `PushPairAnchored` doc comment (`pushanchored.go:111-113`) and the package doc (`doc.go:1135-1136`) are both updated. Covered.
   - The package doc's classified-push section (`doc.go:1509-1545`) describes `pushClassified`, which did not change. Not affected.
   - The doc on the `SyncOptions.BackoffSleep` field (`fabric.go:82`) is not updated. See finding F1.
   - `AddOptions.BackoffSleep` (`add.go:41-43`) feeds `syncOptions()`, but the add and hub paths call `CoalescePushBothAt`, not `PushPairAnchored` (`hubforge/hub.go:424`). Its doc is narrow already, but the diff adds no new route that reaches it. Not affected.
   - The sibling "landed" rule in `coalesce.go:56` (`pushSideDetached`) returns landed false with a nil error, never the "ended" error. The defect described in the board entry does not reach this path, so leaving it alone is correct. Not affected.
   - `PATTERN-pair-remote` ("records side first, code only once it landed") still holds, since the retry stays inside the records side, before the code side. `PATTERN-daemon-wakeups` concerns unbounded waits, and this loop is bounded. Covered.
3. **Test header comment** (`pushanchored_integration_test.go:178`): a duplicated line is replaced with one describing the new subtest. Covered.
4. **Commit messages and departures** (`git log 22a2c25b..HEAD`): neither commit has a `Pattern-Departure:` trailer, and there are no declared PATTERN changes. The commit message's claims (a commit made during the push, a tracking ref left unmoved, origin asked, retry over the push backoff) match the diff. Covered.

**Findings**

- **F1**
  - Severity: LOW
  - Class: consistency
  - Location: `internal/fabricengine/fabric.go:82`
  - Summary: The `SyncOptions.BackoffSleep` field doc says it "sleeps one push backoff delay before a missing branch or unreachable origin is recorded". The diff now also sleeps it between re-pushes of a records side that is still unpushed (`pushanchored.go:213-234`), so the doc no longer lists every use.
    It was already narrower than the code before this diff, because `settleRejection` (`pushclassified.go:173`) uses it too. The diff widens that gap.
  - Fix: reword the field doc by what it does, not by listing its uses. For example: "sleeps one delay of the push backoff between a push's retries; nil means sleep."

No other member of the affected set is left wrong.

**Tool calls**
1. Searched for the changed symbols and the "ended on origin" / "left to its teardown" messages, plus the `BackoffSleep` field docs. My context lacked who else reads the message, and the field's doc text.
2. Searched the whole repo (stencils, docs, PATTERN background) for the changed behaviour wording, and read the `doc.go` classified-push section. My context lacked any non-Go consumers and that doc span.
3. Pulled the matching lines from the other hits (`hibernateengine`, `cleanupcli`, `coalesce.go`, the research doc), to tell whether any of them depend on this message or rule.
4. Read `coalesce.go:50-109` to check whether `pushSideDetached` has the same "ended" fallback. That function was outside the manifest.
