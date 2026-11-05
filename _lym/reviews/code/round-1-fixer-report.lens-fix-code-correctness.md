```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code changed for F1–F5 (fix-code-correctness lens).

No BLOCKING or MEDIUM findings.

- **F1, `FileRaddle` resume branch.** I ran a store write that fails after the board save, then a retry.
  - The retry skips `f.Check` and `fileRaddleEntry`, sets `filed` to the slugs, and `afterSave` writes the store with `store.json` last.
  - A part that landed in the meantime stays done, because `applyDraftRule` (`draft.go:49`) skips released slugs.
  - The loomshed row cannot hit `slug-taken`: with `AlreadyFiled` true, `verify` is skipped and `stageAndFile` still runs.
  - An unparsable proposal still reaches `verify` and is sent back as before, because `AlreadyFiled` is only asked when `parseErr == nil`.
  - I found no `RaddleFileDeps` literal left without `AlreadyFiled`: loomshed test, envkit, shedrecipe fixture and loomcli wiring all set it, and the battencli exemption map lists it.
- **F1 tests.** `ARetryAfterALostStoreWriteWritesTheStoreWithoutTheCheck` fails with the defect in place, since the second `Check` returns an error. `ADifferentHashOrCreatedStampIsNotTheDoneMarker` fails if the marker ignored hash or created stamp, since the store would keep `hash-1`. `AHalfDoneFilingIsFinishedWithoutTheChecks` fails without the skip, since the failing `findings` would send the proposal back.
- **F4.** `NoticeSession < Session` is checked after the `ProposedSession` check, so the new row (Session 1, ProposedSession 1, NoticeSession 0) reaches it. The row's three expected substrings are in the message. Without the check the row would succeed, so it catches the defect. A successful `raddle-propose` always sets `NoticeSession = Session`, so the new refusal cannot block a legitimate approve. Other `raddle-approve` references (`cli.go`, `raddletable.go`, `wiring_test.go:668`) seed no state, so they are unaffected.
- **F5.** `ReadStored` is called when the record or the log is missing, and `found` selects the log. For the new test, the old code would write the board log, so it fails with the defect in place. The "both exist" test still sees 0 `ReadStored` calls. The fresh-directory test still sees exactly 1. The missing-entry and `ReadStored`-error paths behave as before.

**LOW**
- `internal/loomcli/raddlewiring_test.go`, `TestRaddleFileDepsStoreDone`: no row covers the closure's `held.Sketch == slug` clause alone.
  - `store.Holds` also matches a Sketch whose `Filed` contains the run's slug, so a row where another Sketch's `Filed` holds `"plan"` would return true without that clause.
  - Every current row returns false for another reason: a different hash, a different created stamp, a different Sketch directory or an empty store. Deleting the clause would break no row.
  - A row with store sketch `other`, `Filed: []string{"plan"}`, and the matching created stamp and hash, expecting false, would pin it.

Tool calls:
1. `grep` for `applyDraftRule` in `internal/boardengine`, with the `raddle-approve` references in `loomcli` and `loomengine`: my context lacked `draft.go` (needed to check a done part is not reopened on resume) and other callers of the approve verb.
