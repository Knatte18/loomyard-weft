---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/05-capture-run.md:49
    summary: >-
      RunCapture always holds the capture's kill-on-close named job, and cards 9 and 14 run RunCapture in the step's own process for the foreground capture (darn's capture: wait and loom's breakaway fallback), so on Windows the release at the capture's end terminates the row's own step process.
    fix: >-
      Card 5 (05-capture-run.md): drop the clause holding CaptureJobName(worktreeRoot) through proc.HoldNamedJob from RunCapture's sequence, and drop the job from its release list.
      Also drop internal/proc#HoldNamedJob from card 5's Uses.
      Card 7 (07-drift-verbs-cli.md): the hidden capture-child verb holds proc.HoldNamedJob(driftengine.CaptureJobName(worktreeRoot)) before it calls RunCapture, and releases the job only as its last act, after RunCapture returns.
      Add internal/proc#HoldNamedJob to card 7's Uses.
      Card 9 (09-drift-baseline-producer.md, the WaitCapture/fallback sentence at line 48) and card 14 (the RunCapture seam in driftDeps): the foreground run calls RunCapture in the step process with no named job.
      Its command then runs in the step's StartTied job tree and dies with the step, and card 13's teardown job kill reaches only a detached capture-child.
      The target moves were checked together with validate-plan --overlay; the combined run reports no new blocking finding.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/11-drift-verify-producer.md:45
    summary: >-
      Drift-Verify runs its dirty check and mechanical compare (steps 6 and 7) before its seat producer's live-chair probe, yet card 14 gives the row InterruptPolicyReinvoke on the premise that the row reattaches, so a re-step with the verify agent still live can halt dirty or return Done on a passing compare while the agent keeps editing, and that code lands uncompared.
    fix: >-
      Card 11 (11-drift-verify-producer.md): insert a step before step 2.
      DriftVerify.Call builds the verify table and calls Deps.Seats.Probe on it.
      A live chair skips steps 2 to 7 and goes straight to step 8, whose NewMultiLLMProducerSourced resumes the live chair, so its prepareFreshSpawn and ResetRuns do not run.
      TestDriftVerify gains a row: with gateLoopSeats reporting a live chair on entry, the producer resumes it, and no dirty check, verify gate or mechanical compare runs before it.
      Card 14's InterruptPolicies sentence at line 66 then stays true as written.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/11-drift-verify-producer.md:45
    summary: >-
      Step 6 re-runs the drift-verify gate whenever HEAD differs from AttemptHead, which also holds on a first entry (AttemptHead absent, behaviour unstated) and on a loom Finalize reroute where Parent-Align and the Code perch moved HEAD, so the gate runs over commits no Drift-Verify session made, while D6 runs it only when the session committed code.
    fix: >-
      Card 11, step 6: run step 6 only when the entry is the resume from this row's own dirty-tree or test-gate Stuck.
      That is, Status's last history entry is a Stuck of this row whose reason is one of those two halts, the same history test step 2 uses.
      Its verify half runs only when AttemptHead is found and differs from HEAD.
      On a first entry (AttemptHead absent) and on a reroute entry, step 6 is skipped.
      Replace the test row "an entry with HEAD past the attempt head and a failing verify gate is Stuck before any compare" with two rows:
      a resume from the row's test-gate halt with HEAD past the attempt head and a failing gate is Stuck before any compare;
      an entry whose last history entry is another row's (a reroute), with HEAD past the attempt head, runs no verify gate and goes on to the compare.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/14-drift-recipe-rows.md:16
    summary: >-
      Card 14 changes the loom edges but misses TestRouting_ProgressAt in internal/loomrecipe/progress_test.go, which pins Code-Bouncer OnDone to Describe and so fails under card 14's own gate, and the phase prose in internal/reportengine/doc.go, which lists each phase's rows beside the loomPhases and darnPhases tables card 14 edits.
    fix: >-
      Card 14: add internal/loomrecipe#TestRouting_ProgressAt and internal/reportengine/doc.go under Edit.
      The Intent says: TestRouting_ProgressAt's tail subtest expects Code-Bouncer OnDone Drift-Verify and Drift-Verify OnDone Describe;
      the doc's loom sentence names Drift-Baseline and Drift-Await under plan and Drift-Verify under webster;
      the darn sentence names Drift-Baseline under setup and Drift-Verify under review.
      Checked with validate-plan --overlay in the combined run, with no new blocking finding.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/05-capture-run.md:67
    summary: >-
      CheckMainCapture's dirty-tree refusal names lyx loom resume, which captures again, but at a Drift-Await halt that refusal comes from lyx drift capture and the resumed Drift-Await only reads records, so it halts again; and removing a tracked dirty path leaves a tracked deletion, which is still dirty.
    fix: >-
      Card 5, the dirty-tree branch of CheckMainCapture: the way forward is restoring the listed paths to HEAD or stashing them (never removing or committing them).
      The next step depends on the told status.
      At Drift-Baseline it is lyx loom resume, which captures again.
      At a Drift-Await halt it is lyx drift capture again, then lyx loom resume.
      Both keep lyx drift defer followed by lyx loom resume.
      TestCheckMainCapture asserts the wording at each of the two rows.
      Update the overview's Shared Decision "a dirty capture halt names cleaning" (00-overview.md:61) and card 9's dirty-tree sentence to the same wording.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/09-drift-baseline-producer.md:37
    summary: >-
      In loom, Drift-Baseline commits all of _lyx/drift on every way out right after it starts the detached capture, so the capture and capture-child logs are committed mid-write and stay tracked-dirty in the weft through the Plan perch, where a weft pull stops on tracked dirt; D3 and D4 commit only scope.json and baseline.md there and leave the capture log to Drift-Await.
    fix: >-
      Card 9: Deps.Commit takes the anchor-relative paths to commit.
      Without WaitCapture, DriftBaseline.Call commits only the scope record and baseline.md on every way out.
      With WaitCapture, and on the foreground fallback, it commits all of _lyx/drift once the capture has ended.
      TestDriftBaseline asserts the committed paths in each mode.
      Card 14's driftDeps builds Commit as fabricengine.CommitAnchoredPaths over the told paths.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/03-drift-verify-gate-key.md:8
    summary: >-
      Several cards list under Extends refs whose existing sentences they rewrite, which Extends forbids (it only adds), so the method of typing doc-list changes misses rewrites.
      Card 3 rewrites the template's keys comment and gateslot/doc.go:59; card 5 rewrites runproc/doc.go:4's role list and gateslot/doc.go:37's holder sentence; card 6 rewrites gateslot/doc.go:37; card 11 rewrites shedadapters/doc.go:112 and PATTERN-gate-slot.md's site sentences; card 13 rewrites pairteardown/doc.go step 3.
    fix: >-
      Move each rewritten ref from Extends to Edit.
      Card 3: internal/gateslot/template.yaml and internal/gateslot/doc.go.
      Card 5: internal/runproc/doc.go and internal/gateslot/doc.go.
      Card 6: internal/gateslot/doc.go.
      Card 11: internal/shedadapters/doc.go and pattern/PATTERN-gate-slot.md.
      Card 13: internal/pairteardown/doc.go, removing its now-empty Extends group.
      All these moves were checked together with validate-plan --overlay, with no new blocking finding.
  - id: F8
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lyx/plan/07-drift-verbs-cli.md:70
    summary: >-
      Card 7 prescribes two ErrRunsSpent refusal texts for lyx drift compare and lyx drift capture --side, one for the agent session (running) and one for a caller at a halt (blocked), but no named test reaches either.
    fix: >-
      Card 7: TestCapture_OutlivesCaller gains rows.
      Its drift.yaml also declares a fake compare command, and the test writes runs.json at verify_max_runs.
      lyx drift compare at a status running at Drift-Verify refuses with the agent wording (write verify-report.md and end the turn).
      At a status blocked at Drift-Verify it refuses with the caller wording (lyx loom resume spawns a fresh agent with a fresh budget).
      Each row asserts that runs.json is unchanged.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/07-drift-verbs-cli.md:53
    summary: >-
      Accept treats every running status at Drift-Verify as the agent session, but the status is also running during the row's own mechanical compare and pre-spawn checks, so a parent's or operator's accept in that window is recorded as by agent, and as unruled under full.
    fix: >-
      Card 7: a running status counts as the agent session only while the verify seat's chair is live.
      driftcli asks the seat runner's Probe over the verify table, and tells the verb through a VerbStatus field.
      A running status with no live verify chair refuses a caller's accept, naming the wait for the row's halt and the parent's ruling there.
      TestAcceptWithdraw gains that row.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/05-capture-run.md:74
    summary: >-
      StartCapture's two errors, a child that exits before its running record and a record wait the bound outlasts, name no way forward to the lyx drift capture caller, and no TestStartCapture row reaches the bound branch.
    fix: >-
      Card 5: both StartCapture errors end with a way forward.
      It reads the named child log, then runs lyx drift capture again, or lyx drift defer followed by lyx loom resume.
      TestStartCapture gains a row: a fake child that holds the lock and never writes its record past a shortened record-wait bound returns that error.
      Merged from the guard-reach and way-forward lenses.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/03-drift-verify-gate-key.md:8
    summary: >-
      Card 3 adds drift-verify to docs/dotnet-hub.md's example gates block but misses the same full-key example in internal/gatecli/doc.go's package doc.
    fix: >-
      Card 3: add internal/gatecli/doc.go under Extends, and the Intent adds the line drift-verify with tiers [0, 1] and scope diff to its example gates block.
      Checked with validate-plan --overlay in the combined run, with no new blocking finding.
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/07-drift-verbs-cli.md:75
    summary: >-
      Card 7 gives the single capture command two audiences (role with --side, operator without), but a cobra command carries one lyx.audience annotation, so the prescription cannot be built as written.
    fix: >-
      Card 7: capture carries the audience operator.
      Its Long names --side as the Drift-Verify agent's form.
      The verify stencil of card 8 already names lyx drift capture --side directly, so the role index needs no line.
      accept and withdraw keep role, and each Long names the parent's use at a halt.
