---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/creelcli/goclient.go#AwaitsReplyProbe
    summary: "The reply probe always signs as a program, which the daemon refuses from a non-orch strand pane, so runs waited on from an agent's pane, such as webster's fixers and recovery agents driven from the master session, never hold."
    fix: "In AwaitsReplyProbe, claim the way orchcli's notifyPrime does. When LYM_STRAND_NAME names a strand whose role is not an orch's, build the client with NewStrandClient(location, thatName) first. Fall back to NewProgramClient(location, program) only when that call fails with creelengine.ErrStrandClaimOutsidePane. awaits-reply reads the strand from Request.From, so either signature answers the same. Add a creelcli row that calls the probe from inside a non-orch strand pane (the orchRig pane rig with PanePID set to os.Getpid() on a worker strand), and assert it answers true for that strand's open question instead of failing with ErrProgramClaimInStrandPane."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/replyhold.go#releaseReplyHold
    summary: "Two quiet probes release the hold whenever the reply's turn shows no turn start, yet on Claude the reply wakes the session through an asyncRewake Stop hook, and a turn start is read only from UserPromptSubmit. An ungated run can then finalize, or a gated run evaluate its gate, 5 to 10 s into the turn that acts on the reply."
    fix: "Settle the premise first. Add a claudeengine corpus or tier test that records an asyncRewake wake and asserts whether ParseSessionSignals yields a SessionSignalTurnStart for the woken turn. If it yields none, change releaseReplyHold so that a probe sequence that saw the strand awaiting a reply, then quiet, keeps the hold until a turn end is read (eventsRead), which the existing Done path then decides. Keep the two-quiet-probe release only for a question that settled with no reply read. In either case, add a TestRun_Wait_ReplyHold row on an engine that parses session signals: the reply writes a turn start and no turn end for longer than replyQuietProbes times gateRecheckEvery, and the assertion is that Wait returns only after that turn's end. The row also pins the hold clear at wait.go:332-334 and turnStartSeen in pollEventsTick, which survive mutation today."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/creelengine/verbs.go:1095
    summary: "AwaitsReply keys the asked predicate on the strand name alone, and releaseGoneHolder never settles questions a removed strand sent. A later session with the same constant role name, such as the next round's fixer, inherits an earlier session's unread or chased question, and every finishing turn end of the new run is held until that question settles or the deadline passes."
    fix: "Settle a removed asker's open questions in releaseGoneHolder: for each message whose From is the removed strand, ExpectReply is set, Replied is false and Unanswered is empty, record it unanswered with a new reason such as asker-gone. A new session with the same name then never counts them. Add a TestStore_AwaitsReply row in which alpha asks orch a question, alpha's strand is swept as gone, and AwaitsReply(alpha) answers false."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/gatecli/cardfixer.go:128
    summary: "The gate card fixer's runner gets no reply probe, yet the fixer's prompt renders the changed parent directive, which now promises the run waits for the answer. The board entry's waits-for-the-reply goal stays unmet for that run agent, and the directive is false for it. Weaver's runner at internal/weavercli/wiring.go:44 renders the same directive with no probe as well."
    fix: "Call runner.SetReplyProbe(creelcli.AwaitsReplyProbe(location, <the card fixer's program constant>)) where internal/gatecli/cardfixer.go builds the fixer runner. Check whether the runner weavercli/wiring.go:44 builds waits on agent runs whose prompt renders the parent directive (internal/weaverengine/render.go:95), and wire the probe there too if it does. If creelcli cannot be imported at a site, pass the probe in from that site's caller. Update the summary's Bounds list to match."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/replyhold_test.go:562
    summary: "No row has a gated release whose evaluation leaves the run going, so dropping run.replyHold = nil in releaseReplyHold (mutation survivor replyhold.go:75) passes every test. A stale hold would then re-run handleGatedBoundary every gateRecheckEvery in place of the pending and unsent-reprompt branch."
    fix: "Add a TestRun_Wait_ReplyHold row with a gated run whose gate fails with Attempts 2. After the question settles with no reply and the two quiet probes release the hold, assert that the gate is evaluated exactly once and one re-prompt is sent before the next turn end. That count is not one per 5 s tick."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelengine/awaitsreply_test.go:180
    summary: "TestStore_AwaitsReply pins neither the EndedUnreadBy clause, a question returned unread to its asker, nor the reply side's original lookup, unread mail the strand holds that answers no message of its own. Dropping either check passes the table."
    fix: "Add two rows to TestStore_AwaitsReply, both with want false. In the first, a journal records alpha posting an expect_reply message to orch, then a returned event moving it back to alpha with receiver orch. In the second, beta posts plain mail to alpha, which alpha has not read."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/replyhold.go:502
    summary: "The hold's pause paths are untested. One is the release under a covering pause, which sets boundaryHeld. The other is holdForReply in the pause-lift branch at wait.go:777."
    fix: "Add a gated TestRun_Wait_ReplyHold row with a pause-record fixture that covers the run while the hold releases and lifts later. Assert that no gate runs while covered, and that the gate runs once on the first uncovered tick when the probe is quiet."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:714
    summary: "Nothing fails when any of the four runner.SetReplyProbe wiring lines is dropped: TestWire_EverySeamFilled checks only c.env seams. The same gap applies at burlercli/wiring.go:152, webstercli/wiring.go:204 and battencli/drive.go:518."
    fix: "Expose whether a runner has a reply probe through an export_test.go accessor in shuttleengine, or extend each CLI's existing wiring test to assert it on the wired runner. Cover the loom, burler, webster and batten wiring tests."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The Merge danger blast radius names only shuttle. The diff also adds a creel protocol verb that the daemon of another binary answers, and changes the parent directive every non-interactive role renders, so the radius names less than the change can affect."
    fix: "In the summary's Merge danger section, change the blast radius to name shuttle, the creel daemon protocol (the new awaits-reply verb, refused by an older daemon as a probe error) and the parent-directive stencils every non-interactive role renders."
