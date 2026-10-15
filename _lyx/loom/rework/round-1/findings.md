---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: internal/creelengine/verbs.go:130
    summary: read and read --all return different sets than the board names, breaking D8 and its acceptance criterion
    fix: In Store.Read (internal/creelengine/verbs.go), make plain read (no from, no all) return the holder's oldest unread message together with every other unread message from that message's sender, read --from return every unread message from that sender, and read --all return every unread message of the holder and no read ones; reserve and ack exactly the messages returned. Update the --all flag help in internal/creelcli/cli.go (include the read messages still kept), the read paragraph of internal/creelengine/doc.go (line 75), and the read/read --from/read --all rows of TestStore_Mail in internal/creelengine/verbs_test.go to pin these three sets, including a holder with unread mail from two senders where plain read returns only the oldest message's sender.
---

### [BLOCKING] `read` and `read --all` return sets the board does not name

**Location:** `internal/creelengine/verbs.go:130` (`Store.Read`), with `internal/creelcli/cli.go:428` and `internal/creelengine/doc.go:75`.

**Issue:** D8 decides that `read`, `read --from <sender>` and `read --all` "print the messages the board names", and the acceptance criterion repeats it: they "return the sets the board names".
The board entry's Reading section (operator, 2026-10-06) names them: `read` is the oldest unread message together with every other unread message from the same sender, `read --from` is every unread message from that sender, and `read --all` is every unread message.
The change implements plan card 7's wording instead: plain `read` returns every unread message from every sender, and `read --all` adds the read messages still retained.
So plain `read` drains all senders at once where the operator asked for one sender's thread at a time, and `read --all` returns read mail the board never puts in it.
The plan departed from D8 here, and both the webster summary and the landing summary report the departure without a decision that overrides D8.

## Carried-over risks

- Webster-Review F1 (`external-sender-name-formed-outside-agentname`): fixed by `89c5a50d7`, which forms the EXTERNAL name through `agentname.FormatProgram`.
- Plan-Review `idle-paused-errors-unhandled`: `deliver.go` treats an `Idle` or `Paused` error as not deliverable and reports it, so it is neither dropped nor read as idle.
- Plan-Review `reply-does-not-wake-scheduler`: `deliverMailLocked` wakes the receiver for a post, reply and forward alike.
- Discussion-Review `sender-identity-fails-open-on-unreadable-state` and `waiting-verbs-unbounded`: D4 refuses a peer in no known pane when state cannot be read, and D12 bounds the waits with `--timeout`; both match the decisions.

## Outstanding step outside the diff

D20 cuts the `creel-core` board body to a pointer at `internal/creelengine/doc.go` in the same task.
The body still carries the full design: `lyx board upsert` refused the edit while the run held the entry, and the landing summary hands it to the operator.
This is not a finding against the code, since the run cannot make the edit, but the cut is still owed at landing.

## Overlaps

- `engine-registry` (row 17, `running · Merge-In`; this run's `own_row` is 22).
  Both change the record lookup in shuttle's `Runner.Send` and `Runner.Interrupt` (`internal/shuttleengine/run.go`).
  The designs agree: creel's D17 drives a strand with no run record through the runner's default engine, and engine-registry's D4 gives such a strand the set's default engine; each record names the other and has the second lander adapt (creel's first open risk, engine-registry's R1).
  Both edit `pattern/PATTERN-pane-text-injection.md` at different bullets (creel adds the record-less send, engine-registry drops `ModelSwitchSequence`), both add a package to `PATTERN-told-geometry`'s bound list, and both shorten the same `PATTERN.md` index lines (`PATTERN-hub-containment`, `PATTERN-glyph-conversion-chokepoint`, `PATTERN-edit-directive`) for headroom with different wording but no rule changed; these are text overlaps.
  Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml`, the module table in `docs/overview.md`, and a new PATTERN index line.
  Landing order: `creel-core` is further along (row 22 against 17), so it lands first and `engine-registry` takes its changes at Merge-In, where its R1 already plans to move creel's `Runner` construction onto the engine set.
- `lighter-test-rules` (row 3, `running · Discussion-Write`): no changed paths, and its scope (stencil prose assertions) touches nothing creel changes; no overlap.
- Parent commits: `lyx loom peers` lists none.
