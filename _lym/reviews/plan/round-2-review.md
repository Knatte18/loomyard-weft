---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/09-slackengine-relay.md:69
    summary: >-
      The whileAway restart cap does not say when the starter writes LastSpawnAt and NextStartAt, nor who resets StartFailures, so a spawn that fails to start is retried on every tick with no count cap, against PATTERN-child-processes
    fix: >-
      In card 09, in the bullet that begins "The tick's restarts are capped", state that starter.ensure writes LastSpawnAt (the spawn time) and NextStartAt in its RelayState read-modify-write before it calls spawn, and returns the spawn's error after that write.
      A spawn that returns an error then leaves a LastSpawnAt with no accepted first read after it, so the next due whileAway start counts it as one failure, and five such starts end in gave-up.
      In the same bullet give the reset to the starter and drop "Any relay's accepted first read resets StartFailures and NextStartAt" as written:
      when a due whileAway start finds a FirstReadAt later than LastSpawnAt, it sets StartFailures to 0 in the same read-modify-write, counts no failure and spawns;
      EnsureRelay without whileAway still resets the cap before it spawns.
      In the TestEnsureRelay rows add two rows.
      First, a fake spawn that returns an error: five whileAway starts, the clock moved past each wait, call it exactly five times and the fifth answers gave-up, with NextStartAt rising between them.
      Second, a relay with four counted failures whose next spawn reaches an accepted first read and then goes down: the following whileAway start spawns and counts no failure, and StartFailures is 0 after it.
      No target changes, so no overlay check applies.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/03-creel-seams.md:19
    summary: >-
      Card 03 lists internal/creelcli/doc.go under Extends, but ReadWait takes a caller-told timeout and so makes the sentence "Every call passes a 30-second timeout" false, as it does the goCallTimeout comment
    fix: >-
      In card 03 move internal/creelcli/doc.go from the Extends group to the Edit group, delete the then-empty Extends group, and add internal/creelcli#goCallTimeout to the Edit group.
      In the Intent, where the card says internal/creelcli/doc.go gains a sentence naming ReadWait and HubStrands, add that the doc's sentence "Every call passes a 30-second timeout, so a caller under a lock waits at most that long per call" is reworded to exempt ReadWait, whose timeout the caller tells, and that the doc comment of goCallTimeout in internal/creelcli/goclient.go, "bounds every request a GoClient makes", is reworded the same way.
      Checked with lym loom validate-plan --overlay: the baseline run and a run with this change alone both answer ok with no blocking finding.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lym/plan/07-slackengine-inbound.md:44
    summary: >-
      Card 07 leaves open what the poll does when Hub.Paused returns an error after the creel post succeeded, so a policy that keeps the cursor before the message forwards it again on every retry
    fix: >-
      In card 07, in the step 3 bullet that begins "After a successful post", add that a Hub.Paused error is logged at Warn once per episode, sends no paused answer, and still counts the message as handled so the cursor moves past it, because the post already reached creel and a retry would forward it again.
      In the same step name what a Hub.Beams error does for a text that opens with an at-sign slug: it is read before any post, so the poll returns it with the cursor left before the message and retries with the backoff.
      In the TestInboundPoll rows add one row: a Hub whose Paused returns an error after an accepted post forwards the message once, logs the error, sends no answer and moves the cursor, and a second poll forwards nothing.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/10-slack-cli.md:95
    summary: >-
      Card 10 says no list in clitree_test.go changes but never says that newRoot's Long module list must name slack, which that test requires
    fix: >-
      In card 10's Intent, in the sentence that begins "cmd/lym/clitree_test.go and registration_test.go then hold", add that newRoot's Long text, whose "Available modules" list clitree_test.go checks for every mounted module, gains slack beside creel.
      No target changes, since cmd/lym#newRoot is already an Edit target.
---

### [BLOCKING] The restart cap leaves the failing-spawn path and the reset unassigned

