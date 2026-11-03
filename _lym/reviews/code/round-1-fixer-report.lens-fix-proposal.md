```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1, F2 and F3 proposals as the fixer applied them.

Fixed as proposed: F1, F2 and F3, with F1 and F3 sharing one commit. I built a counter-case against each and found no defect.

- **F1:** `Steady` is now `"steady form of " + note`.
  - Swapping `waiting.Text` for `waiting.Steady` in the envelope would give `env["waiting"] == "steady form of the parent's review"`, so the first subtest fails.
  - In the empty-note subtest `Steady` stays non-empty while `Text` is empty. That is intended, because it checks that `statusCmd` gates on `Text`.
  - The reviewer's proposed human-view assertion was not carried out, because the human view needs a terminal writer and `execEnvelope` does not give one. This is a coverage gap in the proposal rather than a defect in the fix, and the envelope assertion still pins `Text`.
- **F2:** The proposed test was not possible against the old code, because `checklistPoll` built its hook internally through `reviewWaitingFor`. The fixer changed the signature to `checklistPoll(location, waiting)` and moved the call into the single caller at `checklist.go:127`.
  - The only caller (`checklist.go:127`) now passes `reviewWaitingFor(location)`, so behaviour is unchanged. I checked the grep shown above and found no other caller.
  - Using `&loomCLI{}` is safe, because `checklistPoll` never reads `c`.
  - A temp-dir `Location` has no `.git`, so `PauseGeometry` may error. The code already tolerates that through `pauseErr`, and `readChecklistPaused` returns nil on a read error, so the last line is still the wait line.
  - A missing seed only sets `rowsErr`, and the frame still ends with `stateLine, waitLine`.
  - With the defect in place the Running row would read `waiting Build: verify running <elapsed>`, so it fails as required.
- **F3:**
  - Whether `ReadJSON` returns an error or `found == false` for the absent file, `statusWatchLine` takes the same unavailable branch, so the assertion holds either way.
  - The `Spec` literal needs no `Hooks`, because that branch returns before the hook runs.

One note, below `LOW`:
- The Done row in `TestChecklistPoll_WaitLineKeepsTheNoteText` (`checklist_test.go`) does not exercise Text versus Steady. It checks that a settled run skips the hook and waits on nothing. `renderChecklist` and its `waitNote` cases may already cover that, so the row is probably redundant under the one-test-per-behavior rule. It is harmless and fails if the state gate is dropped.

Tool calls made: 0. My context already held every changed file, so I read nothing further.
