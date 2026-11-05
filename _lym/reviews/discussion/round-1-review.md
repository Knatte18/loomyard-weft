---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D11 (third paragraph) and D10 (Bound)
    summary: D11's crash recovery and D10's goto bound rest on the dobby rewriting the land request, but the dobby's persisted land episode stops it, and the dobby-stop marker D11 keys on is already removed by the accept
    fix: In D11, replace the sentence that starts the dobby when the dobby-stop marker stands and has the dobby write the land request again. State instead that Beam-Await's entry, on finding the hold named Finalize or a retired row with no land request, reads the tracked ready record for this slug. If the record stands, the entry runs the accept's re-check itself, with no land request, and then is Done or refuses as the accept does. If it does not stand, the entry lifts the hold and starts the dobby whether or not the marker stands. Give the same entry rule in D10's Bound for a goto back to Beam-Await while the ready record stands. Drop the claim that the dobby lands the beam again. Update D11's and D10's acceptance criteria to match. Name the dobby's persisted land episode (dobbyengine State.Land, cleared only on a non-passing reading) in Constraints as the fact this rests on.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 (Bound), D3 (Bound), D10 (Bound)
    summary: A hand lym batten run of a member is gated only by RunHolds, which reads Beam-Await as hold-free, so a member starts and lands into a ready beam, against D2's bound and the entry's hold from lym beam ready on
    fix: Add to D2 that a member run's fork precondition, the RunHolds check in battencli's beam fork, also refuses a member whose beam's ready record names that beam. The refusal names `lym batten cancel <member> --remove`, as the dobby notice does. Rewrite D2's Bound to name both guards, the dobby's start hold and the fork precondition, and add an acceptance criterion that a hand `lym batten run` of a member under a ready beam is refused. Adjust D3's and D10's Bounds to cite D2's guards rather than the dobby alone.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D11 (second paragraph)
    summary: Counting a loop exit after a completed wake as a new stop leaves the repair cap unbounded, and no fact is named that tells a completed wake apart, since the stop key (producer, history length, state) does not change at Beam-Await
    fix: In D11, name the fact the driver reads to tell a completed wake apart, a record Beam-Await writes per completed wake that the driver compares with the last restep settle record of the stop key. State the bound on re-steps, for example that the count resets only when the loop ran a stated minimum time since the last re-step, so a loop that dies after every first wake still reaches the tackler and the arbiter. Add an acceptance criterion that a loop crashing after each first wake reaches the tackler.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (refusal 6) and D8 (first bullet)
    summary: A recorded pull request merged on GitHub at an older head passes lym beam ready, but the already-landed path holds only what the parent already has, so later beam commits are lost or Finalize sticks; PR-Review today refuses this case
    fix: In D3 refusal 6, pass a merged recorded pull request only when its merged head is the beam's head. Otherwise refuse, naming the commits the merge lacks and the way forward, which is to close the record with `lym beam pr close <slug>` and land without the pull request or open a new one with `lym beam pr open <slug>`. State in D8 that Finalize halts on the same mismatch, as PR-Review does today, and add both cases to D3's and D8's acceptance criteria.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 (first bullet) and D6 (status)
    summary: After Finalize merges the parent in and pushes, required checks are pending or a stale approval is dismissed, which GitHub reports as RefusalOther and not RefusalPending, so D8's halt naming the check never fires, and today's way forward names lym batten cancel
    fix: In D8's recorded-pull-request bullet, state that every GitHub merge refusal Finalize keeps after the one-minute retry window (pullrequest's RefusalPending), and every RefusalOther, halts naming GitHub's message, `lym beam pr status <slug>` and `lym loom resume`, never `lym batten cancel`. Record in Constraints, citing classifyRefusal in internal/pullrequest, that a pending required check and a dismissed review come back as RefusalOther. Add the case to D8's acceptance criterion.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D13 (A run at Finalize or past it is untouched) and D8
    summary: A pawl run the old binary left at Finalize after PR-Review opened a pull request keeps it only in the scratch record, which D8 no longer reads for a beam, so under beam.yaml on Finalize squash-merges locally and orphans the open pull request
    fix: In D13, state that a pawl run found at Finalize with an open pull request in its scratch record has that pull request carried into D4's record once, as the move's carry does, without moving the run. Alternatively, state in D8 that a pawl Finalize also reads a scratch-recorded pull request the old PR-Review left. Add the case to D13's acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks
    summary: The board entry asks that a push lock whose holder died is taken as stale, but such a flock is already free; whether lym should stop a live push left from before a restart, or the point drops, is the parent's call
    fix: Parent decides whether issue #47's push-lock point drops or becomes a rule for stopping a live pre-restart push. The writer records the answer in Scope and, if kept, as a Decision with its bound. The open-risk text should also replace the word likely with the researched cause, the detached push child fabric spawns, if that is what held the lock.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D13 (pawl side effects) against D4
    summary: D13 writes D4's tracked landing record and removes the tracked ready record without stating a commit, while D4's writers commit and the existing ready-record removal commits the landing directory
    fix: In D13's pawl list, state that the carry into D4's record and the removal of the ready record are committed together in the beam worktree, as `lym beam reopen`'s removal commits the landing directory today. Add the commit to D13's acceptance criterion.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 (close) against D8 (second bullet)
    summary: lym beam pr close comments that the operator chose to land without a pull request, but on a beam.yaml merge parent D8 still finds or opens one and merges it, so the comment is false and the skip is overridden silently
    fix: In D6's close bullet, state that on a parent whose resolver answers merge or required, close reports that Finalize will still land through a pull request. Alternatively, have close refuse there, naming beam.yaml. Make the GitHub comment conditional on the resolver answering on or off.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 (no verb removes the ready record) against D13
    summary: D3 says only Finalize's landing and Beam-Down end the ready record, but D13's move removes it too
    fix: In D3's paragraph after the refusals, change Finalize's landing and Beam-Down end it to Finalize's landing, Beam-Down and D13's move end it.
  - id: F11
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D13 (pawl side effects) with D10
    summary: D10 removes Beam-Await's reopen handling, the only thing that removes a reopen request, but D13's move does not remove a reopen request the old binary left, so the file stays with no reader
    fix: Add the reopen request to D13's list of removed files and to D13's acceptance criterion.
  - id: F12
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 (notice text) and D5
    summary: D7's notice prescribes lym beam describe, which regenerates summary.md from the commits, so any text the orch or operator edited into the description is overwritten at every member landing, the issue 56 case
    fix: In D5, state whether the orch may edit summary.md by hand. If it may, D5 refuses or warns when summary.md differs from what the last describe wrote, and D7's notice names editing the description or running describe. If it may not, say that describe owns the file and the orch's text comes only through the entry's title and brief.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D11 (arbiter eligibility sentence)
    summary: arbiterstop.Eligibility reads no recipe, so pawl is not excluded today; only Beam-Await's missing InterruptPolicies entry hands the stop back, and the stated extension of eligibility is an empty change a plan writer may try to build
    fix: In D11, replace the sentence on the arbiter's eligibility extending from tapestry, darn and sampler to pawl with one stating that the re-invoke entries in loom's interrupt-policy table make the stop eligible under the arbiter's existing rule. Fix D11's acceptance criterion to match.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D13 (judged first) and Constraints (PATTERN-shed-stop-chain line)
    summary: D13 says the move's halt is judged first as the parent's, but judged first in the stop chain and the driver's JudgesFirst seam means the arbiter judges first; a halt meant for the parent is a paused state or a HandsBack halt
    fix: In D13 and its Constraints line, state that the move leaves the run paused with its reason, which the driver's rule table hands back to the parent, or a blocked halt that HandsBack marks, and drop the words judged first.
  - id: F15
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 (status, unknown) and D8 (retry window)
    summary: D8's retry window is the existing one-minute prMergeRetryWindow but is left unnamed, and D6's short bounded window for unknown has no value, so the plan writer has to pick or find both
    fix: In D8, name the window as Finalize's existing one-minute merge retry window. In D6, give the unknown retry's value, for example the same one minute or a stated number of polls, and state it in D6's acceptance criterion.
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D4 (Rationale)
    summary: D4's Rationale carries a requirement the plan writer needs, that Finalize's scratch pull-request record keeps serving a pull request Finalize opens itself; it belongs in D8's decision text
    fix: Move the clause Finalize's scratch pull-request record keeps serving what Finalize opens itself out of D4's Rationale into D8's second bullet, as the record a pull request Finalize opens or finds is kept in. Leave D4's Rationale with the restart and adopt reason only.
  - id: F17
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Auto-mode assumptions and D2 (Rationale)
    summary: Two auto-mode assumptions restate Scope and D3/D6 without adding an interpretation, and D2's Rationale opens by restating D2's decision
    fix: Cut the assumption No lym beam pr merge verb (D6), which Scope Out and D6 already state, and the assumption lym beam ready refuses more than the two causes the entry lists (D3). Cut D2's Rationale clause the entry holds the beam from lym beam ready on and moves the RunHolds boundary from Parent-Align to Finalize, keeping the reason the dobby enforces the hold.
