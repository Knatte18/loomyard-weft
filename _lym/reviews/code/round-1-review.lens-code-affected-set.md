```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set: I checked what the diff's changes force outside the lines it touches.

**Per change, the set found:**

1. **Refusal path now pushes the records and may carry `push_error`** (`internal/webstercli/reset.go:194-206`).
   - Readers of `push_error` elsewhere: `fabriccli`, `battencli` and `cleanupcli` each build their own envelope. None reads webster reset's. Covered.
   - `output.ErrFields` (`internal/output/output.go:34`) merges the extra fields into the refusal envelope. Covered.
   - Help text (`reset.go:65-67`) says a refusal whose commit landed still pushes, and that a failed push lands in `push_error`. Covered.
   - Package doc (`internal/webstercli/doc.go:35-38`) gains the new sentence. Its hand-over line ("fails the verb, with the reset already made") still holds, because the failure path refuses anyway. Covered.
   - File header (`reset.go:6-7`) is updated. Covered.
   - `pattern/PATTERN-run-fsm.md:45-46` says the reset's own records push writes an owed record that sets `PushOwed`. The diff now makes the failure path do this too. Covered, no edit needed.
   - `pattern/PATTERN-pair-remote.md:37-41` lists the reset's records push as synchronous, its owed record holding the run. Covered, no edit needed.
   - `PATTERN.md:25` and the `PATTERN-pair-remote` summary are unaffected. Covered.
   - `contracts/specs/refusal-spec.md` sets no rule on extra fields in a refusal envelope, so `push_error` on a refusal breaks nothing there. Covered.
   - `contracts/stencils/webster/webster-template-master.md` mentions `webster reset` but says nothing about its push or sync. Nothing to change.
   - `PATTERN-refusal-way-forward`: the refusal keeps `fabricSyncWayForward`, and the new row reaches this refusal. Covered.

2. **`pushResetRecords` extraction** (`reset.go:247-260`).
   - Its callers are the two call sites in `reset.go` only. `pushRecords` is set in `wiring.go:231` (hub) and left nil in `wiring.go:370` (standalone). The skip conditions are unchanged. Covered.
   - The warning log line keeps its text. No test or doc asserts it. Covered.

3. **The "the fabric sync failed" message** is unchanged. Sibling verbs use their own message prefixes, and the only assertion on reset's message is the new row (`reset_test.go:222`). Covered.

4. **Test row and doc comment** (`reset_test.go:168, 182-189, 210-227`).
   - The `PATTERN-run-fsm` "Held by" list names `TestResetCmd_RecordsTheRunTransitionAtTheNewTip`, which still exists. Covered.
   - The `wantRefusal` helper is not used by the new row, so it is unaffected.
   - The ok assertion on the existing rows is kept, so no check is loosened. Covered.

5. **Commit messages** (`git log` of the range).
   - The `Root-Cause:` trailer on 205ad7053 names the defect the diff removes: the return before `c.pushRecords` that ignored `committed`.
   - No commit carries a `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.
   - The directive declares no PATTERN changes, so none have to land.

**Findings:** none.

**Tool calls:**
1. Grep for `push_error`, "the fabric sync failed" and the reset's push log line across `.go`, `.md` and `.yaml`. The manifest held no readers outside `reset.go`.
2. Grep the docs and stencils that name `webster reset` for push and sync statements, and read `output.ErrFields`. The manifest lacked the stencils and specs and the envelope builder.
3. Grep the stencil, `refusal-spec.md` and `PATTERN-refusal-way-forward.md`, and read `PATTERN-pair-remote.md:30-45`. The manifest held only those pair-remote lines' grep hits, not their text.
4. Grep `refusal-spec.md` for envelope field rules. The manifest lacked that spec.