---

### [BLOCKING] Foreground capture holds a kill-on-close job in the step's own process

**Location:** `_lyx/plan/05-capture-run.md:49`, `_lyx/plan/09-drift-baseline-producer.md:48`, `_lyx/plan/14-drift-recipe-rows.md:77`

**Issue:** `RunCapture` "holds the named job `CaptureJobName(worktreeRoot)` through `proc.HoldNamedJob` for its whole life" and releases it at the end.
Card 9 calls `Deps.RunCapture` for darn's `capture: wait` and for loom's breakaway fallback, and card 14 builds that seam over `RunCapture` in the step's process.
`holdNamedJobPlatform` (`internal/proc/namedjob_windows.go:49-63`) assigns the calling process to the job, and its release is `CloseHandle`.
`createKillOnCloseJob` sets `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE` (`namedjob_windows.go:25-27`), so closing the last handle terminates every process in the job, the row's step process among them.
Darn on Windows always takes this path, and the Models hub the task targets is a Windows machine.
Linux makes `HoldNamedJob` a no-op, so no gate of this hub sees the defect.

### [BLOCKING] Drift-Verify's mechanical steps run before the live-chair probe its Reinvoke policy relies on

**Location:** `_lyx/plan/11-drift-verify-producer.md:45`, `_lyx/plan/14-drift-recipe-rows.md:66`