---

The record's direction fits the board entry. D1, D4, D5, D6, D9, D12 and D14 hold as written.
The verdict is BLOCKING on F1.
Issue #57's purpose, Beam-Await recovering with no hand step, rests on the dobby rewriting the land request, and today's dobby state forbids that.
Six MEDIUM design gaps sit around the ready window and the in-flight move.

### [BLOCKING] D11 and D10 rest on the dobby rewriting a land request it never rewrites

**Location:** decision-record.md D11, third paragraph; D10, Bound.
**Issue:** D11 recovers a crash between the accept's removals and Done in two steps: the dobby is started "when the dobby-stop marker stands", and then "the dobby writes the land request again from the tracked ready record".
Both steps fail.
First, the accept removes the dobby-stop marker together with the land request before Done (`internal/pawlengine/beamawait.go` `accept`), so in that window the marker is gone and the dobby stays down.
Second, the dobby keeps its land episode in its state file across restarts (`internal/dobbyengine/land.go` `recordLand`: "so neither this dobby nor a restarted one acts on it again"), and clears it only on a reading that does not pass.
With the ready record standing and nothing working under the beam, the reading keeps passing, so a restarted dobby returns early and never asks again.
Today's interrupted-landing notice names a hand `lym beam land` for exactly this reason.
D10's bound ("the dobby lands the beam again once nothing works under it") breaks the same way after a goto from Finalize.
Three lenses found this independently.

