---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/17-creel-read-sets.md:27
    summary: The prescribed Request.All assertion has the alpha-pane test process claim programClaim ly:BATTEN, which resolveSender refuses on Linux, so the row cannot run as written
    fix: In the Intent paragraph about TestDaemon_Protocol, keep the ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread row on asStrand true and only delete its last assertion (the read --all one that expects the acked message back). Add a new row to the same table, ReadAllHandsOutEveryUnreadMessageFromEverySender, with asStrand false. It starts the daemon, has a client with programClaim("ly:BATTEN") send VerbRead once so the mailbox is known, then has a client with externalClaim() and a client with programClaim("ly:CARTER") each post one message to "ly:BATTEN", and sends the BATTEN client a Request with Verb VerbRead and All true. It asserts the bodies handed out are exactly the two posted ones, in post order, which a plain read could not hand out because they come from two senders and it hands out one sender's. Say in the Intent that the row is asStrand false because on Linux a program claim from a non-orch strand pane is refused, and that exactly two bodies are asserted so the old read-plus-unread set fails it.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/17-creel-read-sets.md:26
    summary: The Read_SetsAndInbox sequence reuses the label b1 after it was read and does not say which messages are unread or read at the from-plus-all step, so the row may not separate the new set from the old one
    fix: Replace the sentence after the first plain-read steps with an explicit sequence of posts and reads, each with its exact want. After the first set (plain read gives a1 and a2 with b1 still counted by Inbox, the next plain read gives b1, a further one gives nothing), post a3 from alpha, b2 from batten and a4 from alpha. Then read with from set to batten (the sender that is not the oldest) and expect only b2, leaving a3 and a4 unread. Post b3 from batten, read with from batten and all set and expect only b3 (the old set would add the read b1 and b2, and without the narrowing it would add a3 and a4). Post b4 from batten, read with all set and expect a3, a4, b4, then read with all set once more and expect nothing.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/17-creel-read-sets.md:22
    summary: The wording changes miss three sentences that say the unread ones among the handed-out messages are reserved, which no longer holds once every handed-out message is unread, and the closing grep cannot catch them
    fix: Add `internal/creelengine#Reading` to the Edit list; a trial run of validate-plan --overlay with that line added reports no finding the baseline lacks. In the Intent say that the doc comment of Reading changes from its unread ones to the messages it hands out being the unread ones reserved until Ack or Release, that the second line of Store.Read's comment (The unread ones are reserved so a concurrent read by the same holder skips them) says They are reserved, and that the second sentence of internal/creelengine/doc.go's read paragraph (The unread ones are reserved until the client acknowledges them) does the same. Extend the closing grep paragraph to also search for unread ones over internal/creelengine and add verbs.go's Reading comment to the sites it lists.
---

### [BLOCKING] The Request.All assertion needs a second sender the asStrand rig cannot give

**Location:** `_lyx/plan/17-creel-read-sets.md:27`

**Issue:** The Intent keeps the replacement assertion inside the `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` row and has a second client with `programClaim("ly:BATTEN")` post to alpha.
That row runs with `asStrand` true (`internal/creelengine/server_test.go:272`), where `newRig` sets alpha's `PanePID` to `os.Getpid()` (line 68), so every call from the test process resolves to the alpha pane.
In `resolveSender` (`internal/creelengine/identity.go:40-50`) a peer inside a pane with a program claim is accepted only when the pane is an orch strand, so alpha gets `creel: ly:alpha:driver may not sign as the program ly:BATTEN`.
The test runs with `GOOS` set to `runtime.GOOS` (line 76), so on Linux the second post fails and the assertion that is meant to pin `All` crossing the protocol never gets that far.
The other rows that use `programClaim` run with `asStrand` false (for example `ReplyReachesAWaitingExternalPost`, line 328).
The rig has no second pane process, and an `externalClaim` from the same process also resolves to alpha, so no other second sender exists inside this row.
The row also needs an exact count: "hands out both" is also satisfied by the old set, which would return the acked note as a third message, so the assertion has to be exactly the two posted bodies.
The `Request.All` guard itself is sound in principle: with `All` dropped before `Store.Read` at `server.go:380`, a plain read hands out one sender's message and the two-body assertion fails.
Only the setup cannot be built as written.
The round-1 review proposed this setup and the round-1 fixer applied it as proposed; no round disputed it, and this finding is not a re-raise but a defect the fix carried in.
The round-1 fixes for the `read --all` plus `--from` row and for `Store.Read`'s doc comment hold: the Intent names both, and F2 above only tightens the first.

### [LOW] The Read_SetsAndInbox sequence leaves the from-plus-all step ambiguous

**Location:** `_lyx/plan/17-creel-read-sets.md:26`

**Issue:** The Intent says the row reads `a1`, `a2` and then `b1` through plain reads, then "a fresh set read with `--from`", then "a read with `from` set to batten and `all` set, while `b1` and an alpha message are still unread, returns only `b1`".
By that point `b1` has been read, so the label is reused for a different message or the sequence contradicts itself.
The step separates the new `all` from the old only if a read batten message sits in the pool beside the unread one, since the old set added the read ones for that sender; the Intent does not state that.
The fresh `--from` step likewise tells `--from` from plain read only if the named sender is not the one with the oldest unread message, which the Intent does not say.
The step with `all` alone is fine: it separates new from old because read messages exist in the pool and the last read must return nothing.
The fix gives a complete sequence with distinct bodies, so the implementer cannot write a row that passes both semantics.
Class is `consistency` because the Intent disagrees with itself on what is unread at that step.

### [LOW] Three "the unread ones are reserved" sentences are left unnamed

**Location:** `_lyx/plan/17-creel-read-sets.md:22`

**Issue:** The Intent names the `Store.Read` first line, the `Request.All` comment, `readCmd`, and the `doc.go` first sentence as stale, and its closing grep covers only `still kept` and `still retained`.
Three more sentences describe a read that may hand out read messages: the `Reading` type comment (`internal/creelengine/verbs.go:121`, its unread ones are reserved), the second line of `Store.Read`'s comment (`verbs.go:131`) and the second sentence of the `doc.go` read paragraph (`doc.go:76`).
Once `all` no longer adds read messages every handed-out message is unread, so "the unread ones" now implies a read subset that does not exist.
`Reading` is also not among the card's `Edit` targets, so the comment sits on a symbol the card does not claim; the overlay run shows adding it is admissible.
The card's own stated check for stale wording misses these because it greps for two phrases only.
Class is `scope`: the card names some stale sites and misses others of the same kind.
