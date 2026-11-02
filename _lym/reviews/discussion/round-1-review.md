---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D2 (restart paragraph) and Acceptance criteria (D2 attach bullet)
    summary: On Darn-Write a restarted driver's finalize evaluation fails the clean entry on the live fixer's uncommitted edits and halts gate-failed without ever reaching the wrapped verify entry, so it never waits on the fixer
    fix: Rewrite D2's restart paragraph so that a held live fixer record is waited on before any entry of the row is evaluated, not only inside the wrapped entry. For example, every entry of a row that carries a fixer entry is wrapped so that it first waits for a live fixer record of the row to turn terminal, and only then runs its own closure; the evaluation then runs from the first entry over the fixer's finished tree. Name the Darn-Write clean gate (`loomshed.NewCleanGate`, which fails on any uncommitted or untracked path) as the reason in the first Constraints fact, beside the pending case. Extend the D2 attach acceptance criterion to say the Darn-Write case, that a restart while the fixer holds uncommitted edits waits on the fixer and judges the re-run, never halting on the clean entry.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 (Expired probe paragraph), D1 (record bullet), D8 (first bullet)
    summary: The Expired probe answers true for any fixer record from before an attach, but terminal records stay in the run-wide scratch store, so a later restart, or a later visit of the row, finalizes the run at once and spends the writer's unspent attempts
    fix: In D1, state that the core marks a record terminal in the store when the wait ends, as webster's `VerifyFixers` records carry a status. In D2, change the probe to answer true only when the store holds a non-terminal record from before an attach. In D8, state that a row's record set is keyed to the row visit, or that a new spawn replaces the entry's set, so a goto, reset or row re-entry never reads an earlier visit's record. Add a D2 acceptance bullet that a restart with only a terminal record does not finalize.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 Bound (finalize overrun sentence)
    summary: D2's bound that a finalize outruns the row deadline by at most one fixer timeout plus one inner gate run is wrong, since the wrapped closure runs inner gate, fixer and inner gate again, and a blocked evaluation that crosses the deadline is followed by the deadline finalize
    fix: Replace the sentence with the real bound, that a row run can outrun its deadline by up to two fixer timeouts plus four inner gate runs, one blocked evaluation and the finalize after it, or bound it lower by a rule the record states, such as a finalize evaluation that finds a fixer already spent in this attach running the inner closure alone with no new fixer. Whichever is chosen, state it once in D2's Bound and make the D2 acceptance bullet on finalize match it.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 step 3
    summary: The fixer-or-re-run-phase join rule misses the time after the re-run ends, when a call compares against the pre-fixer tree and starts a fresh run that drops the fixer reply and can spawn a second fixer, and it states no bound on what a mismatched-tree call can join during the re-run
    fix: Rewrite D4 step 3 as follows. During the fixer phase, a same-key call joins whatever tree identity it sees. When the re-run starts, the run's recorded tree identity is rewritten to the tree the re-run starts over, and from then on the `joinable` tree rule stands against that identity, so a call after the re-run's exit joins and gets its result. Then add to D4's Bound what the fixer-phase join lets through and what bounds it. It lets through only a call with the same key (verb, run directory and words) while the fixer works. The caller sees the post-fix verdict, never a verdict over a tree it changed after the re-run began.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (record bullet), D4 Bound, D8 (second bullet)
    summary: For the card caller the fixer record lives in the detached run's directory, so a runner killed mid-fixer leaves the run unjoinable and the next call starts a fresh run that spawns a second fixer while the first still edits the worktree
    fix: In D8 move the agent-call fixer's record from the run directory to the call's key directory, which outlives a run. In D4 state that a fresh run of a key first attaches to a live fixer recorded there, and stops it if it cannot attach, before it spawns one. In D4's Bound, extend "a detached run cancelled on its lapsed lease stops its fixer" to a runner that died, whose fixer the next run of the key attaches to or stops. Add the killed-runner case to the D4 acceptance bullets.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D7 (first bullet)
    summary: D7 drops only the Discussion lens procedure's validator loop, but Plan-Write's twill directive also tells the writer to run its lens review once validate-plan passes and carries validate-plan self-check commands, so on the default lens path the Plan writer still loops and the plan fixer never runs
    fix: In D7's first bullet, name the Plan-Write lens procedure beside the Discussion one. With the fixer on, its self-check rows (`SelfCheckCommand` and `FinalSelfCheckCommand` in `internal/loomengine/twill.go`) and the run-the-lens-review-once-validate-plan-passes ordering render without the validator loop, and the writer runs its lens review right after writing. With the fixer off, today's text stands. Add that to the D7 acceptance bullet.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5 (Plan-Write bullet and leave-list)
    summary: Plan-Write's forked gate adds a lens-review presence check that only the writer's lens review should clear, yet the Plan-Write fixer may edit any file under _lym/plan/, so it can write the lens-review record and pass the gate with no review done, or spend a fixer on a failure it must not fix
    fix: Add to D5's leave-list that a missing or incomplete lens-review record is always left for the writer. Exclude the lens-review record from the Plan-Write fixer's edit scope. Name the re-check as `lym loom validate-plan --lens-review` for a forked run, so the fixer sees that check too. Add to the D5 acceptance bullet that the Plan-Write fixer prompt names the lens-review record as the writer's.
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D5
    summary: D5 opens a new routing edge where an unattended agent's edits pass the gate without the writer seeing them, and its limits are prompt rules only, yet D5 states no bound on what that edge lets through and what catches it
    fix: Add a Bound paragraph to D5. It states what the edge can let through. Go's re-run judges only what the gate checks, so a deleted, skipped or weakened test, an edit beyond the caller's edit scope, or a design-changing fix the fixer did not leave can pass the re-run and reach the writer's row unseen. It then names what bounds that for each caller. Darn-Write's fixer commits carry the `fix:` prefix and are judged by the run's later Code-Bouncer and Code-Burler-Review rows. Plan-Write's and Discussion-Write's edits are judged by the Plan and Discussion Bouncer and Burler-Review rows. The Webster fork reads the uncommitted diff before it commits the card. Say which of these limits Go enforces and which rest on the prompt.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7 (second bullet)
    summary: D7's Webster fork stencil text that a failed card gate call has already been through a cold fixer is unconditional, while D3 says an empty card fixer leaves the caller exactly as today
    fix: Make D7's second bullet conditional. The Webster fork stencil renders the card-gate fixer text only when the hub's `card` gate key carries a non-empty `fixer`, and today's text otherwise. Extend the D7 acceptance bullet to cover the fixer-off case of the Webster fork prompt.
  - id: F10
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 (step 1 and Bound bullets one and two)
    summary: D4 makes the detached runner decide the hand-off by the caller being an agent or a fixer strand, but the front strips the strand name from the runner's environment and the run request carries only verb, words and directory, and the record does not ground how a call is told apart
    fix: In D4's Bound, ground the two tests in the existing front. A call hands off only when it is a capped call, which `frontDecision` in `internal/gatecli/detachfront.go` decides from a non-empty strand name, no `--wait` and no enclosing slot, so an operator's call or a `--wait` call never hands off. The front parses the caller's role from its strand name and writes the role into the run request, since the runner's environment carries no strand name. A run whose role is `gate-fixer` or `verify-fixer` never hands off.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 Bound (fourth bullet)
    summary: The bound that a card gets at most the fork's self-fix cap plus one fixers rests on the fork's prompt cap, since a delivered run is never joinable and each repeated red call on an unchanged tree starts a new run and a new fixer, but the bullet reads as a Go bound
    fix: Reword D4's fourth Bound bullet to say that Go bounds one fixer per red call, and that the number of red calls per card rests on the fork stencil's self-fix cap, which Go does not enforce.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (first bullet) against D6 and its acceptance bullet
    summary: D1 registers every core strand with runproc, which webster's fix-verify fixer is not today, so a run's cancel, goto or reset would newly end it, while D6 and its acceptance bullet say fix-verify spawns, waits and records as before
    fix: In D6, state the change. Webster's `verify-fixer` strand is now registered with runproc under its run, so ending the run's processes stops it, beside `RemoveRecoveryStrands`. Reword the D1, D6 acceptance bullet from "as before" to name that one difference.
  - id: F13
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md Auto-mode assumptions (second bullet)
    summary: That a 30-minute fixer timeout covers a fixer's own full gate run on the largest gate key is a measurable fact left as an assumption, while the fixer runs the gate at least twice, failed packages first and then the whole call
    fix: Research the fact from the hub's gate run records, the per-key durations that `lym gate test`'s cap resolver already reads, for the card, darn-write, and the plan and discussion validators. Record the longest observed whole-call time, with its source, as a Constraints fact. Then either keep 30 as the default with that evidence, or set the default above twice the longest call, and drop the assumption line.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D1 (lines 33 and 40) and D6 (lines 126 and 127)
    summary: Citations of things the task removes carry no (removed) mark, against the discussion spec rule that a task marks what it creates or removes
    fix: "Add ` (removed)` after these citations. Mark `internal/websterengine/coldfixer.go` in D1 and D6, and `awaitColdFixer` in D1. Mark `contracts/stencils/webster/webster-prefix-verify-fix-cold.md` in D6, which moves under `contracts/stencils/gatefix/` (new). If the Notes' `RenderVerifyFixColdPrompt` is removed rather than rewritten, mark it too."