**Issue:** `InterruptPolicyReinvoke` (`internal/loomshed/interruptpolicy.go:16-19`) holds only when re-calling the row reattaches to a live agent before it does anything else.
`MultiLLMProducer.Call` probes for the live chair (`internal/shedadapters/multillm.go:84-99`), but card 11 reaches that producer only at step 8, after the dirty check and the verify gate (step 6) and the mechanical compare (step 7).
A re-step with the verify agent still live therefore either halts dirty while the agent works on, or returns Done on a passing compare while the agent keeps changing code.
Either way it breaks D6's rule that no fix goes uncompared.
`TestInterruptPolicies_MatchEngineIdentity` requires Reinvoke of every non-Implementer row, so the fix belongs in card 11's order.

### [MEDIUM] Step 6's trigger is wider than D6's "when the session committed code"

**Location:** `_lyx/plan/11-drift-verify-producer.md:45`

**Issue:** `HEAD != AttemptHead` also holds on a loom Finalize reroute (`contracts/recipes/loom-recipe.yaml`, `Finalize` `on_reroute: Parent-Align`), where Parent-Align's merge and the Code perch's fixes moved HEAD.
The `drift-verify` gate then runs over commits no Drift-Verify session made, and a failure Stucks the row with "fix the tests".
On a first entry `AttemptHead` reports found false, and the card does not say what step 6 does then.

### [MEDIUM] Card 14 misses a routing test and the phase prose

**Location:** `_lyx/plan/14-drift-recipe-rows.md:16`

**Issue:** `internal/loomrecipe/progress_test.go:62-64` asserts `Code-Bouncer`'s `OnDone` is `Describe`, which card 14's recipe edit makes false, so the card's own gate fails.
`internal/reportengine/doc.go:129-131` lists every row of each phase for loom and darn, and gains no Drift row.

### [MEDIUM] Dirty-tree way forward does not progress from a Drift-Await halt, and "removing" does not clean

**Location:** `_lyx/plan/05-capture-run.md:67`, `_lyx/plan/00-overview.md:61`

**Issue:** The one refusal text is shared by the row and by `lyx drift capture` (card 7).
At a `Drift-Await` halt, `lyx loom resume` re-enters a row that only reads records (card 10) and never captures, so it halts again on the same missing or failed capture.
The listed paths are tracked dirty paths, and deleting one leaves a tracked deletion, which the next check reads as dirty again.

### [MEDIUM] Drift-Baseline commits capture logs mid-write in loom

