---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:53
    summary: D2 retries a daemon spawn for up to an hour with an elapsed-time cap only, which PATTERN-child-processes forbids
    fix: >-
      In D2's outbound worker bullet and its retry bullet (lines 53 and 56), state a cap on the COUNT of daemon starts per receiving hub for one pending message or one worker episode, a number the writer picks, after which the worker only reconnects and never starts the daemon again until a new post or a daemon restart of the sending hub.
      Name the cap beside the one-hour limit, and extend the PATTERN-child-processes Constraints bullet (line 172) from "logs the spawn and reaps it" to include "caps spawn attempts by count".
      Add a D2 acceptance criterion that a receiving hub whose daemon exits at start is started no more than that many times across the hour, each spawn logged and reaped, while the message still returns undelivered at the hour.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:94
    summary: The drop-notice bound is keyed on the claimed strand, which a forger chooses freely, and it hides every drop after the first in a window from the orch
    fix: >-
      Rewrite the D3 drop-notice bound (line 94) so the notices to the orch are bounded whatever claimed strands a forged flood names.
      Key the window on the sending hub alone, whose key space D1 limits to the registered hubs, or add a per-receiving-hub cap on notices per hour on top of the per-strand key, and state that the in-memory window map stays bounded in size.
      Also settle how a drop inside a window reaches the orch, since the operator's entry says the orch gets a notice of each drop and the current rule sends the count only with a later drop, which may never come.
      The writer picks a rule that meets both, such as one notice per drop up to a stated burst size per sending hub per hour and a counted notice after that, or states that a drop inside a window is seen only in the journal and in a later notice's count, and says why that meets the entry.
      Extend the D3 flood acceptance criterion (line 192) with a flood naming many different claimed strands for one sending hub, which yields at most the stated number of notices.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:73
    summary: A genuine message whose id and full tuple repeat a held message after a hub's ids restart is answered accepted and stored nowhere, so the sender journals it delivered
    fix: >-
      Add to the `deliver` request (D2 line 53), to the duplicate tuple of D3 steps 3 and 6, and to the confirm tuple of steps 4 and 5, a value no new incarnation of a sending hub repeats, such as the outbound message's journaled post time to the nanosecond, so a retry of one message carries the same value and a message of a re-created hub or of a hub whose scratch journal was removed does not.
      A held message with that hub path and id whose post time differs is then an id clash, refused as step 3 describes.
      Add an acceptance row to the D3 criterion at line 192 that a message with the same sender, receiver, reply-to id, expects-reply mark and body as a held one, from a sending hub whose ids restarted, is refused as an id clash and is never answered accepted without being stored.
      State in D3 the residual bound that remains, if any, for two messages that share the post time.
  - id: F4
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:142
    summary: The D7 bound for sessions launched before the deploy asserts how a session reads a stamp that contradicts its old trust sentence, which no test or text guarantees
    fix: >-
      Replace the sentence at lines 143 that says a session holding the old text still reads an untrusted orch of another hub as data with the exposure stated as fact.
      The old trust sentence in `internal/shuttleengine/claudeengine/settings.go` tells a session to follow mail from an orch strand as the operator's own instructions and names no hub, so for a session launched before the deploy the stamp wording alone argues against it.
      Name who is exposed, which includes every long-lived session such as the orch of a hub that lists no trusted hub, whose cross-hub mail is all stamped untrusted, and state the bound the design does choose.
      One such bound is a rollout step in D9's setup, and in the plan's deploy note, that every session of a hub able to receive cross-hub mail restarts after the deploy before any sandbox hub's orch may mail it.
      Keep the bound at lines 142 as a statement of what the old text lets through, and drop the claim that the stamp settles how the session reads it.
  - id: F5
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:57
    summary: The return notice relays the receiving daemon's reason text under the CREEL program sender with no stated bound on its content or length
    fix: >-
      In D2's retry and return bullets (lines 56 to 59), state that the `<shortname>:CREEL` notice names the receiver's reason either as a creel-authored code or as the remote answer's text cut to a stated length and quoted as data.
      The trust sentence has an agent follow a lym program's mail as the operator's own instruction, and a process that binds the receiving hub's socket while its daemon is down authors that text.
      Add the relay to D3's bound list at line 92 or give the notice its own bound line, so the decision that introduces the routing edge states what it lets through and what limits it.
---

### [BLOCKING] D2's daemon-start retry is capped by time alone

**Location:** `_lym/discussion/decision-record.md:53`, `:56`, `:172`
**Issue:** D2's outbound worker "starts that hub's daemon when none answers" on each delivery attempt, and every transient failure retries from one second doubling to a minute until the message is an hour old, so a receiving hub whose daemon exits at start is spawned about once a minute for an hour per pending message.
`PATTERN-child-processes` says a retry loop around a real spawn caps attempt COUNT, not only elapsed time, and the record's own Constraints bullet for that entry names only the spawn log and the reap.
`internal/creelcli/spawn.go` shows the client route the record points at starts the daemon once per client call and waits on its exit, so the cap is new text the design must carry for a worker that loops.
The count is a number the writer picks, so this is a design gap and no decision for the parent.

