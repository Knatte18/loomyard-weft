---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/14-beam-pr-status-close.md:28
    summary: PullRequestMode reads GitHub's rules unconditionally through pullrequest.Open, so lym beam ready (card 15 refusal 5) fails on every hub with no GitHub origin or no usable token, whatever the parent's mode
    fix: In card 14's PullRequestMode bullet (14-beam-pr-status-close.md:28), state that hubLandDeps reads GitHub's rules only when landingshed.OriginMissing(origin) is empty, and that a missing origin fact, an unusable token or a non-transient read failure reads as no rule with one logger.Warn, exactly as landingshed.resolveWithParentRule reads it (pullrequestmode.go:113-139); add `plan:internal/landingshed#OriginMissing` under card 14's Uses. In card 15's refusal 5 (15-beam-ready-refusals.md:34), state that a PullRequestMode error refuses naming the cause and `lym beam ready <slug>` again. Add TestMarkReady rows in card 15 - an off parent whose origin is not on GitHub, with none recorded, writes the record with no rules read; an on parent whose rules read fails with a token failure, with none recorded, writes the record - and a TestPRClose row in card 14 for the same no-GitHub off parent.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/01-shed-retired-rows.md:25
    summary: Cards 1 and 5 add a second retired-row mechanism (the recipe's retired map, Shed.Retired, loomrecipe.RetiredRowNames) beside the existing loomrecipe.RetiredRows rename at arm, and reconcile neither the code nor the docs that call RetiredRows the recipe's retired-row data
    fix: In card 1's Intent (01-shed-retired-rows.md:25) and card 5's pawlengine bullet (05-pawl-rows.md:65), name the existing arm-time rename (loomrecipe.RetiredRows applied by loomCLI.renameRetiredRows through shedengine.RenameProducers, internal/loomcli/arm.go:288) and state the split - RetiredRows renames a row that lives on under a new name, at arm, with no side effect; the recipe's retired map moves a run off a row the recipe dropped, at step, pausing it with the recipe wiring's side effects. In card 5's Intent, add that internal/loomrecipe/doc.go's RetiredRows paragraph (doc.go:39) and internal/loomcli/doc.go's retired-row sentence (doc.go:11) say the two apart, and that RetiredRowNames' doc says it reads the recipe's retired map and never RetiredRows.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/05-pawl-rows.md:65
    summary: Text that states the old hold boundary, Parent-Align's merge-in at landing, the reroute and the BeamDescribe row survives every card - runTakesNoSlot's doc, dobbyengine's slot paragraph, loomrecipe's pawl paragraph, mergeParentCmd's help, beamcli's merge-in paragraph and pawlwiring.go's header
    fix: Card 5 adds `internal/battencli#runTakesNoSlot` under Edit and its Intent says that doc comment (battencli/dobbyreads.go:214) now reads that a pawl run takes a slot from Finalize on, and that internal/loomrecipe/doc.go's pawl paragraph (doc.go:43-46) names the six rows, drops the reroute edge and `lym beam reopen`, and holds the beam from Finalize on. Card 7's Intent says internal/dobbyengine/doc.go's slot rule (doc.go:119) reads Finalize for Parent-Align. Card 12 adds `internal/loomcli/pawlwiring.go` under Edit and its Intent drops "beyond the BeamDescribe row" from that file's header. Card 15 adds `internal/beamcli#mergeParentCmd` under Edit and its Intent rewrites that Long (beamcli/cli.go:55-61) and internal/beamcli/doc.go's merge-in paragraph (doc.go:92) to say the orch runs the merge-in before `lym beam ready`, which refuses while the parent holds commits the beam lacks, and Finalize merges the parent in again at landing.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/15-beam-ready-refusals.md:34
    summary: Cards 14 and 15 call ResolvePullRequest with taskHasCommits hard-coded true, so a beam with no commits beyond its parent under a required or rules-required parent is refused naming lym beam pr open, which card 13 refuses for that very beam, leaving no way forward
    fix: In card 15's refusal 5 (15-beam-ready-refusals.md:34) and card 14's prClose bullet (14-beam-pr-status-close.md:30), pass deps.BeamHasCommits(slug), card 13's seam, as ResolvePullRequest's taskHasCommits in place of true, with a BeamHasCommits error refused naming the cause and the rerun; add a TestMarkReady row in card 15 - a required parent, a beam with no commits beyond its parent and none recorded passes refusal 5 and writes the record.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-beam-describe-verb.md:44
    summary: judgeAwaitingRun passes a Beam-Await run that is running, blocked or paused and leaves failed unstated, but Beam-Await's own halt is an error stop persisted as failed (shedengine run.go:410) and it never halts blocked, so D5's accepted halted run is the unnamed case and TestDescribe pins an unreachable one
    fix: In card 12's judgeAwaitingRun bullet (12-beam-describe-verb.md:44), state that a run at Beam-Await passes in every state but done - running, paused, failed and blocked - since Beam-Await halts by an error stop the status records as failed; in card 12's TestDescribe description (12-beam-describe-verb.md:50), make the halted passing row a failed status at Beam-Await in place of the blocked one.
  - id: F6
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/10-beam-await-entry.md:44
    summary: A crash inside awaitRun.accept after the land request is removed and before the await marker is, followed by a resume from a halted status, rewrites the board to Beam-Await and leaves the marker standing, so neither of card 10's detections fires and the dobby, its land episode recorded, never writes a request again
    fix: Card 10 adds `internal/pawlengine#awaitRun.accept` under Edit, and its Intent states that accept removes the await marker first, then the dobby-stop marker, then the land request, so a crash between the removals leaves either the land request standing, which the next wake accepts as today, or the marker gone with the ready record standing, which card 10's second detection settles; TestBeamAwait gains a row - await marker gone, land request still standing, ready record standing - that is accepted and Done.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/10-beam-await-entry.md:55
    summary: The new TestBeamAwait rows of cards 5 and 10 enter with the hold named PR-Review, a state no path reaches - the old accept wrote only Parent-Align and card 1 moves a PR-Review status before Beam-Await runs - while the one reachable retired-name hold, Parent-Align, is left to the existing row
    fix: In card 5's TestBeamAwait sentence (05-pawl-rows.md:70) and card 10's row "hold named PR-Review with ready standing" (10-beam-await-entry.md:55), name Parent-Align as the retired hold the row enters with, the name an old binary's accept wrote.
  - id: F8
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/03-pr-merge-state.md:41
    summary: Client.RequiredChecks matches required contexts only against check runs, so a required context a CI reports as a commit status reads missing, and lym beam pr status and the blocked cause name a reported check as absent
    fix: In card 3's RequiredChecks bullet (03-pr-merge-state.md:41), state that Client.RequiredChecks also reads Repositories.GetCombinedStatus at head and takes a context's conclusion from its commit status when no check run carries it, missing only when neither does; TestClient_Calls (03-pr-merge-state.md:47) gains a row whose required context is reported only as a commit status.
---

### [BLOCKING] Beam pull-request mode reads GitHub's rules on every hub

**Location:** `_lym/plan/14-beam-pr-status-close.md:28`, used by `_lym/plan/15-beam-ready-refusals.md:34`

**Issue:** Card 14's `PullRequestMode` closure answers whether GitHub's rules require a pull request "through the API's `RequiresPullRequest`", and card 13's `PullRequestSide` builds that API through `pullrequest.Open`.
`pullrequest.Open` refuses an origin that is not a GitHub repository and fails with no token (`internal/pullrequest/client.go:113-121`).
Card 15's refusal 5 reads `PullRequestMode` for every beam with no pull request recorded, and states nothing about its error.
So on a hub with no GitHub origin, or one without a usable token, `lym beam ready` cannot pass refusal 5 for any parent mode, `off` included.
The plan makes `lym beam ready` the only road to Finalize, so no beam on such a hub can land, against D3's acceptance criterion that `ready` writes the record when no refusal applies.
The resolver the plan reuses reads the rules only when the origin facts pass, and reads a token failure as no rule (`internal/landingshed/pullrequestmode.go:113-139`).
Card 15 calls `ResolvePullRequest` but reads the rules outside that gating, which `PATTERN-pull-request-mode`'s one resolver covers today.

### [MEDIUM] A second retired-row mechanism beside `loomrecipe.RetiredRows`

**Location:** `_lym/plan/01-shed-retired-rows.md:25`, `_lym/plan/05-pawl-rows.md:65`

**Issue:** The repository already carries retired rows per recipe in `loomrecipe.RetiredRows` (`internal/loomrecipe/recipekeys.go:20-31,154-162`).
Arming renames a status naming one through `loomCLI.renameRetiredRows` (`internal/loomcli/arm.go:288`).
`internal/loomrecipe/doc.go:39` and `internal/loomcli/doc.go:11` describe that table as each recipe's retired-row data.
Cards 1 and 5 add the recipe YAML's `retired` map, `Shed.Retired` and `loomrecipe.RetiredRowNames` with a different meaning (move and pause with side effects, at step).
Neither card names the existing mechanism.
The two docs and the near-namesake `RetiredRowNames` then mislead the next reader about which table a run's old row goes through.

### [MEDIUM] Old-flow text no card rewrites

**Location:** `_lym/plan/05-pawl-rows.md:65`

**Issue:** Text that no card's Intent touches stays false once the plan lands:

- `internal/battencli/dobbyreads.go:214` (`runTakesNoSlot`) and `internal/dobbyengine/doc.go:119` say a pawl run takes a slot "from Parent-Align on".
- `internal/loomrecipe/doc.go:43-46` lists the nine pawl rows, the PR-Review reroute that `lym beam reopen` asked for, and the hold from Parent-Align.
  Card 18 asserts that no text names `lym beam reopen` after card 6.
- `internal/beamcli/cli.go:55-61` (`mergeParentCmd`) tells the operator that "the beam's own landing brings the parent in at Parent-Align whether or not one ran", and `internal/beamcli/doc.go:92` says the same.
  With D3's refusal 3, the merge-in is now required before `lym beam ready`.
- `internal/loomcli/pawlwiring.go:1` describes the file as the pawl wiring "beyond the BeamDescribe row", which card 5 removes.

### [MEDIUM] An empty beam under a required parent has no way to `ready`

**Location:** `_lym/plan/15-beam-ready-refusals.md:34`, `_lym/plan/14-beam-pr-status-close.md:30`

**Issue:** Both cards call `ResolvePullRequest(slug, mode, "", BeamWithoutPullRequest, rulesRequired, origin, true)`.
For a beam with no commits beyond its parent, the resolver itself answers no pull request before any origin fact (`pullrequestmode.go:83-85`), and Finalize lands it so.
With `true` hard-coded, a `required` or rules-required parent gets a yes, and refusal 5 names `lym beam pr open <slug>`.
Card 13's `prOpen` refuses "a beam with no commits beyond its parent".
The two refusals point at each other, so the operator cannot land a beam whose members were all cancelled.
Card 13 already adds the `BeamHasCommits` seam that answers the fact.

### [MEDIUM] `judgeAwaitingRun` names an unreachable halt and leaves out the reachable one

**Location:** `_lym/plan/12-beam-describe-verb.md:44`

**Issue:** `judgeAwaitingRun`, reused by cards 13 to 15, passes a run at Beam-Await that is "running, blocked or paused".
It refuses one "not at Beam-Await or done", and leaves `failed` unstated.
`BeamAwait.Call` returns Done, Yield or an error (`internal/pawlengine/beamawait.go:71-94`), and a producer error persists as `failed` (`internal/shedengine/run.go:410`).
So Beam-Await never halts `blocked`, and its real halt is the case the card leaves to the implementer's reading.
D5's acceptance criterion says "a halted run at Beam-Await is accepted".
Card 12's test row "one blocked" pins a state no run reaches, and no row pins the failed halt.

### [LOW] Accept's removal order leaves a crash window that card 10's entry rule misses

**Location:** `_lym/plan/10-beam-await-entry.md:44`

**Issue:** `awaitRun.accept` removes the land request, the dobby-stop marker and the await marker in that order (`beamawait.go:306-310`).
A crash after the first removal, followed by a resume of a halted status, sees no land request and the await marker still standing.
The resume's status write (`run.go:317-318`) names Beam-Await on the board.
So neither card 10 detection fires: the board names no hold, and the await marker stands.
The dobby keeps its land episode recorded (`internal/dobbyengine/land.go:50`) and never writes the request again, which is the stall D11 sets out to remove.

### [LOW] TestBeamAwait rows enter with an unreachable PR-Review hold

**Location:** `_lym/plan/10-beam-await-entry.md:55`, `_lym/plan/05-pawl-rows.md:70`

**Issue:** The old accept wrote only Parent-Align as the hold (`beamawait.go:295`).
Card 1 moves a status at PR-Review or Beam-Describe before Beam-Await runs, and the entry's next status write names Beam-Await.
So the new rows that enter with the board naming PR-Review test a state no path produces.
The reachable retired-name hold is Parent-Align.

### [LOW] Required checks read from check runs only

**Location:** `_lym/plan/03-pr-merge-state.md:41`

**Issue:** `Client.RequiredChecks` collects the required contexts from the branch rules and matches each against `Checks.ListCheckRunsForRef` alone, reading `missing` when no check run carries it.
A required status context that a CI reports through the commit-status API is then shown as missing.
This shows in `lym beam pr status` and in the `blocked` cause of `lym beam ready`, which D6 asks to list each required check with its conclusion.

## Rejected

- guard-reach finding 4 (Parent-Align's conflict session not ended by the move): D13 settles that the move ends a live PR-Review session.
  Widening it to Parent-Align's fixer session is scope the answer key does not license, and the lens shows no evidence that such a session outlives the halt.
- guard-reach finding 5 (`lym beam merge-parent` after ready): D2 already settles a member filed after ready, which stays unstarted, blocks the landing and gets the cancel notice card 7 posts.
  A guard on the verb is scope the answer key does not license.
- repo-invariants F1 and F2: duplicates of F2 and F3.
- affected-set findings 3 and 4: merged into F3, which covers the same method gap.
- Handler candidate, `pawlengine`'s new `finalizeName` against `PATTERN-named-constants`: rejected.
  The repository declares the row name per package under its import graph (`landingshed/finalize.go:36`, `battenshed/innerrun_owner.go:15`, `fabricengine/replay.go:37`, `loomshed/loomshed.go:33`).
  The card keeps the precedent `parentAlignName` set, and the overview states the import reason.
- Handler candidate, card 5 removing `NewBeamDescribe` and `beamDescribeProducer` under an `Edit` of `beamdescribe.go` rather than member `Delete` glyphs: rejected.
  The overlay run shows `caller-uncovered` refuses the `Delete` shape, because the file-level `ShedProducer` assertion at `beamdescribe.go:67` references the type, and `redundant-file-target` forbids pairing the file glyph with them.
  The `Edit` of the file is the admissible shape.

## Focus notes

- D2, D10, D11 and their added entries: cards 5, 6, 7 and 10 carry the entry rule and the fork refusal.
  No card has the dobby write the land request again; F6 and F7 are the gaps found.
- Card order over the named symbols: every `Uses` and Intent dependency is created at or before its consumer.
  Card 5 builds with `beamwiring.go` uncalled, since unexported dead code compiles and no gate or test flags it.
- Attack surface, cards 7, 12, 13, 14, 16 and 17: each Intent carries a bound.
- Reachability, cards 8, 11, 13, 14 and 15: each prescribed refusal has a reaching input and test row.
  F1, F4 and F5 are the defects found on that path.
- Test economy: `TestEligibility_InterruptedPawlRows` covers `Eligibility`'s interrupted-reinvoke branch, which no arbiterstop test covers (`arbiterstop_test.go` tests blocked, batten and awaiting stops only).
  `TestWakeStamp` and `TestHubLandDeps_ReadyReads` cover new functions and closures.
  None is redundant.
- Edit-card shape: the re-signed members of cards 7, 11, 13 and 17 carry arrows.
  `NewBeamAwait` and `markReady` keep their signatures.
- `Extends`: each `doc.go` under `Extends` only gains text.
  No `Custom` card exists, and card 18's `Prosa` group targets file glyphs only.
- PATTERN changes: cards 11 and 13 fence all four declared entries.
- Board and filing: no card edits this task's entry or files a task.
  `lym board upsert` in card 12 names the beam's own entry.
- Verify lines: no card carries a `**Verify:**` line or a raw test command.

The fixes that add or pair targets (F1, F3, F6) were checked together with `lym loom validate-plan --overlay` against a `{}` baseline, and add no blocking finding.