### [MEDIUM] A hand member run lands into a ready beam

**Location:** decision-record.md D2, Bound; D3 and D10 Bounds.
**Issue:** Between `lym beam ready` and the accept, the beam's status names Beam-Await, which `boardengine.RunHolds` reads as hold-free.
A hand `lym batten run <member>` is gated only by that check (`internal/battencli/beamfork.go`), and `PATTERN-dobby-starts-runs` admits a hand pass.
The member starts, runs and lands into the ready beam.
That moves the beam head past the head D3 checked against the pull request, and past the commits the description lists.
D2's bound claims the opposite.

### [MEDIUM] D11's repair-cap reset is unbounded and names no distinguishing fact

**Location:** decision-record.md D11, second paragraph.
**Issue:** The repair count is keyed by (producer, history length, state) in `internal/sheddriver/driverules.go`, and every loop exit at Beam-Await leaves the same key.
D11 names no fact that marks a wake as completed.
Once every such exit resets the cap, nothing bounds a loop that dies after each first wake: it re-steps forever and never reaches the tackler, the arbiter or the parent.
The rubric's attack-surface rule requires a weakened guard to state its bound.

### [MEDIUM] A pull request merged on GitHub at an older head passes ready

**Location:** decision-record.md D3 refusal 6; D8.
**Issue:** Refusal 6 compares heads only for an open pull request, and it passes a merged one.
The already-landed path holds only what the parent already holds.
Today's PR-Review halts when a pull request was merged at a head other than the one it gated (`internal/landingshed/prreview.go`), and the record drops that guard.

### [MEDIUM] Finalize's recorded-pull-request halts miss GitHub's actual refusal classes

**Location:** decision-record.md D8, first bullet.
**Issue:** Finalize merges in the parent and pushes a new head, so required checks restart and a stale approval may be dismissed.
`classifyRefusal` in `internal/pullrequest/client.go` maps those refusals to RefusalOther.
RefusalPending covers only a moved base and mergeability still being computed.
D8's "pending merge past the retry window halts naming the check" therefore never fires for a check, and the RefusalOther path today names `lym batten cancel`, which is wrong for a beam.

### [MEDIUM] Runs at Finalize under the old binary orphan their pull request

**Location:** decision-record.md D13 ("A run at Finalize or past it is untouched"); D8.
**Issue:** The old PR-Review records its pull request in the run's scratch record.
D8 decides a pawl landing from D4's tracked record only, and D4's Rationale keeps the scratch record only for what Finalize opens itself.
Under beam.yaml `on`, a run left at Finalize squash-merges locally and leaves the open pull request behind, which contradicts the entry's "a run at Finalize or later goes on as before".

