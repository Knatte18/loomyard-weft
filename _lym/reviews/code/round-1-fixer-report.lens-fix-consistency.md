```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for agreement with its surroundings (docs, siblings, twins), through the fix-consistency lens.

**LOW. Carrier doc does not mention the new end-of-sequence rewrite.**
- Changed text: `internal/hibernateengine/carrier.go`, `wakeWatch` and its call in the Tick goroutine.
- `internal/hibernateengine/doc.go:49-61` describes the Carrier's take-up and re-exec hold and says nothing about the file rewrite. It is the only place that states what wakes the tick.
- `internal/beamcli/doc.go:147` says "a written request wakes the tick that takes it up". That is now also true of a sequence ending, which the doc leaves out.
- A reader of those docs would not learn why the request file is rewritten after a sequence ends.

**LOW. The hibernatecli doc does not mention the directory created by `NewDobbyCarrier`.**
- Changed text: `internal/hibernatecli/deps.go`, `NewDobbyCarrier` now calls `os.MkdirAll(hibernateDir(prime))`.
- `internal/hibernatecli/doc.go:22` describes `NewDobbyCarrier`, and the function's own doc comment (`deps.go:51`) also omits the new side effect. Both still read as if it only builds deps.
- The inline comment above the `MkdirAll` call explains why, so the gap is the doc and the function comment.

**LOW. Wording of the extended resuming reason.**
- Changed text: `internal/hibernateengine/watch.go:69-72`.
- The base reason is "…ended in phase resuming, so its wake could not be seen" and the fixer appends ", the wake reconcile did not run, and `lym cleanup --apply` runs it". The result is a run-on clause chain inside the notice's "failed: <reason>; <outcome>" sentence.
- The `carry` doc comment (`watch.go:61`) says only that the request "ends failed at once", so it also omits the skipped reconcile.
- This is readability only.

**Checked, no finding:**
- The F1 test compares the `fabriccli` literals with `Request.Terminal()`, and `hibernateEndedPhases` matches the engine's `PhaseDone`, `PhaseFailed` and `PhaseCancelled`.
- `UpdateJSON` already runs `MkdirAll` on the request's directory at write time, so F3 is still needed: the watch opens before the first write.
- A remaining `daemon` wording in the `hibernatecli`/`fabriccli` comments (for example the `hibernateDaemonLive` name) is the engine's own `DaemonPID` vocabulary, not a twin of F6.
- The existing `TestTakeUp` row still matches on "ended in phase …", which the new reason keeps.

Tool calls made:
1. `grep` over `internal` and `docs` for daemon wording, "ended in phase" and "wake reconcile", plus the Carrier and request-file mentions in the two `doc.go` files. My context lacked the doc files' text.
2. `grep` of `state.UpdateJSON` to confirm whether it creates the directory and always writes. My context lacked `internal/state`.