**Location:** `_lym/plan/09-slackengine-relay.md:69`
**Issue:**
The focus asked whether the cap kept in `StartFailures` and `NextStartAt` survives the relay's rewrite at step 3, and whether a relay that dies before step 3 still counts.
Both hold as written: step 3 is a read-modify-write that keeps the starter's fields, `internal/state` has `UpdateJSON` for it, and the last `TestEnsureRelay` row covers a relay that dies before step 3 with an earlier `FirstReadAt` standing.
Every field card 09 changes belongs to `RelayState`, which card 04 creates.
What the card leaves open is the order of the starter's own writes.
It says `ensure` "records `LastSpawnAt` whenever it spawns" and that "every spawn sets" `NextStartAt`, and the failure is only counted when the next due start judges the previous spawn from `LastSpawnAt`.
`spawn` can return an error before any relay exists, for example when `cmd.Start` fails because the lym binary was replaced mid-deploy.
If the fields are written after a successful spawn, that path writes nothing, and every prime dobby tick retries the spawn at once, with no count and no backoff.
That is a retry loop around a real spawn that caps neither count nor rate, which `PATTERN-child-processes` forbids.
Writing `LastSpawnAt` after the spawn also lets a fast relay's `FirstReadAt` land before it, which then counts a healthy relay as failed.
The reset has the same gap.
"Any relay's accepted first read resets `StartFailures` and `NextStartAt`" names no writer: step 3 keeps them "as they stand" and step 4 writes only `FirstReadAt`.
Without a reset, five failures that are not in a row still end in `gave-up`.
No `TestEnsureRelay` row reaches a failing spawn or the reset.

### [MEDIUM] `internal/creelcli/doc.go` is extended where one of its sentences turns false

**Location:** `_lym/plan/03-creel-seams.md:19`
**Issue:**
The focus asked about `internal/creelengine/doc.go`.
Card 03 now lists it under **Edit:**, and its Refusal code section is rewritten, not extended, so that one is correct.
The same check on `internal/creelcli/doc.go`, still under **Extends:**, finds a contradiction.
That doc says "Every call passes a 30-second timeout, so a caller under a lock waits at most that long per call".
`ReadWait(timeout, handle)` takes the caller's timeout, so the sentence is false once the card lands, and so is the comment on `goCallTimeout` in `goclient.go`, "bounds every request a GoClient makes".
`Extends` is for adding to a ref without changing what is there.

### [MEDIUM] Card 07 leaves a `Hub.Paused` error after a successful post unspecified

**Location:** `_lym/plan/07-slackengine-inbound.md:44`
**Issue:**
The poll reads `Hub.Paused` only after the post succeeded, and `Hub.Paused` returns an error.
`pauserecord.ReadCoverage` returns an error for an unreadable record, so the case is reachable.
The card gives that error no outcome.
If an implementation returns it with the cursor left before the message, each retry forwards the same message to the orch again until the record reads.
D4 asks for each message to be handled once.

### [LOW] Card 10 does not carry the `newRoot` `Long` edit that `clitree_test.go` requires

**Location:** `_lym/plan/10-slack-cli.md:95`
**Issue:**
`cmd/lym/clitree_test.go` fails when `root.Long` does not name a mounted module, and the list sits in `newRoot`'s `Long`.
The card's sentence "no list in them changes" is true of the test's own lists, but a reader takes it to mean the mount needs no text edit.

## Rejected

- Whether `RelayState` fields survive the relay's step 3: they do, as above; not a defect.
- Whether a relay that dies before step 3 counts a failure: it does, and a row covers it; not a defect beyond F1.
- Whether `AwaitFirstRead` acts on a gone relay's `FirstReadAt` or refusal: the check is the recorded pid plus start time against a running process, a gone relay reads as not yet written, and the `TestRelayWaits` stale-refusal row reaches it; not a defect.
- `PATTERN-daemon-wakeups`' keep-warm cap against the constant 30-second bound `lym slack away` gives an orch: raised in my first pass and withdrawn once round 1's review was read, because D6 fixes 30 seconds and that bound lies under any cap; no new evidence turned up.
- Where `relay --hub-path` gets the prime location with the hub directory as working directory: card 10 gives `relay` no `lymcwd.Resolve` and builds the prime through `lymcwd.NamedWorktree(hub, name)`, which applies no cwd gate and spawns no git; `NewProgramClient`, `DobbyStart`'s `Beams` and `PauseGeometry` take only the location's hub path, worktree name and anchor; not a defect.
- Whether card 10's `Intent` states what a by-hand `relay --fork` lets through and its bound: it names any pane and off Linux, and bounds it by the configured-hub check in the relay, the absence of presence, token and setup writes, and creel's environment-only identity off Linux; not a defect.
- Whether cards 11, 12 and 13 list a `doc.go`, help text or `docs/overview.md` line under **Uses:** that their change falsifies: none lists one there, and each such text they change is under **Edit:**; not a defect.