### [MEDIUM] Push-lock point needs the parent's decision

**Location:** decision-record.md Open risks.
**Issue:** The writer correctly found that a flock frees when its holder dies (`internal/lock/lock.go` uses gofrs/flock), so the entry's stale-lock ask has nothing to do.
Whether lym should instead stop a live push left from before a restart is a scope choice only the parent can make.
The cause is still stated as "likely"; the detached push child fabric spawns (`internal/fabricengine/coalesce.go` `CoalescePushRequest`) is the candidate the record should name once confirmed.
This answers the focus question: the choice is the parent's, and the cause is a fact the writer should state.

### [MEDIUM] D13's tracked-file changes carry no commit

**Location:** decision-record.md D13, pawl side effects.
**Issue:** The carry writes D4's tracked record, and the move removes the tracked ready record.
Every other writer of these files commits: D4's verbs, and `lym beam reopen`'s removal (`internal/beamcli/doc.go`).
D13 is silent on the commit, which leaves the beam worktree dirty.

### [LOW] D6 close comments a choice D8 overrides on a merge parent

**Location:** decision-record.md D6 close; D8 second bullet.
**Issue:** On a `merge` parent, D8 opens and merges a pull request after the orch closed one "to land without", so the GitHub comment is false.

### [LOW] D3 misstates who ends the ready record

**Location:** decision-record.md D3.
**Issue:** D13's move also removes the ready record.

### [LOW] A reopen request left by the old binary outlives both D10 and D13

**Location:** decision-record.md D13; D10.
**Issue:** After D10, nothing reads or removes a reopen request, and D13's move leaves one standing.

### [LOW] Describe overwrites a hand-edited description

**Location:** decision-record.md D7; D5.
**Issue:** D7's notice sends the orch to `lym beam describe`, which regenerates `summary.md`.
D5 is silent on whether the orch's own text in the file survives, which is the issue #56 concern.

### [LOW] D11's arbiter-eligibility extension is an empty change

**Location:** decision-record.md D11.
**Issue:** `arbiterstop.Eligibility` has no recipe gate.
Only the missing `InterruptPolicies` entry hands a pawl stop back today, so the policy change alone admits it.

### [LOW] D13's "judged first" uses the arbiter's term for a parent hand-back

**Location:** decision-record.md D13; Constraints.
**Issue:** The driver's `JudgesFirst` seam routes a stop to the arbiter.
D13 means the opposite.
The record should name the paused state or the HandsBack halt it means.

### [LOW] Retry windows unnamed or unvalued

**Location:** decision-record.md D6 status; D8.
**Issue:** D8's window exists in code as one minute (`prMergeRetryWindow` in `internal/landingshed/finalize_pr.go`), so it is settled but unnamed.
D6's window for `unknown` is a new value the record leaves open.
This answers the focus question: neither window is a deferred research fact.
Both are values the writer should state.

### [LOW] D4's Rationale hides a requirement

**Location:** decision-record.md D4 Rationale.
**Issue:** Finalize's scratch record serving the pull requests it opens is a requirement the plan writer needs, but it appears only in a Rationale.
Under the completeness-before-leanness test, it moves into D8's decision text.

### [NIT] Restated assumptions and rationale

**Location:** decision-record.md Auto-mode assumptions; D2 Rationale.
**Issue:** Two assumptions and D2's opening Rationale clause restate Scope and Decisions without adding anything.

## Focus verdicts

- D11: unbounded (F3), and its recovery premise is false (F1).
- D13: the move's removal of a ready record that stood at the move is correct, since without it D2's start hold and the D5 and D6 refusals would lock the orch out of the new verbs. Only the commit is missing (F8). No other status moves: the map is per recipe, and any other unknown row keeps the goto refusal. A run at Finalize is a separate gap (F6).
- D2 and D10: D2 lets a hand member run through (F2). D10's goto lets nothing new in, but cannot land again (F1).
- D6 and D8: see F15. Neither window is a deferred research question. F5 is the substantive gap behind D8's halt.
- Open risks: a parent choice (F7).
- Relocation: F16 extracts D4's requirement into D8. F17 cuts the restatements. Nothing in D2's Rationale carries a requirement.

## Rejected

- Bound-breaker's note that D7's fresh-machine notice may list already-announced commits: the bound says at most one repeated notice, and that holds.