---

The diff holds a run's finishing turn end while creel reports its strand awaiting a reply, and it rewords the parent directive and the drift judge's mode text. That builds the board entry's two goals as designed. Two defects keep the hold from working on the paths the entry reports: the probe's sender claim, and the release's dependence on a turn start the reply's wake may not write.

### [BLOCKING] The reply probe is refused from a non-orch strand pane

**Location:** `internal/creelcli/goclient.go#AwaitsReplyProbe`

**Issue:** `AwaitsReplyProbe` always builds `NewProgramClient`, a claim with `Program` set and no strand name. The daemon's `resolveSender` refuses a program claim whose peer runs inside the pane of a strand that is not an orch, wrapping `ErrProgramClaimInStrandPane` (`internal/creelengine/identity.go:75-80`).

Wired runners do run in such panes. The webster master stencil has the master session drive `lym webster begin-batch`, `fix-verify`, `recover-batch` and `fix-plan` through its Bash tool (`contracts/stencils/webster/webster-template-master.md:22`), and `websterengine/runlevel.go:826` waits on the run handle in that CLI process. Every probe from there fails. `Run.awaitsReply` reads that failure as no reply awaited, so the run finalizes as before, with one warning.

The runners' existing notifier already handles this case. `orchcli.notifyPrime` (`internal/orchcli/notify.go:243-273`) tries a strand claim first when `LYM_STRAND_NAME` names a non-orch strand, and falls back to the program only on `ErrStrandClaimOutsidePane`. The probe skips that, so the hold misses webster's recovery agents and fixers, the first of the two agents the board entry names.

### [BLOCKING] The quiet-probe release can end a run mid-turn on the reply

**Location:** `internal/shuttleengine/replyhold.go#releaseReplyHold`

**Issue:** The hold ends early only on `eventsRead || turnStartSeen` (`wait.go:332-334`). Otherwise two quiet probes, `gateRecheckEvery` apart, release it. The summary justifies the second probe by saying it "sees that turn start first".

On Claude, `ParseSessionSignals` reads a turn start only from `UserPromptSubmit` (`claudeengine/signals.go:170-172`). The reply reaches the idle agent through creel's idle waiter, an `asyncRewake` Stop hook (`claudeengine/settings.go:212`), which claims the mail, marking it read, before it exits 2. The repository's channel research (`docs/research/creel/creel-hook-channels-2026-10-28/report.md`, the asyncRewake row) records that this wake started a turn by itself and showed a `Stop hook feedback` line. It records no prompt submission.

If that turn writes no turn start, the probe goes quiet once the reply is read. The hold then releases 5 to 10 s into the turn that acts on the reply: an ungated run finalizes done, and a gated run's `boundaryIdle` finds no unmatched turn start and evaluates the gate mid-turn. The test cannot show the case either way. `TestRun_Wait_ReplyHold` drives `fakeEngine`, which parses no session signals, and writes the reply's turn end on the same step the probe goes quiet. Mutation survivors at `wait.go:333`, `:969` and `:986` confirm that nothing pins the hold clear. The premise is unverified rather than disproved; the fix asks to settle it.