### [MEDIUM] Drop notices are keyed on a value the forger chooses

**Location:** `_lym/discussion/decision-record.md:94`, `:192`
**Issue:** The notice window is "per sending hub and claimed strand", the key round 2's F1 proposed, and step 1 of D3 accepts any well-formed strand name of a registered hub as the claimed sender, so a forged `deliver` flood that varies the claimed strand gets one notice and one window entry per request.
That floods the receiving hub's orch with mail from a `CREEL` program the orch follows, and the window map in daemon memory grows with the number of names; D3's flood bound states the journal has no cap but gives the notices a bound that this flood walks past.
In the other direction, a drop that falls inside a window sends nothing, and the count rides the first drop after the hour, which may never come, so a genuine message dropped in a burst never reaches the orch although the operator's entry says each drop gets a notice so that it is seen.
The acceptance criterion tests one claimed strand only.

### [MEDIUM] The duplicate tuple cannot tell a restarted id sequence from a retry when the messages are identical

**Location:** `_lym/discussion/decision-record.md:73`, `:78`, `:192`
**Issue:** Step 3 answers `accepted` with the held id, and stores nothing, when a held message has the same sending hub path and id and the same sender, receiver, reply-to id, expects-reply mark and body hash.
The Constraints say ids restart when a hub's scratch directory is removed or the hub is re-created at the same path, and held messages live seven days, so a repeated short message such as a status question from the same strand to the same receiver after such a restart matches a held message of the old incarnation.
The receiver then answers `accepted` without a confirm, the sender journals the message as delivered, and no agent ever sees it, which is the silent drop the operator's entry says must never happen.
The id clash rule covers only a tuple that differs, and the bound at line 95 repeats that limit.
Round 2's F6 raised the restarted-id case and its fix, the whole-tuple match, closes the differing messages; this finding is the identical-message remainder that fix leaves.

### [LOW] The D7 bound for pre-deploy sessions rests on a reading nothing guarantees

**Location:** `_lym/discussion/decision-record.md:142`
**Issue:** Round 2's F4 proposed the stamp wording that is now in the record, and the wording stays; what this finding asks for is only that the bound state the exposure as fact, which is why it is LOW.
`internal/shuttleengine/claudeengine/settings.go` renders the creel trust sentence into the session's append-system-prompt file at launch, and that sentence has the session follow mail from "an orch strand" as the operator's own instructions with no hub named.
The record says a session holding it "still reads an untrusted orch of another hub as data" because the stamp says so, which is a prediction about how a model weighs two contradicting texts, and the same line grants the trusted case follows the old sentence.
The exposed sessions are those that outlive the deploy, among them the orch of a hub that lists no trusted hub, where every cross-hub message is stamped untrusted and a sandbox hub's orch is the sender the design is built to contain.
The restarted daemon delivers cross-hub mail to them as soon as the binary lands, so the bound as written does not state what the old text lets through.

### [LOW] The return notice carries remote-authored text with program authority

**Location:** `_lym/discussion/decision-record.md:57`, `:59`
**Issue:** The return notice names "the reason", which for a `refused` answer or the last transient answer is text the receiving daemon sent, and the notice reaches the sender's agent as mail from `<shortname>:CREEL`.
The trust sentence has an agent follow a lym program's mail as the operator's own instruction, and a process that binds the receiving hub's socket while its daemon is down, which D3's bound at line 92 already leaves open, writes that text with no length limit.
The Decision that introduces the routing edge states no bound on what the relayed reason may hold.

## Rejected

- Focus 1, whether D3 states the journal entry, the notice and the receiving orch: it does; step 6 names the drop event's fields and the notice to `<shortname>:orch` with the hub's own shortname, and the criterion at line 192 tests the journal entry and the notice; only the flood bound fails, as F2 records.
- Focus 2, whether `returned` and `unanswered` need the trust mark: they do not; `returned` holds the outbound message whose sender is the local strand, and an outbound message is held by no strand, so no chase or settle produces `unanswered` for it; D6 covers the hook line, the reminder, the `read` text and JSON and the `reply` field, and `lym creel inbox` prints sender names and counts with no body.
- Focus 3, whether a sender can journal a return for a message the receiver holds: it can only when the last `deliver` call before the return finds the receiver unreachable, and D2's closing bound states that case.
- Focus 4, how an older binary renders inbound cross-hub mail: it renders none, since D3 step 6 journals a new event and `internal/creelengine/fold.go` ignores an unknown event but for its id; D2's downgrade bound states that the mail reaches no agent until a binary carrying this task runs again.
- Focus 5, whether a reply to a message from another hub is measured stamped at `--reply`: D4's first bullet says so and the D4 criterion tests it, in contrast to the unstamped `validateLine` call `Store.Reply` makes today; the plan writer changes that call for a reply to a cross-hub message, and the record needs no further text.