---

### [BLOCKING] Darn-Write restart halts on the clean entry before reaching the fixer

**Location:** `_lym/discussion/decision-record.md` D2, restart paragraph, and the D2 attach acceptance bullet.

**Issue:** D2 relies on the finalize evaluation reaching the wrapped entry to attach to the live fixer.
`evaluateGate` runs entries from the first and marks every entry after the first failure not reached (`internal/shuttleengine/wait.go:1636-1646`).
Darn-Write's first entry is `clean` (`contracts/recipes/darn-recipe.yaml:105-111`), and `NewCleanGate` fails on any uncommitted or untracked path (`internal/loomshed/cleangate.go:16-27`).
A fixer mid-edit leaves such paths, so the finalize halts gate-failed on `clean` findings over the fixer's half-done work, without ever waiting on it.
This is the hazard the record cites against a pending result, reached through D2's own restart path, and the acceptance bullet cannot hold on Darn-Write.

### [MEDIUM] The Expired probe reads terminal fixer records

**Location:** D2's probe paragraph, D1's record bullet and D8's first bullet.

**Issue:** The probe answers true for any record from before an attach.
No decision marks or clears a terminal record, and the row's records sit in the run-wide scratch directory that no goto, reset or re-entry clears.
A restart while a re-prompted writer is mid-turn finalizes at once (`wait.go:376-379`).
It then runs the inner gate over the writer's half-done edits and spawns a new fixer while the writer is live.
Webster attaches only to a last record whose status is empty (`internal/websterengine/verifyfixer.go:81`).
Reported by interactions, existing-paths and bound-breaker; merged here.