### [MEDIUM] A reused strand name inherits an earlier session's open question

**Location:** `internal/creelengine/verbs.go:1095`

**Issue:** `asked` matches `msg.From == strand` over the whole pool. A spawned role's strand name is its module's constant, so the next round's fixer or reviewer in the same worktree carries the same name. `releaseGoneHolder` (`sweep.go:215-258`) settles only what a removed strand held, never the questions it sent.

An earlier session's question that the orch has not read, or read and left unanswered, therefore counts for the new session. The chase runs every 15 minutes (`chase.go:17`) before it settles the question. Meanwhile every finishing turn end of the new run is held, until the deadline when the question stays unread.

### [MEDIUM] The card fixer's runner gets no probe, though its directive promises the wait

**Location:** `internal/gatecli/cardfixer.go:128`

**Issue:** The board entry asks that an agent asking with `--expect-reply` waits for the reply. The diff wires the probe only into runners that already had a notifier. The gate card fixer renders the non-interactive parent directive (`internal/gatefixer/prompt.go:77`), which now says "your run waits for the answer". Its runner gets `SetPause` and `SetSessionEnd` but no probe, so the defect the entry reports still holds for it, under a promise that is now false.

The summary's Bounds section names the exclusion as the writer's choice, and no `Added after Discussion` entry records it. Weaver's runner (`internal/weavercli/wiring.go:44`), whose writer renders the same directive (`internal/weaverengine/render.go:95`), is unwired too. The lens did not confirm that that runner waits on agent runs.

### [LOW] No row pins the release's hold clear for a gate that keeps the run going

**Location:** `internal/shuttleengine/replyhold_test.go:562`

**Issue:** The gated rows' gate passes at once, so dropping `run.replyHold = nil` in `releaseReplyHold` (survivor `replyhold.go:75`) passes. A stale hold would shadow the pending and re-prompt branch and re-evaluate the gate every 5 s.

### [LOW] TestStore_AwaitsReply misses the returned-question and foreign-mail cases

**Location:** `internal/creelengine/awaitsreply_test.go:180`

**Issue:** Without the `EndedUnreadBy` clause, a question returned unread to its asker would count forever, and no row catches that. Without the `original.From == strand` check, any unread reply the strand holds would count, and no row catches that either.

### [LOW] The hold's pause paths are untested

**Location:** `internal/shuttleengine/replyhold.go:502`

**Issue:** Nothing tests two paths: a release under a covering pause, which sets `boundaryHeld`, and the re-probe when the pause lifts (`wait.go:777`).

### [LOW] The four SetReplyProbe wiring lines are unpinned

**Location:** `internal/loomcli/wiring.go:714`

**Issue:** Dropping the wiring line in loom, burler, webster or batten fails no test. The wiring is what carries the fix to real runs.

### [LOW] The Merge danger blast radius understates the change

**Location:** `_lym/landing/summary.md`

**Issue:** The blast radius names `shuttle` alone. The diff adds a creel daemon verb that another binary's daemon answers or refuses, and it rewords the parent directive every non-interactive role renders. The two-way call itself holds: an older daemon refuses the verb, and the probe error falls back to the old behaviour.

## Rejected

- code-tests NIT, no row asserts that the hold sends no notice: the held branch never reaches `handleHeldTurnEnd`, so the absence of a notice follows from the code's structure, and a row for it would pin nothing a reader cannot see.
- code-correctness tool note that the notifier signs as a program the same way: `notifyPrime` tries a strand claim first for a non-orch strand, which is the basis of F1, so this note is not adopted.
- code-tests MEDIUM on the untested turn-start hold clear: merged into F2, whose fix adds that row.
- code-affected-set MEDIUM on the parent directive being false for unwired runners: merged into F4, the same defect seen from the stencil side.
- Focus directive 4, the awaits-reply verb taking its strand from `From` with no sender check: the verb is read-only and answers one boolean, and its only sender is the Go client, as `doc.go` claims. Any hub caller can learn whether a strand awaits a reply, which reveals nothing that reaches an agent. Not a defect.
- Focus directive 2, starvation in the else-if chain: the hold branch skips the pending re-evaluation and the unsent re-prompt by design, while the deadline and liveness checks run after the chain on every tick. No starvation was found.
- The assistant reviewer returned no report: its seat died at startup, as its end marker records, so none of its findings enter this review.
