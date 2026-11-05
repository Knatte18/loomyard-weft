---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/09-slackengine-relay.md:55
    summary: "The restart cap resets StartFailures at the relay's first accepted read, so a relay that dies after its first read is respawned every minute with no cap on the count, against PATTERN-child-processes' rule that a retry loop around a spawn caps attempt count."
    fix: "In _lym/plan/09-slackengine-relay.md judge a relay by how long it lived, not by whether it reached its first read. Declare one named constant `relayHealthyLifetime` (10 minutes) in start.go. When a due whileAway start finds the relay down and a LastSpawnAt stands, read the last relay's lifetime as RelayState.LastSlackOK minus LastSpawnAt, where a zero LastSlackOK or one before LastSpawnAt reads as zero (a live relay writes LastSlackOK at least once a minute). A lifetime under the constant counts one failure; a lifetime at or over it sets StartFailures to 0 and counts none. Step 4 keeps writing FirstReadAt, which AwaitFirstRead reads, but no longer sets StartFailures to 0: change the sentence 'The relay's accepted first read resets the count (step 4)', step 4's text, and the TestRunRelay row 'a first read accepted with StartFailures standing at 4 writes FirstReadAt and StartFailures 0' so the row asserts StartFailures is kept. Replace the TestEnsureRelay row 'a relay with four counted failures whose next spawn reaches an accepted first read ...' with a row where the fake relay writes LastSlackOK eleven minutes after LastSpawnAt and then goes down, and the next due whileAway start spawns and leaves StartFailures at 0; add a row where the fake relay writes FirstReadAt and a LastSlackOK one minute after LastSpawnAt and then dies, repeated over five due starts, and the sixth start answers gave-up. Any other design that caps the spawn count of a relay that dies after its first read is acceptable. No target or verify line changes, so no overlay run is needed."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/plan/07-slackengine-inbound.md:42
    summary: "Card 07 says the poll calls Hub.Beams only for a text opening with an at-sign slug, but an away or back text reaches Hub.Beams through SetPresence's beams argument, and no outcome is assigned to a SetPresence error, so the cursor move and the Slack answer for that message are open."
    fix: "In _lym/plan/07-slackengine-inbound.md change the sentence 'The poll calls `Hub.Beams` only for a text opening `@<slug> `, and a `Hub.Beams` error comes before any post...' to say the poll calls Hub.Beams for a text opening `@<slug> ` and, as SetPresence's beams argument, for an `away` or `back` that changes the presence; the pre-post rule stays for the routing call. Under the presence-command bullet add: SetPresence returns changed true with an error when the write stood and only the beams read or a notice failed; the poll then logs the error at Warn once per episode, still answers with the resulting state, and counts the message handled so the cursor moves, since a retry finds no change and sends no notice. SetPresence returns changed false with an error when the write itself failed; the poll then returns the error with the cursor left before the message and the relay retries with its backoff, which duplicates nothing because no notice or answer was sent. Say the same of `status` when ReadPresence errors: the poll returns it with the cursor left. Add two TestInboundPoll rows, an `away` whose Hub.Beams errors after the write (presence slack, one answer, cursor moved, a second poll forwards nothing) and an `away` whose presence write fails (no answer, cursor left, handled once the write works), and one TestSetPresence row, a beams error after the write returns changed true and the error. No target or verify line changes, so no overlay run is needed."
  - id: F3
    severity: LOW
    class: scope
    location: _lym/plan/07-slackengine-inbound.md:57
    summary: "The prescribed branch where a Hub.Beams error returns with the cursor left before the message has no TestInboundPoll row that reaches it; the rows reach the unknown-slug answer and the Paused error only."
    fix: "In _lym/plan/07-slackengine-inbound.md add to the TestInboundPoll rows: a `@payments x` text whose Hub.Beams returns an error posts nothing to the mailbox, sends no Slack answer, returns the error and leaves the cursor before the message, and the poll after Hub.Beams recovers forwards it once as `[slack] x` to `<shortname>:payments:orch`. No target or verify line changes, so no overlay run is needed."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/09-slackengine-relay.md:73
    summary: "The wait 'doubled for each counted failure up to 30 minutes' names a clamp that no input reaches: a spawn happens only with at most four counted failures, so the wait is 1, 2, 4, 8 or 16 minutes and never 30."
    fix: "In _lym/plan/09-slackengine-relay.md drop 'up to 30 minutes' from the NextStartAt sentence and write 'doubled for each counted failure' with the values 1, 2, 4, 8 and 16 minutes before the fifth failure ends in gave-up; if F1's redesign changes the count's reach, drop or keep the clamp to match what its rows reach. No target or verify line changes, so no overlay run is needed."
  - id: F5
    severity: LOW
    class: scope
    location: _lym/plan/09-slackengine-relay.md:72
    summary: "Three edges of the cap are unassigned: the StartOutcome of a whileAway start before NextStartAt is unnamed, FirstReadAt equal to LastSpawnAt is neither accepted nor failed, and no TestEnsureRelay row pins that LastSpawnAt and NextStartAt are stored before spawn runs."
    fix: "In _lym/plan/09-slackengine-relay.md name the outcome of a whileAway start before NextStartAt (for example `waiting`, which card 12 treats like any outcome but gave-up and needs no change for), state that a spawn counts as accepted when FirstReadAt is not before LastSpawnAt (or, after F1, the lifetime rule's own boundary), and extend the TestEnsureRelay row 'a down relay answers started with one spawn ... and records LastSpawnAt' so the fake spawn reads RelayState when called and asserts LastSpawnAt and NextStartAt are already stored. The rows that write FirstReadAt from a fake spawn then advance the fake clock before writing it, so equality is never what they rely on. No target or verify line changes, so no overlay run is needed."
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/03-creel-seams.md:38
    summary: "Card 03 says a refused or lost confirmation makes ReadWait return its error, but Client.CallWatching resumes a Wait read after a lost connection by sending the read again, so only a refused confirmation, the told timeout or exhausted reconnects return an error and the ReadWait doc written from this text would be false."
    fix: "In _lym/plan/03-creel-seams.md reword the ReadWait bullet: it returns nil once the daemon confirmed the acknowledgement of the read it last made; a connection lost before or after the handler ran sends the read again, up to MaxReconnects, so the handler can run a second time for mail it already saw; a refused confirmation, the told timeout and exhausted reconnects return their error. Keep the sentence that its timeout spans the wait, the handler and the confirmation, and say card 8's outbox record is what makes a second handler run post nothing twice. The ReadWait doc comment and the creelcli doc.go sentence take the same wording. No target or verify line changes, so no overlay run is needed."
  - id: F7
    severity: LOW
    class: consistency
    location: _lym/plan/10-slack-cli.md:69
    summary: "Card 10 names the setup verb as `setup --channel <id> --operator <user id>` and the relay verb as `relay ... [--fork | --after <pid>]`; if either is used as the cobra Use string, TestCLIForm_EveryCommand's Use check rejects the `<user id>` token, the `[--fork` token and the unmarked flags."
    fix: "In _lym/plan/10-slack-cli.md state the two Use strings the form check admits: `setup --channel <id> --operator <user-id>` with both flags marked required, and `relay --hub-path <hub> --prime <name>` with `--hub-path` and `--prime` marked required and `--fork` and `--after` declared as optional flags that the Use string does not list. No target or verify line changes, so no overlay run is needed."