### [MEDIUM] D2's deadline overrun bound is understated

**Location:** D2 Bound, finalize sentence.

**Issue:** The wrapped closure runs the inner gate, the fixer, then the inner gate again (D2 steps 1-3).
The deadline exit finalizes and evaluates again (`wait.go:19`, `:420`), and while a closure blocks the loop reads no deadline.
So a run can exceed its deadline by up to two fixer timeouts plus four inner runs, not one timeout plus one run.
Reported by bound-breaker and interactions; merged here.

### [MEDIUM] The D4 join rule fails after the re-run and has no bound

**Location:** D4 step 3.

**Issue:** After the re-run writes its exit, the run is out of the fixer and re-run phases.
`joinable` then compares against the tree recorded before the fixer (`internal/gatecli/detachfront.go:316-326`).
A call that lands between `done: false` answers starts a fresh run, losing the fixer's reply and, if red, spawning a second fixer.
During the re-run, a same-key call over a tree changed after the re-run began joins and receives a verdict over a tree it no longer has.
The Decision states no bound for either case.
Reported by walkthrough and bound-breaker; merged here.

### [MEDIUM] A killed card runner allows a second concurrent fixer

**Location:** D1 record bullet, D4 Bound, D8 second bullet.

**Issue:** D1 promises that a restarted caller attaches to a live fixer instead of spawning another.
The card caller's store is the detached run's directory.
A runner killed hard leaves its run unjoinable because `runnerLive` is false, so the next call starts a new run with no record and spawns a second fixer beside the live first.
D4's bound covers only a lease-lapse cancel.
Reported by interactions and bound-breaker; merged here.

### [MEDIUM] D7 leaves Plan-Write's lens-procedure validator loop

**Location:** D7 first bullet.

**Issue:** Plan-Write's twill directive carries `SelfCheckCommand: "lym loom validate-plan"` and `FinalSelfCheckCommand: "lym loom validate-plan --lens-review"` (`internal/loomengine/twill.go:66-67`).
D7 names only the Discussion lens procedure's loop.
On the default lens path the Plan writer keeps looping, so the plan fixer never runs, which is the failure D7's rationale names.

### [MEDIUM] The Plan-Write fixer and the lens-review presence check

**Location:** D5 Plan-Write bullet and leave-list.

