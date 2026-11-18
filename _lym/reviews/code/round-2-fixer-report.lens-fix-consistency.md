```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for agreement with its surroundings, through the fix-consistency lens.

**[MEDIUM] `sessionHolder` aborts at the first bad registry entry, so the holder-registration caller loses a holder it used to find.**
- Changed text: `internal/shuttleengine/claudeengine/resume.go`, `sessionHolder`. The loop now returns an error as soon as it meets a record with `ReadErr` or `DecodeErr`.
- Why it conflicts: `readRegistry` returns records in directory order. A bad entry listed before the live entry for the session therefore hides that entry.
- Evidence: `internal/orchcli/start.go:203` (`StartSession`) does `if pid, found, err := run.SessionHolder(); found { HolderPID = pid } else { HolderErr = err }`.
  - Before the fix, an unrelated unreadable entry was skipped and the holder was found.
  - Now `HolderPID` is lost and `HolderErr` is set for every session while any bad entry exists.
- The `SessionHolderFinder` doc in `internal/shuttleengine/engine.go:387-391` says the caller reads it "to register the provider process of a session it launched", so that contract regresses.
- Suggested change: keep scanning, return the live match if one is found, and return the remembered error only when no match is found. `stepstop.go` and `relaunch.go` still refuse on the error, because they ask only "is it held?".
- Test gap: the new `UndecodableEntryIsAnError` row has a single bad entry and no good one, so it cannot catch this. A row with a bad entry plus a live matching entry would.

**LOW — no finding:** the `loopExitStop` doc and the doc.go/`OnStepOrphaned` wording ("newest interrupted record") still agree with the new record writer. `deadLoopStop`'s own doc doesn't mention the record, which is fine because the `loopExitStop` doc carries it.

Tool calls:
1. grep for "interrupted record" and related phrases across non-test Go and markdown files, to find sibling docs.
2. `sed` of `internal/shedverbs/doc.go` lines 196-210, `internal/shuttleengine/engine.go` 384-396, `internal/shedverbs/loop.go` 85-95 and `internal/shuttleengine/run.go` 1090-1110, to read those sibling docs and `Run.SessionHolder`, which my context did not hold.
3. grep for `.SessionHolder(` call sites, to find every consumer of the changed lookup.
4. `sed` of `internal/orchcli/start.go` 195-222, to read the registration caller's error handling.
5. grep for "undecodable|unreadable" in `internal/shuttleengine` non-test files, for docs that say entries are skipped. It found none.
