---
verdict: APPROVED
---

The run at `0cad35749` matches the `creel-core` board entry's goal (creel stage 1: daemon, pool and journal, every verb, delivery through shuttle gated on idle, no module sending through it yet) and the decisions D1–D20.

## Attempt 1's finding

Attempt 1 rejected on D8: `read` and `read --all` returned sets the board does not name.
Card 17 (`3456a3177`) fixes it.
`Store.Read` (`internal/creelengine/verbs.go`) now hands out the oldest unread message with every other unread message from its sender for plain `read`, every unread message from one sender for `--from`, and every unread message for `--all`, never a read one, and reserves exactly what it hands out.
The `readCmd` help, the `Request.All` comment and the read paragraph of `internal/creelengine/doc.go` name the same three sets.
`TestStore_Mail`'s `Read_SetsAndInbox` row pins them with two interleaved senders, the oldest sender holding fewer messages, and `TestDaemon_Protocol`'s `ReadAllHandsOutEveryUnreadMessageFromEverySender` row pins `All` across the socket.
This also settles the Plan-Review carry-over `read-sets-inbox-sequence-ambiguous`, since card 17 fixes the exact sequence and the test follows it.

## Commits since attempt 1

- `487044936` merges `engine-registry` (`88ea645bb`, now on `main`) into the branch.
  D17's record-less strand is driven through `engineFor(RunState{})`, the set's default engine, which is what D17 and engine-registry's own decision both say; creel's adapter (`internal/creelcli/delivery.go`) takes its engine set through an injected `enginesFor` over `internal/shuttleengines`.
- `b7d97c892` adds the merge review's F1 test: record-less `Send` and `Interrupt` consult the set's default engine.
- `0cad35749` shortens the `PATTERN-creel-store` index line for the headroom gate.
  The rule is unchanged: `pattern/PATTERN-creel-store.md` still states D20's entry in full, the daemon as the journal's only writer included.

## Carried-over risks

- Discussion-Review `sender-identity-fails-open-on-unreadable-state` and `waiting-verbs-unbounded`: D4 refuses a peer in no known pane when state cannot be read, and D12 bounds every wait with `--timeout`; attempt 1 found the code matches both, and card 17 does not touch them.
- Plan-Review `read-sets-inbox-sequence-ambiguous`: settled as above.
- The record's open risks (Windows AF_UNIX unverified, identity bounds off Linux, subject prompt-injection, orch unproven until `creel-notices`, sweep cost) are bounds the decisions accept, not departures.

## Outstanding step outside the diff

D20 cuts the `creel-core` board body to a pointer at `internal/creelengine/doc.go` in the same task.
The body still carries the full design, since the run cannot write its own board entry while it holds it.
The cut is owed at landing; it is not a deviation in the code.

## Overlaps

- `engine-registry`: landed on `main` as `88ea645bb` and already merged into this branch; the merge review found the two sides' shuttle `Send`/`Interrupt` lookup, `pattern/PATTERN-pane-text-injection.md` and `pattern/PATTERN-told-geometry.md` edits consistent.
  `lyx loom peers` lists no parent commits beyond it.
- `engine-config-seams` (row 7, `running · Plan-Write`): no changed paths yet.
  Its record plans to change how shuttle runners and reed engines are constructed, and names creel's construction sites in `internal/creelcli` for the second lander to adapt; no decision conflicts with creel's.
  Landing order: `creel-core` (own row 22) lands first and `engine-config-seams` takes its changes at Merge-In.
- `engine-model-choice` (row 5, `running · Discussion-Burler-Review`): no changed paths yet.
  Its record names `creel-core` as touching `shuttleengine` in parallel, with whichever lands second adapting; it keeps parent asks on `SendMessage` and leaves the move to creel to `creel-agents`, which agrees with this run's scope.
  Landing order: `creel-core` lands first and `engine-model-choice` takes its changes at Merge-In.
- `lighter-test-rules` (row 9, `running · Plan-Burler-Review`): no changed paths; its record's `claims_test.go` risk names an earlier state, and this run's diff touches no `claims_test.go`.
  No overlap.
- Shared bookkeeping, not overlaps: `cmd/lyx/testdata/test-budget.yaml`, the creel line in `docs/overview.md`'s module table, and the new `PATTERN-creel-store` index line in `PATTERN.md`.

## Superseded

None: `creel-notices` and `creel-agents` are stages 2 and 3, which build on this one.