**Location:** `_lyx/plan/09-drift-baseline-producer.md:37`, `_lyx/plan/14-drift-recipe-rows.md:77`

**Issue:** Card 9 commits all of `_lyx/drift` on every way out, and card 14 builds `Commit` over `DirRel()`.
In loom that commit runs right after the detached capture starts, so `logs/capture-<stamp>.log` and `logs/capture-child-<stamp>.log`, still open and growing, become tracked.
They stay tracked-dirty in the weft until Drift-Await commits again.
A weft pull that has to replay meets tracked dirt and stops (`internal/fabricengine/pull.go:354-362`, `weftDirtyStop`).
D3 has the row commit `scope.json` and `baseline.md`, and D4 gives the capture log to `Drift-Await` in loom.

### [MEDIUM] Extends used on refs the cards rewrite

**Location:** `_lyx/plan/03-drift-verify-gate-key.md:8`, `_lyx/plan/05-capture-run.md:21`, `_lyx/plan/06-compare-run.md:11`, `_lyx/plan/11-drift-verify-producer.md:15`, `_lyx/plan/13-capture-teardown.md:13`

**Issue:** The plan spec defines `Extends` as adding to a ref "changing nothing already there", which `extends-resign` and the edge rules rely on.
Each listed ref's change rewrites an existing sentence or inline list:
- `internal/gateslot/template.yaml`'s "The keys lyx reads" comment;
- `internal/gateslot/doc.go:59` and `:37`;
- `internal/runproc/doc.go:4`'s parenthesised role list;
- `internal/shedadapters/doc.go:112`;
- the "Every verify site …" and "The sites are …" sentences of `pattern/PATTERN-gate-slot.md`;
- `internal/pairteardown/doc.go`'s step 3.

The same misuse recurs across five cards, so it is filed once, against the method.

### [MEDIUM] Run-budget refusal wording has no reaching test

**Location:** `_lyx/plan/07-drift-verbs-cli.md:70`

**Issue:** The two `ErrRunsSpent` texts live in driftcli's `compare` and `capture --side` verbs.
`TestTakeRun` reaches only the counter's error, and `TestAcceptWithdraw` reaches only `CheckVerifyRow`.
No named test asserts either message, though the decision record's Constraints require a test to reach each refusal.

### [LOW] Accept tells the agent session from a caller by `running` alone

**Location:** `_lyx/plan/07-drift-verbs-cli.md:53`

**Issue:** The status is `running` through the row's own mechanical compare and pre-spawn checks too, so a parent's accept in that window is recorded with `by: agent`.
D7 requires each entry to carry who accepted it.
Under `full` the entry still halts for a ruling, so the effect is the wrong attribution only.

### [LOW] StartCapture's errors name no way forward, and one branch has no test

**Location:** `_lyx/plan/05-capture-run.md:74`

**Issue:** "A child that exits before that record is an error naming the child's log, and so is a start the bound outlasts."
From `lyx drift capture` neither error names a next step.
`TestStartCapture` has no row in which the record-wait bound expires.

### [LOW] Gate-key example in gatecli's doc missed

**Location:** `_lyx/plan/03-drift-verify-gate-key.md:8`

**Issue:** `internal/gatecli/doc.go:285-293` carries a full `gates:` example listing every key lyx reads, beside the `docs/dotnet-hub.md` block card 3 updates.

### [LOW] One command given two audiences

**Location:** `_lyx/plan/07-drift-verbs-cli.md:75`

**Issue:** `clihelp.AudienceAnnotation` takes one value per command (`internal/clihelp/annotations.go:20-31`), and the index lists a command only under that one (`internal/clihelp/index.go:54`).
`capture [--side]` is prescribed as `role` with `--side` and `operator` without, which no single annotation expresses.

## Rejected

- affected-set NIT, `PATTERN-config-layers.md`'s strict-callers list lacking `internal/driftengine`: the list already omits `darnengine` and `dobbyengine`, so this is existing staleness that no card of this plan introduced.
- way-forward F4, card 13's capture-kill error naming no way forward: a `TerminateNamedJob` failure is a propagated OS error, which refusal-spec exempts as transient raw I/O, and `endOwnedBeforeSession`'s own error is the precedent.
- Handler check, `drift.enabled` classified `CategoryNone`: `loomengine.ConfigSpec` records that a review budget of 0, the per-run off switch, is no guard, so an off switch at `CategoryNone` follows PATTERN-config-layers' precedent.
- Handler check, the Shared Decision "a dirty capture halt names cleaning" dropping D3's "committing": not a defect in direction. A commit moves HEAD off the merge base, so D3's step would never progress, and PATTERN-refusal-way-forward together with the record's own Constraint wins over D3's wording. F5 fixes only the remaining wording.
