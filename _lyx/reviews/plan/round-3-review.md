---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/17-creel-read-sets.md:28-34
    summary: Read_SetsAndInbox never pins the oldest-message rule for plain read, since alpha is at once the sender of the oldest message and the sender with the most unread messages
    fix: "In card 17's Intent, replace the first posting and plain-read segment of the `Read_SetsAndInbox` sequence so the oldest message's sender is not the sender with the most unread messages. Post `b1` from batten, `a1` from alpha, `b2` from batten, `a2` from alpha and `a3` from alpha. A plain read gives exactly `b1`, `b2`, with `a1`, `a2`, `a3` left unread and counted by `Inbox`; the next plain read gives exactly `a1`, `a2`, `a3`; a further one gives nothing. Update the Inbox expectation the Intent states for that point to batten 2 and alpha 3 before the first read, and relabel every later message of the sequence (`a3`, `b2`, `a4`, `b3`, `b4` today) to labels not used yet, such as `a4`, `b3`, `a5`, `b4`, `b5`, so no label repeats. The rest of the sequence (a `from` read of the non-oldest sender, a `from` plus `all` read, an `all` read, a final empty `all` read) stays as it is."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/17-creel-read-sets.md:39
    summary: The Intent says two exact bodies in the new protocol row fail the old read-plus-unread set, but the row holds no read message, so the old `all` hands out the same two bodies
    fix: "In card 17's Intent, in the sentence that begins `It asserts the bodies handed out are exactly the two posted ones`, drop the clause `and asserting exactly two bodies fails the old read-plus-unread set as well`. State instead that the row fails a plain read, which hands out one sender's body only, and so fails an `All` dropped before `Store.Read`; the `Read_SetsAndInbox` sequence is what pins that no read message comes back."
---

### [MEDIUM] The oldest-message rule of plain read is not pinned by the sequence

**Location:** `_lyx/plan/17-creel-read-sets.md:28-34` (the `Read_SetsAndInbox` sequence in the Intent).

**Issue:** D8 and the findings name plain `read` as the oldest unread message together with every other unread message from its sender.
In the card's sequence the first plain read is the only one that picks among senders: alpha holds `a1` and `a2` against batten's one `b1`, so alpha is both the sender of the oldest message and the sender with the most unread messages.
An implementation that picks the sender with the most unread messages therefore passes every step of the row.
The later plain reads have a single sender left, and the `from` reads name their sender, so no step distinguishes the oldest-message rule from a busiest-sender rule.
The row rules out the old `read` and `read --all`, but not this alternative reading of "the oldest message's sender", which the board's wording excludes.
The proposed fix makes the oldest sender batten, who holds fewer messages than alpha, with its messages interleaved among alpha's.
The first plain read then gives `b1`, `b2` under the correct rule and `a1`, `a2`, `a3` under a busiest-sender rule.

### [LOW] The protocol row's claim about the old read set is false

**Location:** `_lyx/plan/17-creel-read-sets.md:39`.

**Issue:** The Intent says that asserting exactly two bodies in `ReadAllHandsOutEveryUnreadMessageFromEverySender` "fails the old read-plus-unread set as well".
In that row both posted messages are unread and none has been read, so the old `Store.Read` with `all` set, which returned the unread messages plus the read ones, hands out the same two bodies.
The row passes under the old semantics; it only fails a plain read and an `All` dropped before `Store.Read`, which is what the focus directive asks of it and what the row does deliver.
The sentence misleads the implementer into believing the row also guards the no-read-messages half, which only the `Read_SetsAndInbox` sequence's final `all` reads do.

## Rejected

- Every sender of the new `TestDaemon_Protocol` row resolves under `asStrand` false on Linux: `creelfake.LiveStrand` gives every strand `DefaultPanePID` (999999999), which no test-process ancestor holds, so `paneOfPeer` returns no pane and no unreadable reason; `programClaim("ly:BATTEN")` and `programClaim("ly:CARTER")` pass `checkProgramClaim` and resolve as program senders, and `externalClaim()` resolves as EXTERNAL; the existing `ReplyReachesAWaitingExternalPost` row runs the same shape.
- Dropping `Request.All` before `Store.Read` in `server.go` makes the new row read as a plain read, which hands out the EXTERNAL sender's one body, so the two-body assertion fails as the card says.
- The `Read_SetsAndInbox` sequence reuses no label after it was read, and its read steps after the first fix name the unread set; its `from` plus `all` step and its final `all` steps fail under the old `all` semantics, which would add the read `b1`, `b2` and, later, every read message.
- Every comment or doc sentence that says which handed-out messages are reserved or unread is named for rewording or consistent already: `Reading`, `Store.Read`, `Request.All`, `readCmd` and `doc.go` are Edit targets, and the grep over `still kept`, `still retained` and `unread ones` finds exactly the sites the Intent lists. `daemon.read`, `Reading.Ack`, `Reading.Release`, `unreadLocked` and the `reserved` field comment stay true under the new sets.
- Verify coverage: the `## verify:` section runs `go build`, the three `go vet` tag compiles, `go test ./...` and `go test -tags integration ./...` in the template's order, so `internal/creelengine` and `internal/creelcli` are covered.
- Other tests that read through `Store.Read`: every other plain read in `verbs_test.go`, `deliver_test.go`, `server_test.go` and `creelcli/cli_test.go` has one sender, and the `holderOf` helper's callers hold only unread messages, as the card says.