**Issue:** `NewLensReviewPlanGate` adds a refusal for a writer that never recorded its lens review (`internal/loomshed/gates.go:292-301`).
The Plan-Write fixer may edit any file under `_lym/plan/`, and D5's leave-list does not name the lens-review record.
The fixer can write the record and pass the forked gate with no review done.
A fixer that does not write it still spends a spawn on a failure that only the writer can clear.
Reported by existing-paths and walkthrough; merged here.

### [MEDIUM] D5 states no bound for the fixer's unattended edits

**Location:** D5.

**Issue:** The fixer is a new route by which edits pass a gate without the writer seeing them.
Its edit scope, its leave-list and "never weakens a test" are prompt rules only.
Go's re-run judges only what the gate checks, so a weakened or deleted test passes it.
The rubric's attack-surface question asks the Decision that introduces such an edge to state what it lets through and what bounds it, and D5 says neither.
The bounds exist downstream: Darn's Code-Bouncer and Code-Burler-Review rows, tapestry's Plan and Discussion bouncer and review rows (`contracts/recipes/darn-recipe.yaml`, `contracts/recipes/tapestry-recipe.yaml`), and the Webster fork reading the diff before its commit.
The record names none of them.

### [MEDIUM] The Webster fork text is unconditional

**Location:** D7 second bullet.

**Issue:** With the `card` key's `fixer` empty, D3 promises today's behaviour.
D7's fork stencil text would still say the failed call has been through a cold fixer whose reply its report carries.

### [LOW] D4's caller tests are not grounded in the gate front

**Location:** D4 step 1 and the first two Bound bullets.

**Issue:** The detached runner decides the hand-off by the caller's role.
The front strips `agentname.StrandNameEnv` from the runner's environment (`detachfront.go:130`), and the run request carries verb, args and directory only.
"A call from an agent strand" is already a fact of `frontDecision` (`detachfront.go:232-235`), but the record does not say so, nor how the role reaches the runner.
Walkthrough rated this MEDIUM, and bound-breaker and existing-paths read it as plan detail.
It is kept at LOW: the mechanism is a small seam, but the bound should cite the test it rests on.

### [LOW] The card fixer count rests on a prompt cap

**Location:** D4 Bound, fourth bullet.

**Issue:** A delivered run is never joinable (`detachfront.go:319`), so each repeated red call starts a new fixer.
The per-card total is bounded only by the fork stencil's self-fix cap.

### [LOW] runproc registration changes webster's fixer

**Location:** D1 first bullet against D6.

**Issue:** Today webster's cold fixer has no runproc record and is stopped only through `RemoveRecoveryStrands` (`internal/websterengine/strand.go:134`).
D1 registers every core strand, so a run's cancel, goto or reset would newly end it, while D6 says "as before".

### [LOW] The 30-minute timeout is an unresearched fact

**Location:** Auto-mode assumptions, second bullet.

**Issue:** Whether a fixer's own gate runs, failed packages and then the whole call, fit within 30 minutes on the largest key is a fact.
It can be measured from the gate run records the cap resolver already reads, so it is not a choice to assume.

### [LOW] Removed citations lack the (removed) mark

**Location:** D1 and D6.

**Issue:** D6 removes `internal/websterengine/coldfixer.go` and moves `contracts/stencils/webster/webster-prefix-verify-fix-cold.md`.
D1 lifts and removes `awaitColdFixer`.
The spec says a task marks what it creates or removes, yet these citations carry no mark.
`lym loom validate-discussion` passes, since the paths exist today, so this is the writer's convention rather than a gate failure.

## Rejected

- PATTERN-run-process-owner against D1's stop through the stop seam: registering a pane pid and stopping the strand through shuttle's stop already coexist for the PR-Review session (`internal/loomcli/sharedbootstrap.go:333-335`), and PATTERN-shuttle-provider-seam requires the stop verb, so this is no violation.
- D2's claim that an earlier entry's pass stands is not broken on its normal path. Plan-Write and Discussion-Write carry one gate entry each (`contracts/recipes/tapestry-recipe.yaml:122-124`, `:268-270`), and on Darn-Write the verify re-run fails on a dirty tree. The restart path that does break it is F1.
- Duplication between D2 and the Constraints: D2's clause that the closure blocks as `loomshed.NewVerifyGate` does repeats the second Constraints fact, but it carries the requirement that no `shuttleengine` change is made, which the plan writer needs, so nothing is relocated. D3's setting-merge fact appears only in Constraints, and D8's rationale is one line tied to D1 and D2's restart path.
- Board-entry rules: no Decision edits this task's own board entry or files or promotes a task. D3 touches config templates, and D7 touches stencils only.
