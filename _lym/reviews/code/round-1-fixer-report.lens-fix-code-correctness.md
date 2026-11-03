```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed branch and each added test by hand against a concrete input.

No findings.

- **`checklistPoll(location, waiting)`**: the only call site is `checklist.go:127`, and it now passes `reviewWaitingFor(location)`, so behaviour in production is unchanged. The `waiting != nil` guard is still there, so a nil hook still gives no note. No test called `checklistPoll` before, so no other caller broke.
- **`TestChecklistPoll_WaitLineKeepsTheNoteText`**:
  - Running case: no seed is recorded, so `rowsErr` is set and the frame is the recipe-unreadable list. The last line is the wait line, `"waiting Build: verify running 6m"`. Taking `note.Steady` instead would give `"... <elapsed>"` and fail the test.
  - Done case: the hook is skipped and `checklistWaitLine` returns `"waiting on nothing"`.
  - A failing pause geometry or an absent pause record is tolerated, so the test does not depend on a `.git` entry.
  - The case runs in parallel with its own `t.TempDir()`.
- **`TestStatusCmd_WaitingNote_RunningOnly` with `Steady: "steady form of " + note`**: with `Text` empty the envelope still gets no `waiting` key, because `statusCmd` checks `waiting.Text != ""`. Putting `Steady` into the envelope would break the first subtest.
- **F3 addition in `TestStatusWatchLine_AppendsWaitingNote`**: a missing status file takes the unavailable branch, whether the read errors or returns `!found`. The test asserts `line == key == UnavailableLine("loom")`, so a key that differed from the line would fail.
- **Gate**: the parent's gate run compiled both packages and passed (tier0: 1617 passed, 0 failed).

Out-of-scope note: the one-shot human view carries `waiting.Text` too. The test cannot assert it because `writerIsTerminal` is false in tests, so that half of F1's fix text is only covered through the envelope.

Tool calls: none beyond the context.
