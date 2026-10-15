---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-creelengine-delivery.md:17
    summary: Delivery worker backs off only while busy, held or paused, so a dormant receiver or an idle one inside its reminder backoff is probed every second for as long as mail stays unread, against PATTERN-daemon-wakeups
    fix: >-
      In card 9 replace the sentence "A worker probes `Delivery.Idle` from a 1-second base, doubling to 8 seconds while the receiver reads busy, held or paused, and returns to the base on a new post, inject or release;"
      with "A worker wakes from a 1-second base and doubles its wait to an 8-second ceiling after every wake that delivers nothing (the receiver reads busy, held, paused or dormant, or reads idle while its reminder waits out the backoff), and returns to the base on a new post, inject or release or on a wake that delivers;",
      and add a half sentence that this departs from D19's list of busy, held or paused because PATTERN-daemon-wakeups wins, as the sweep sentence already does for D15.
      In the same card's What-counts sentence for `pattern/PATTERN-daemon-wakeups.md` change "creel's delivery workers (1 second doubling to 8 seconds)" to "creel's delivery workers (from 1 second, doubling to 8 seconds after every wake that delivers nothing)".
      In `TestScheduler_Reminders` extend the row that reads the probe interval off the fake clock (1, 2, 4, 8, 8 seconds, back to 1 after a new post) so it also asserts the same readings for a dormant receiver with unread mail and for an idle receiver whose reminder is inside its backoff, and 1 second again after a wake that delivers.
      The change is prose and test-row text only; it adds, drops or moves no target and changes no verify command, so no validate-plan overlay run applies.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-creelengine-delivery.md:29
    summary: Card 9 adds pause and hold refusals to the interrupt forms that card 7 creates in verbs.go, yet lists no Edit target for them, and card 7 names card 9 as the completer of its own Interrupt and Inject
    fix: >-
      Make card 7 own the whole interrupt check chain, since everything it needs is already in card 7's seam and Store (`Delivery.Paused`, `Delivery.Interrupt`, the hold records).
      In card 7 change the `Inject` sentence ending "card 9 adds the pause and hold refusals of the `interrupt` form and its interrupt key" and the `Interrupt` sentence "the privilege and orch checks below run here before card 9's pause and hold refusals and the guarded interrupt" so that `Store.Interrupt` and `Store.Inject` with the interrupt flag run, in order, the privilege check, the orch refusal, the pause refusal, the hold refusal (naming owner and expiry, the owner's own call passing) and then `Delivery.Interrupt`, reporting `ErrNoRunningTurn` as nothing sent; `inject --interrupt` then queues its text at the head.
      In card 9 delete the paragraph beginning "Both interrupt forms (D10)" down to "queues its text at the head", keeping only that the head-queued text is typed at the next idle.
      Move the `TestScheduler_Injects` rows "interrupt and inject --interrupt each refused under a pause and refused from a non-owner under a hold naming owner and expiry, with nothing queued, interrupted or typed", "the hold owner's interrupt passing under its own hold" and "interrupt reported nothing-sent on ErrNoRunningTurn" into `TestStore_InjectAndHold` in card 7, leaving "inject --interrupt interrupts then types its text first at the next idle" in card 9.
      Test counts and the budget rows are unchanged, and no target is added, dropped or moved, so no validate-plan overlay run applies.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-creelengine-pool-verbs.md:28
    summary: A reply to an orch strand is mail that grows its unread set but journals no withheld event, so log and the fold miss it while D16 has log show every orch reminder as withheld
    fix: >-
      In card 7's `Reply` sentence add that a reply whose receiver is an orch strand (`agentname.MatchesRole(role, agentname.RoleOrch)`) journals one `withheld` event naming `creel-notices`, as `Post` and `Forward` do.
      In card 9 change the parenthesis "(a post, forward or return to it)" to "(a post, reply, forward or return to it)" and "card 7's post and forward write theirs" to "card 7's post, reply and forward write theirs".
      In card 7's `TestStore_Mail` add to the reply row a reply to an orch message that log shows as one withheld line.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-creelengine-pool-verbs.md:40
    summary: The privileged-inject Bound says the TTL bounds how stale an inject lands, but the TTL does not run while the receiver is dormant or paused, and the Bound is silent on what interrupt can do
    fix: >-
      Rewrite card 7's first "Bound:" paragraph so it is true to D9 and names every bound.
      It keeps that a privileged sender may type any single-line text without a control character, slash commands included, into any non-orch strand's pane at its next idle, and that the interrupt forms stop a running turn with shuttle's guarded Escape.
      It then says that the TTL (10 minutes by default, 1 hour at most) bounds staleness only while the receiver is live and unpaused, so an inject queued for a dormant strand waits untyped until the strand is live and idle, and expires when the removed-holder sweep finds the strand gone from every reed state;
      and that what holds the typing back is the pause, hold and orch rules of cards 7 and 9, the pre-write pane guard of card 5 and the refusal of a non-privileged sender.
      Mirror the dormant sentence in the `doc.go` text of card 16 only if card 16 restates the bound.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/14-creelcli-verbs.md:38
    summary: TestCreelCLI sets the claim's LYX_STRAND_NAME through the process environment, since the package seam holds only clock and spawn, but names neither the non-parallel state nor the cleared variable its EXTERNAL rows need off Linux
    fix: >-
      In card 14's `TestCreelCLI` paragraph state that the strand rows set `LYX_STRAND_NAME` with `t.Setenv` and every EXTERNAL row sets it to the empty string with `t.Setenv`, so no row inherits the name of the strand an agent runs the gate from;
      off Linux the environment is the whole identity, so an inherited name of the hub's shortname would make an EXTERNAL row resolve as that strand.
      Add that `TestCreelCLI` therefore does not call `t.Parallel` and carries a comment naming the environment as the process-global state, as `PATTERN-test-speed` asks.
      With this the strand and EXTERNAL rows hold on a platform where `peer_other.go` reports no peer.
      Card 10 needs no change, since its tests build the `Claim` field directly.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/07-creelengine-pool-verbs.md:42
    summary: Card 9 reads the sweep and probe waits off the fake clock, but card 7 describes creelfake.Clock only as advancing when told, with no record of the durations asked of it
    fix: >-
      In card 7's `creelfake` sentence change "`Clock`, a fake clock that advances only when told" to "`Clock`, a fake clock that advances only when told and records every duration passed to `After`, in order, for a test to read",
      so card 9's `TestScheduler_Reminders` and `TestSweep_RemovedHolders` waits (1, 2, 4, 8, 8 seconds and 30 s up to the 4-minute ceiling) can be read from the kit card 7 creates.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/07-creelengine-pool-verbs.md:43
    summary: Tests over creelfake are planned in the creelengine directory, but creelfake imports creelengine, so they compile only as an external test package, which no card says
    fix: >-
      In cards 7, 9 and 10 state that `verbs_test.go`, `deliver_test.go` and `server_test.go` are `package creelengine_test` files, which use exported API only, while `journal_test.go`, `identity_test.go` and `testmain_test.go` stay in package `creelengine` (they reach unexported code and use no kit that imports creelengine).
      An in-package test importing `creelfake` is an import cycle, the one `internal/testkit/shuttlefake`'s header records for `internal/shuttleengine`.
      No target changes, so no validate-plan overlay run applies.
---

### [BLOCKING] Delivery worker never backs off for a dormant receiver or an idle one inside its reminder backoff

**Location:** `_lyx/plan/09-creelengine-delivery.md:17`, with `_lyx/plan/09-creelengine-delivery.md:34` and `_lyx/plan/09-creelengine-delivery.md:35`

**Issue:** `PATTERN-daemon-wakeups` says a wake that spawns a process or captures a pane doubles its wait from a base to a named ceiling while nothing changes.
Card 9's worker doubles from 1 to 8 seconds only "while the receiver reads busy, held or paused", which is D19's list and which the card copies.
Card 9 also has the worker learn liveness from `Strands` (line 18, "delivery resumes when `Strands` reports it live"), the only seam call that reports `Live`, and card 13 builds `Strands` from `git worktree list` plus a reed `Status` per worktree, so each such wake is a git spawn plus a tmux query.
Two states a worker sits in for hours fall outside the list.
A dormant receiver with unread mail has a worker, because "a receiver with nothing pending is not probed" does not exempt unread mail.
An idle receiver whose reminder went out and whose unread set is unchanged still has unread mail, so it has a worker too, and the card sends nothing until the 5-minute to 1-hour backoff ends.
Both loops wake at the 1-second base with nothing changing, for the daemon's life.
D19 and `PATTERN-daemon-wakeups` conflict here, and the constraint wins, as the card already says for the sweep against D15.
The What-counts line and the `TestScheduler_Reminders` waits row repeat the narrow list, so both need the same correction.

### [MEDIUM] Interrupt checks are split across cards 7 and 9 with no Edit target for the second half

**Location:** `_lyx/plan/09-creelengine-delivery.md:29`, with `_lyx/plan/07-creelengine-pool-verbs.md:35` and `_lyx/plan/07-creelengine-pool-verbs.md:36`

**Issue:** Card 7 creates `Store.Inject` and `Store.Interrupt` in `verbs.go` and says card 9 adds the pause and hold refusals and "its interrupt key" to them.
Card 9 lists only new files under Create and no `verbs.go` or `Store` member under Edit, so the card that changes `Store`'s interrupt behavior never names it.
Card 10's server dispatches to the `Store` verb, so the interrupt verbs are complete only once card 9 has patched a file it does not list.
Everything the refusals need (`Delivery.Paused`, `Delivery.Interrupt`, holds) is already in card 7's seam and `Store`, so card 7 can own the chain and card 9 keeps the worker.
"Its interrupt key" is also undefined in both cards; the fix drops it.

### [MEDIUM] A reply to an orch strand journals no withheld event

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md:28`

**Issue:** Card 7 journals `withheld` for "a post or forward to an orch receiver", and card 9 lists the changes of an orch's unread set as "a post, forward or return to it".
A reply is mail ("to a strand or program it is mail") and replies are how a driver or agent answers an orch's post, so an orch's unread set grows without the event.
D16 has `log` and the journal show an orch's reminders as withheld, and card 7 builds `Log`'s withheld view from those events, so such a reply would be unreminded and invisible in `log`.
Card 6's event list and fold already carry `withheld`; only the writer in `Reply` and the two enumerations are missing.

### [MEDIUM] The privileged-inject Bound overstates what the TTL bounds

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md:40`

**Issue:** The Bound states what a privileged inject can type and into which panes: any single-line text, slash commands included, into any non-orch strand's pane at its next idle.
It then says the TTL "bounds how stale it lands".
D9 counts the TTL only while the receiver is live and the pause does not cover it, and card 9 repeats that.
An inject queued for a dormant strand therefore stays queued and lands at the first idle after the strand is live, at any later time, and only the removed-holder sweep ends it once the strand is gone from every reed state.
D9's own Bound limits the TTL claim to a long busy turn.
The Bound also says nothing of the interrupt forms, which stop a running turn for the same privileged senders.
The attack-surface rule asks the Intent to state what a change lets through and what bounds it, and a bound that does not hold is as unhelpful as a missing one.

### [MEDIUM] TestCreelCLI's strand identity rests on unnamed process-global environment

**Location:** `_lyx/plan/14-creelcli-verbs.md:38`, with `_lyx/plan/14-creelcli-verbs.md:21` and `_lyx/plan/14-creelcli-verbs.md:22`

**Issue:** The CLI builds its `Claim` from `$LYX_STRAND_NAME` (line 21), and the only test seam named is the clock and spawn (line 22).
So the strand rows ("with the claim's `LYX_STRAND_NAME` set to that strand's name") can only set the process environment, which forbids `t.Parallel`, and the card says neither.
The EXTERNAL rows script no strand, so they take whatever `LYX_STRAND_NAME` the test process inherited; agents run the gate from a strand pane.
On Linux an inherited name of another hub's shortname falls through to EXTERNAL.
Off Linux, `peer_other.go` reports no peer and the environment is the identity, so an inherited name that parses under the test hub's shortname resolves the row as that strand and breaks it.
The strand rows themselves hold off Linux, because the name they set is the identity there; card 10's rows build the `Claim` field directly and are unaffected.

### [LOW] creelfake.Clock is not described as recording waits

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md:42`

**Issue:** Card 9's `TestScheduler_Reminders` and `TestSweep_RemovedHolders` read the probe and sweep intervals "off the fake clock's waits", and card 10's timeouts need the same clock.
Card 7 creates `creelfake.Clock` as one that "advances only when told" and says nothing of recording the durations given to `After`, and card 9 does not edit the kit.
The waits-row of the focus directive is satisfiable only if the kit records them, so card 7 should say it does.

### [LOW] Tests importing creelfake cannot be in package creelengine

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md:43`, with `_lyx/plan/09-creelengine-delivery.md:35` and `_lyx/plan/10-creelengine-daemon.md:36`

**Issue:** `creelfake` imports `internal/creelengine` (card 7, "imports only `internal/creelengine` and the standard library"), and cards 7, 9 and 10 plan `verbs_test.go`, `deliver_test.go` and `server_test.go` "over `creelfake`" inside `internal/creelengine`.
A `_test.go` file in package `creelengine` that imports `creelfake` closes an import cycle, which `internal/testkit/shuttlefake`'s header gives as the reason `internal/shuttleengine` keeps its own in-package fake.
The three files use exported API only (`Store`, `Scheduler`, `Run`, `NewClient`), so the external package `creelengine_test` works; the plan should say so.

## Rejected

Each item below is a focus-directive question with a verdict of not a defect, or a check that held.

- Test tiers of cards 13 and 14: `delivery_test.go` and `cli_test.go` both state `//go:build integration` with the spawn that forces it (`fabricengine.List`'s `git worktree list` via `hubforge`, and `preflight.ResolveMode` running git through `lyxcwd.Resolve`), which `PATTERN-test-speed` places in tier 2; `testmain_test.go` stays untagged and calls only `gitkit.HermeticGitEnv` and `tmuxkit.Main`, both untagged-safe; the plan's `go test -tags integration ./...` runs both files.
- Sweep interval of card 9: it states a 30-second base, a named 4-minute ceiling, the reset conditions, the What-counts entry text and a `TestSweep_RemovedHolders` row that reads the waits off the fake clock, so it meets `PATTERN-daemon-wakeups`; the departure from D15's fixed 30 seconds is stated and licensed by the constraint.
  F1 is about the worker, not the sweep.
- Card order of 7, 9 and 10: card 7 creates the waiter registry, the wake notification and an in-process way to register a waiter, and its own reply rows use it, so it depends on nothing a later card creates; card 9's sweep rows answer waiters registered in-process through card 7's registry, and card 10 only binds a connection to the same registry.
  F2 concerns an Edit listing, not an order break.
- `withheld` in card 6: it is in the event list, the fold ("the withheld reminders per orch receiver") and `TestJournal_Fold`; card 9 names the writers (the verb or sweep that changes the unread set), and F3 covers the one change it omits.
- Reconnect rows of card 10: `TestDaemon_Lifecycle` has a row where an EXTERNAL `post --wait-reply` loses its connection, a strand replies while no waiter is connected, and the reconnected waiter receives the reply at once with a `delivered` event journaled, plus rows for the injected and returned-undelivered outcomes.
- `PATTERN.md` headroom in card 11: `HeadroomBytes` is 95 percent of 16 KiB, which with the file's current size leaves 9 bytes, as the card states.
- `lens-review.md` sits in the plan directory but is not in the Card Index and is not a card; it is not judged.
