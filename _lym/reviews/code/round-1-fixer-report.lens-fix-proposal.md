```yaml
lens: fix-proposal
outcome: reported
```

Task: the fix-proposal lens, challenging the review proposals the fixer carried out as written (F2, F3, F5, F6).

**LOW** — `internal/websterengine/state.go`, `LoadState` prepend (F2) against the `MasterNotePaths` doc "in spawn order, each once" and the new `MasterNotePath` doc.

Prepending the legacy key breaks spawn order when an older binary rewrote a newer state:

1. A new binary saves `masterNotePaths: [webster-master.md]`.
2. An older binary resumes the run. It keeps `masterNotePaths` as an unknown key (PATTERN-tolerant-records) and writes `masterNotePath: webster-master-2.md`, the latest spawn's note.
3. A new binary loads the state and gets `[webster-master-2.md, webster-master.md]`, newest first.

The allowance is unaffected, because `CheckParent` matches any listed path regardless of order. Only the "in spawn order" claim is wrong in this case. The `MasterNotePath` doc's "held by a state written before MasterNotePaths existed" also misses this second source of the key.

F2's other effects held up:

- `LoadState` is the only typed `ReadJSON[State]` decode of webster's state.
- `masterNotePath` is now a declared `omitempty` field, so `keepUnknown` does not carry the old key back, and the save drops it.

F3, F5 and F6: no findings.

- **F6:** `findingIdentity` is built from the session ID and `Key`, not `Detail`, so the reworded Detail changes no ledger identity.
- **F5:** removing either append guard makes its new row fail on the exact-slice comparison.

Extra tool calls:
- One `grep` for other decoders of `state.json` and for `findingIdentity`; my context did not show `auditledger.go` or the repo-wide readers.