---

### [BLOCKING] The restart cap does not cap a relay that dies after its first read

**Location:** `_lym/plan/09-slackengine-relay.md` step 4 (line 55) and the `starter.ensure` paragraphs (lines 69-77).

**Issue:** `PATTERN-child-processes`'s background file says "A retry loop around a real spawn caps attempt COUNT, not only elapsed time", and card 09 cites it for the prime dobby tick's restarts.
The card caps only failures in a row that never reached an accepted first read: step 4 sets `StartFailures` to 0 at that read, and the judging rule (lines 72 and 76) counts no failure when `FirstReadAt` is later than `LastSpawnAt`.
Trace: a tick spawns at T0 and writes `LastSpawnAt` T0 and `NextStartAt` T0+1 minute; the relay's first read is accepted at T0+3s, which writes `FirstReadAt` and `StartFailures` 0; the relay then dies at T0+10s, for instance on a panic while handling one message.
The next due tick finds the relay down, finds `FirstReadAt` later than `LastSpawnAt`, counts no failure and spawns again; `StartFailures` stays 0, so the cycle repeats once a minute with no end and no `gave-up`.
The first read is the relay's first act, before any Slack call, so it proves creel accepts `<shortname>:SLACK`, not that the relay stays up.
The rule counts the spawn attempts of an unbounded loop only while they fail early, so the count is not capped.
Paths in focus entry 1, as settled: the spawn that errors before any relay exists (first start writes `LastSpawnAt`, returns the error; starts 2 to 5 each count one failure and call spawn; the sixth answers `gave-up`, matching the named row), and the writers (the starter writes `LastSpawnAt`, `NextStartAt` and the increment, the relay's step 4 writes the reset) are assigned and each reached by a row; the defect is in what the reset condition lets through.
The fix judges a relay by its lifetime, derived from `LastSlackOK`, and moves the reset to the starter's judgment; any design that caps the count across a post-first-read death meets the finding.
The round-2 fixer put the reset in step 4 so that a healthy relay's re-exec, which zeroes `FirstReadAt` at step 3, cannot leave a stale count behind; the lifetime rule keeps that, since a re-exec changes no `LastSpawnAt`, so a relay that lived long before it died is judged long-lived whatever its last image did.

### [MEDIUM] SetPresence errors and the Hub.Beams call behind them have no assigned outcome

**Location:** `_lym/plan/07-slackengine-inbound.md` step 3, the Hub.Beams sentence (line 42) and the presence-command bullet (line 40).

**Issue:** Card 07 says "The poll calls `Hub.Beams` only for a text opening `@<slug> `".
The same card declares `SetPresence(geom, mailbox, beams func() ([]string, error), presence, now)`, which posts a notice to every beam's orch `beams` lists, and the poll's `away` and `back` handling calls it; the only beam source the poll has is `Hub.Beams`.
So a `Hub.Beams` error can arrive through `SetPresence` after the presence write stood, and the card says nothing about that case: it is neither the pre-post error (cursor left) nor the post-post error (cursor moved).
`SetPresence`'s own contract, "a failed notice is joined into the returned error after the write stands", leaves the poll without a rule for the cursor and for the Slack answer to the operator's `away`.
Leaving the cursor retries a command whose write already stood and whose retry sends no notice; moving it with no answer drops the operator's reply; the card must pick one per error shape.
`status` has the same gap when `ReadPresence` errors on an unknown stored value (card 04).
The recorded decision for `Hub.Paused` and `Hub.Beams` errors is carried as written; this finding is the third error site the decision did not name.
`SetPresence` returns `(bool, error)`, so `changed` already tells a failed write (false) from a stood write with a failed notice (true), which is the distinction the fix assigns outcomes by.

### [LOW] No test row reaches the Hub.Beams error branch

**Location:** `_lym/plan/07-slackengine-inbound.md`, the `TestInboundPoll` rows (line 57).

**Issue:** The decision record's added entry prescribes that a `Hub.Beams` error for an at-sign text returns with the cursor left before the message and retries with the backoff, and card 07 repeats it as a guard branch.
The `TestInboundPoll` rows reach the unknown-slug answer (`@nope x`) and the `Hub.Paused` error, and none makes `Hub.Beams` return an error.
The branch is a prescribed guard with no named test that reaches it; the decision's own one-row addition named only the Paused row, so this is a gap the plan can close with one row of the same test.

### [LOW] The 30-minute clamp on the restart wait is unreachable

**Location:** `_lym/plan/09-slackengine-relay.md`, the `NextStartAt` sentence (line 73).

**Issue:** The sentence sets `NextStartAt` to the spawn time plus one minute "doubled for each counted failure up to 30 minutes", and the same card ends the loop at the fifth counted failure.
A spawn happens only after at most four counted failures, so the wait is 1, 2, 4, 8 or 16 minutes; the 30-minute bound never applies and no row reaches it.
A bound no input reaches is code for a state the plan's own cap rules out.

### [LOW] Three edges of the cap are unassigned

**Location:** `_lym/plan/09-slackengine-relay.md`, lines 70-76 and the `TestEnsureRelay` rows (line 87).

**Issue:** First, a `whileAway` start before `NextStartAt` "spawns nothing and counts nothing" but the card names no `StartOutcome` for it; card 12 handles only `gave-up`, so an implementer must invent a value, and `started` would be false.
Second, the judging rule counts a failure when `FirstReadAt` is "zero or earlier" than `LastSpawnAt` and counts none when it is "later"; equality is in neither branch.
The row where a fake spawn writes `FirstReadAt` as step 4 would runs on a fake clock the card does not say advances, so equality is the likely value the row meets.
Third, the card says `LastSpawnAt` and `NextStartAt` are written "before it calls `spawn`" so a fast relay's `FirstReadAt` never lands before its own `LastSpawnAt`; no row has the fake spawn observe the stored state when it is called, so a starter that writes after the spawn passes every named row.

### [LOW] ReadWait's lost-confirmation contract contradicts Client.CallWatching

**Location:** `_lym/plan/03-creel-seams.md`, the `GoClient.ReadWait` bullet (line 38) and the doc-text paragraph (line 50).

**Issue:** The card says `ReadWait` "returns nil only once the daemon confirmed the acknowledgement, and a refused or lost confirmation returns its error".
In `internal/creelengine/client.go`, `exchange` returns a `connLostError` when the confirmation line is lost (`awaitAckConfirmation`) or the acknowledgement cannot be written (`ErrAckUnsent`), and `CallWatching` (lines 160-178) resumes any `connLostError` of a request with `Wait` set by calling `resume`, which for `VerbRead` returns the same request, up to `MaxReconnects`.
So a lost connection after the handler ran sends the read again, and the handler can run a second time for mail it already saw; only a refused confirmation, the told timeout and exhausted reconnects return an error.
The `ReadWait` doc comment and the `creelcli/doc.go` sentence the card plans from this text would be false.
The behavior is safe for card 08, since its outbox record skips ids already posted, but the contract text says something else.
Focus entry 2, settled: every `creelcli` and `creelengine` text the card lists (both `doc.go` files, `goCallTimeout`, `resolveSender`) sits under `**Edit:**`, none under `**Extends:**`, and no other sentence in those two packages is made false by `ReadWait` or `HubStrands`.

### [LOW] Card 10's verb forms fail the Use check if copied into Use

**Location:** `_lym/plan/10-slack-cli.md`, the `setup` and `relay` verb bullets (lines 69 and 75).

**Issue:** `TestCLIForm_EveryCommand` walks every command of `newRoot()` including hidden ones and applies `useProblems`: each Use token after the name must be `<arg>`, `[<arg>]`, `<arg>...`, `[<arg>...]`, a `{...}` span or a `--flag <value>` pair whose flag the command marks required.
Card 10 writes `setup --channel <id> --operator <user id>`, whose `<user id>` has a space and fails the argument form, and `relay --hub-path <hub> --prime <name> [--fork | --after <pid>]`, whose `[--fork` and `|` tokens fail it.
The card never states the Use strings, so an implementer who copies the verb headings meets the failure at the card's own gate.
Focus entry 4, settled otherwise: the texts `clitree_test.go` checks are carried by card targets, with `Short`, `Use`, `Long` and the audience on `slackcli#Command` and `slackCLI.command`, and the root `Long` list and the mount on the `cmd/lym#newRoot` `Edit`; `registration_test.go` needs only the mount, which `newRoot` carries.

## Rejected

- Card 07's `Hub.Paused` error outcome (logged at Warn once per episode, no paused answer, cursor moved, one `TestInboundPoll` row) matches the decision record's added entry and is reached by a named row; not a defect.
- Card 07's `Hub.Beams` error for an at-sign text (cursor left before the message, retried with the backoff) matches the decision record's added entry; only its missing row is raised (F3).
- Card 09's spawn-error path before any relay exists, and the writer of each `RelayState` field named in the cap, are assigned and reached by the `TestEnsureRelay` row of six starts over a fake spawn that errors; not a defect.
- Card 03 carries every false-by-`ReadWait` text under `**Edit:**` rather than `**Extends:**`; not a defect (F6 concerns the contract wording only).
- A lasting `Hub.Beams` error holding the cursor with no bound was deferred in round 2 because it is the parent's recorded decision; not re-raised here, and F2 and F3 leave that decision as written.
- The root `Long` list in `cmd/lym#newRoot` is carried by the card's `Edit` of that glyph, and the root `Long` stays under the 2000-byte cap with one more name; not a defect.
