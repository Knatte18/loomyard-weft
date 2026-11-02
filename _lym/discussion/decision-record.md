# Decision record — cold-gate-fixer

## Goal

When a writer's gate fails, a fresh, cheap agent clears the mechanical failures instead of the writer fixing them at its full context.
One shared cold gate fixer serves four callers: the Darn-Write verify gate, the Plan-Write plan gate, the Discussion-Write discussion gate and the Webster card gate.
It is generalised from webster's cold verify-gate fixer (`lym webster fix-verify`), which becomes a caller of the same core.

## Scope

In:

- A shared package, `internal/gatefixer` (new), that spawns, attaches to, waits on and stops one cold fixer strand and renders its prompt.
- A wrap of a row gate entry that hands a failed evaluation to the fixer and re-runs the gate before the writer hears of it, on the Darn-Write, Plan-Write and Discussion-Write rows.
- The agent-run gate call `lym gate test --gate <key>` handing a red verdict to the fixer when the gate key names one, which serves the Webster card gate.
- Webster's `fix-verify` moved onto the shared core and the shared cold prompt frame.
- Config keys for each caller's fixer model-spec, and one fixer timeout for the four callers.
- The writer stencils that today loop on their own validator: Plan-Write's and Discussion-Write's, the Discussion chair's included, and the Webster fork's card-gate text.

Out:

- Any other gated row: Record-Write, Parts-Write, Parts-Plan-Write, the review fix rows, Drift-Verify and Describe keep today's re-prompt.
- Darn-Write's `clean` and `description` gates: a leftover file is the writer's to keep or delete, and the description is the writer's own account of its change.
  They get no fixer of their own; D2 only makes them wait on a live fixer of the row before they run.
- `lym gate card-verify` lines, which run after the card's commit and are arbitrary commands.
- The darn writer's own `lym gate test` runs during its work: config load refuses a `fixer` on the `darn-write` gate key, as on every key but `card` (D3).
- Webster's plan-level verify gate ladder (fixer fork first, cold fixer last), which keeps its trigger and order.

## Decisions

### D1 — One shared cold fixer core in `internal/gatefixer` (new)

The package owns the life of one cold fixer strand: spawn-or-attach, the bounded wait and the stop.
It is lifted from `internal/websterengine/coldfixer.go` (removed) and the spawn half of `internal/websterengine/verifyfixer.go`.

- The four new callers run the strand under the reed role `gate-fixer`; webster runs it under its `verify-fixer` role (D6).
  `internal/gatefixer` (new) declares both role constants, `verify-fixer` moved out of `internal/websterengine/verifyfixer.go`, owns them as the set of fixer roles and spawns under no other role, so a module that tests whether a strand is a fixer imports `gatefixer` alone.
- The strand starts through shuttle's `Starter`, so it is an interactive reed strand like every other.
  It is registered with `internal/runproc` under its worktree's pair slug, read from the worktree's fabric origin (`fabricengine.ReadOrigin`) as the detached gate runner's registration reads it, with a runproc role of its strand role's name, so every path that ends the pair's processes through runproc stops it, as does a goto (D8).
  A row or card caller with no pair slug, on the prime or outside a hub, spawns no fixer, and its gate failure reaches the writer as today; webster's fixer runs unregistered there, as today.
- Its prompt is the shared cold frame stencil plus one caller body stencil (D5), read through `internal/stencilstore` at call time, with the parent, edit, pattern and friction directives rendered as the webster cold fixer renders them today.
  Skills go on the launch spec.
- Its shuttle output file is its reply file.
  The wait reads the fixer done once the reply exists, and dead when its newest turn ended without one, when the caller's timeout has passed since the spawn, or when its strand is gone, in that order, as `awaitColdFixer` (removed) does today.
- Every terminal result stops the strand through the stop seam; done also reads the reply and removes the shuttle run dir.
- The record of a live fixer, with strand, session, events path and offset, spawn time and the paths of its report and reply, is kept through a store the caller supplies (D8), saved before the wait, so a caller that restarts attaches to the live strand instead of spawning a second.
  The record is marked terminal in the store with the fixer's done or dead status, as webster's `VerifyFixers` records carry a status: by a row or card caller once it holds a result judged over the fixer's finished tree, so a restart during the re-run still finds it live, and by webster when the wait ends, as today.
  Only a record with no terminal status is live, and only a live record is attached to.
  A spawned strand whose shuttle run cannot be resolved, whose runproc registration fails or whose record cannot be saved is stopped at once.
- A fixer never judges the gate: the caller re-runs its own gate after every fixer and acts on that result alone.

Rationale: the webster code already solves spawn, wait, dead classification and crash attach.

### D2 — A row gate entry with a fixer holds its own evaluation while the fixer works

A row gate element whose `fixer` setting (D3) holds a model-spec has its closures wrapped.
The wrap applies to the entry's `Gate` and `Forked` closures as the row's own constructor leaves them, and sets the entry's `Final` (below), so Plan-Write's lens-review `Forked` closure, which default runs evaluate, is wrapped too.

1. The wrapped closure runs the inner closure; a pass or a pending result is returned as it is.
2. On a failure at an evaluation the loop reaches at the writer's idle turn boundary, it writes the findings text to a report file, spawns or attaches to the fixer with that report, and blocks until the fixer is terminal.
   No gate slot is held while it waits: the inner gate run has ended and released its slots, so the fixer's own gate calls can take them.
3. It runs the inner closure again and returns that result.
   When it still fails, its `Findings` text is the new findings followed by the fixer's reply, or by a note that the fixer ended dead with its reason and that any uncommitted change it left is in the worktree.
   The shuttle loop then re-prompts the writer exactly as today, counting the entry's attempt, with the row's own `Reprompt` line where it has one, such as Darn-Write's; Darn-Write's kept findings, which a resumed writer is pointed at, carry the same text.

Only an evaluation at the writer's idle turn boundary spawns a fixer.
A final evaluation that runs closures, the deadline or liveness finalize reached whatever the writer is doing and the restart finalize below, spawns none.
A closure takes no argument, so the wrap tells the two apart through shuttle's `Final` closure: it sets the entry's `Final`, and `MayHold` so shuttle accepts `Final` on a must-pass entry, to a closure that waits on a live fixer record of the row visit (D8) until it is terminal and then runs the inner closure once.
That inner closure is the one the run would otherwise pick: the inner `Forked` closure when the row's spec runs forked, which the row decides at build, as Plan-Write does from `plan_lenses`, and the inner `Gate` otherwise.
The wrapped closures never return pending, so `MayHold` changes nothing else.
A boundary evaluation that itself finalizes, on a pass, a terminal failure or a spent budget, leaves a memoised verdict finalize reads without running a closure; every other finalize runs the closures, through `Final` (`handleGatedBoundary` and `evaluateGate`, `internal/shuttleengine/wait.go`).

The closure blocks rather than returning pending, as `loomshed.NewVerifyGate` already blocks for a whole test run; no `shuttleengine` change is made.
Only the wrapped entry is re-run after the fixer: an earlier entry's pass from the same evaluation stands, so a fixer's edit scope must stay inside what the wrapped gate itself checks.
On Darn-Write this holds because the verify gate fails on a dirty tree itself.

Every other gate entry of a row that carries a fixer entry is wrapped to wait only: when the store holds a live fixer record of the row, the wrap waits until that record is terminal and then runs its own closure; with no live record it runs its closure at once.
An evaluation runs entries from the first, so whichever entry it calls first does the waiting, and every entry is judged over the fixer's finished tree, never over its half-done edits.
On Darn-Write this keeps the `clean` entry, which goes first, from failing on a live fixer's uncommitted edits.
An entry that fails over a fixer the same evaluation waited on carries that fixer's reply or dead note in its findings, as step 3 says, and spawns no second fixer.
The record is marked terminal by the first entry that fails over the finished tree, or by the fixer entry once its closure returns.

The wrapped fixer entry also sets the `Expired` probe, read-only: it answers true when the store holds a live fixer record saved before the run's last attach, and false for a terminal one.
A driver restarted while a closure blocked thus finalizes the run at once, and the finalize evaluation waits on that fixer from its first entry, spawns none and judges every entry over the finished tree; a still-failing gate halts the run gate-failed with the row's resume way forward, and the writer's unspent attempts are not used.

Bound: one fixer per failed evaluation at the writer's idle turn boundary, none for a failure judged over a fixer the same evaluation waited on and none at a final evaluation.
A re-prompt send that fails busy, unlanded or paused re-evaluates at the boundary on a later tick without spending an attempt (`handleGatedBoundary`, `internal/shuttleengine/wait.go`), so the budget alone does not bound the fixers: the wrap keeps one record per fixer it spawns in the row visit's set (D8), and once they number the entry's `attempts` plus one it spawns none and returns the failure as it stands.
Each fixer is bounded by the fixer timeout.
While a closure blocks the loop reads no deadline, so a row run can outrun its deadline by up to one fixer timeout plus three runs of the inner gate: an evaluation that starts before the deadline blocks across it through a run, a fixer and a re-run, and the deadline finalize after it runs the inner closure once more through `Final`, spawning no fixer.
While the closure blocks, the loop reads no pause, cover or liveness signal, as during a verify run today.

Rationale: a fixer is spawned only while the writer sits idle at its turn boundary, so the fixer edits a worktree nobody else is touching, and the writer hears only what the fixer left.

### D3 — Callers and config: one model-spec key per caller, `sonnet[medium]` by default

- Row gates opt in through a new optional key of a recipe gate element, `fixer`, naming a `<module>.<key>` setting that holds a model-spec; it resolves through `Env.Setting` at build, as the `parts` gate reads `tapestry.parts_max`.
  - Darn-Write's `verify` element: `fixer: darn.gate_fixer`, a new key in `internal/darnengine/template.yaml`.
  - Plan-Write's `plan` element: `fixer: tapestry.plan_gate_fixer`, a new key in `internal/loomengine/tapestry.yaml`.
  - Discussion-Write's `discussion` element: `fixer: loom.discussion_gate_fixer`, a new key in `internal/loomengine/template.yaml`.
- The agent-run gate call opts in through a new optional field `fixer` of a gate key in gate.yaml, a model-spec, accepted on the `card` key only: config load refuses `fixer` on any other key, naming the key and the way forward to remove the field, so no other agent's in-work gate call hands off.
  The template in `internal/gateslot/template.yaml` sets it on the `card` key.
- Each of the four defaults to `sonnet[medium]`; a model-spec carries both model and effort.
  An empty value turns that caller's fixer off, and the caller behaves exactly as today.
- One timeout, `fixer_timeout_min` in gate.yaml, default 30, bounds the wait of the four callers' fixers; webster's `fix-verify` keeps `recovery_timeout_min`.
- A `fixer` value that does not parse as a model-spec, or a `fixer` element key naming a setting that does not exist, refuses the build or the config load with the key and the way forward to set it.

Rationale: rows already look values up by setting key, and a gate key already owns its tiers and scope, so each caller's key sits with what it configures.

### D4 — The Webster card gate: an agent's gate call hands a red verdict to the fixer

The card gate is the Webster fork's own `lym gate test --gate card --card <id>` call; it is no shuttle row gate.
When an agent strand calls `lym gate test --gate <key>` and the key's `fixer` is set, a red verdict does not end the run:

1. Once the gate run has ended and released its slots, the detached runner (`internal/gatecli/detachrun.go`) spawns the fixer with the gate's failure report, the gate call and the card ids the call names.
   Before its gate run, a run of a key attaches to a live fixer the key directory records (D8) and waits for it, or stops it when it cannot attach, so it never runs the gate over a fixer's half-done edits or spawns a second fixer beside it.
2. It waits for the fixer, then runs the same gate call again, and the run's result is that re-run's, with the fixer's reply added to its report.
3. The caller's repeated calls join the run and answer `done: false` while it lasts.
   During the fixer phase, a call with the same key joins whatever tree identity it sees, since the fixer is what changes the tree.
   When the re-run starts, the run's recorded tree identity is rewritten to the tree the re-run starts over, and from then on the tree-identity rule of `joinable` stands against it, so a call after the re-run's exit joins and gets its result.

Bound:

- Only a capped call hands off: `frontDecision` in `internal/gatecli/detachfront.go` caps a call only when its strand name is set, it gives no `--wait` and it runs in no enclosing slot, so an operator's call or a `--wait` call never hands off.
- The runner's environment carries no strand name, so the front parses the caller's role from its strand name and writes the role into the run request.
  A run whose role is in the set of fixer roles `internal/gatefixer` (new) owns (D1), numbered or not as reed numbers a held role (`gate-fixer-2`), never hands off: the front tests the role against each role of that set through `agentname.MatchesRole`, so `internal/gatecli` imports nothing of `internal/websterengine`.
  Such a fixer strand's call keys its detached run with its strand name, so it never joins the run waiting on it.
- The card fixer registers with runproc under the owner the detached runner registers under, the caller's pair slug read through `fabricengine.ReadOrigin`; a call with no such slug, on the prime or outside a hub, hands off nothing, and its red verdict ends the run as today.
- The run request carries no parent name, so the card fixer's prompt renders the parent directive's no-parent variant: a failure it would ask about is one it leaves and names in its reply (D5), which the fork reads.
- The fixer-phase join lets through only a call with the same key, the verb, run directory and words, while the fixer works.
  The caller sees the post-fix verdict, never a verdict over a tree it changed after the re-run began.
- A detached run cancelled on its lapsed lease stops its fixer, and the fixer of a runner that died is attached to or stopped by the next run of the key, since its record outlives the run.
- Go bounds one fixer per red call.
  The number of red calls per card rests on the fork stencil's self-fix cap, which Go does not enforce; a webster recovery strand's re-run of the card's gate adds one fixer per red call of its own.

Rationale: the fork stays idle on its gate call while the fixer works in the same worktree, and it sees only the post-fix result.
The trigger is the gate key, never webster or the card scope, and the guard reads the fixer roles from `gatefixer`, so the gate verb learns nothing of webster.

### D5 — The line between the fixer and the writer

The fixer is handed every failure the report lists; Go does not sort findings by code, but for one case: a forked Plan-Write evaluation whose only blocking finding is the missing lens review spawns no fixer, and the writer is re-prompted with it at once.
Its body stencil tells it to leave, and name in its reply, any failure whose fix would add, remove or change a decision, an acceptance criterion, a PATTERN change, a card or a card's scope, or a test's intended assertion, or would need a choice the decision record, plan or card does not make.
A missing or incomplete lens review record is always left for the writer.
Go's re-run after the fixer then hands exactly what is left to the writer.

Every fixer: reads the report and the source behind each failure itself, fixes every failure it can in one pass, never deletes, skips or weakens a test, re-runs the failed packages or the validator first and then the whole gate call, and writes its reply as its final action.
Each caller's body sets its edit scope:

- Darn-Write: source only, never `_lym`; each fix committed `fix: <summary>`; the worktree left clean.
- Plan-Write: only the files under `_lym/plan/` but the lens review record, which is the writer's; no commit, since the row commits them; re-checked with `lym loom validate-plan`, or `lym loom validate-plan --lens-review` for a forked run.
- Discussion-Write: only the three discussion files; never an `### Added after Discussion` entry, a suffixed `## PATTERN changes` bullet or a carry-over block; no commit; re-checked with `lym loom validate-discussion`.
- Webster card gate: source in the worktree, reading the card and the uncommitted diff; no commit, since the fork commits the card.

Bound: the fixer is a route by which an unattended agent's edits pass a gate without the writer seeing them.
Go's re-run judges only what the gate checks, so a deleted, skipped or weakened test, an edit beyond the caller's edit scope, or a design-changing fix the fixer did not leave can pass the re-run and reach the writer's row unseen.
Go enforces the re-run, the fixer's role and timeout, and that the later rows below run.
Whether those rows review rests on the hub's config, since a Bouncer whose max-bounces setting is 0, such as loom.yaml's `code_max_bounces`, settles approved at once.
The edit scope, the leave-list, the test rule and the commit rule rest on the prompt.
What judges the fixer's edits, per caller:

- Darn-Write: its commits carry the `fix:` prefix, and the run's later Code-Bouncer and Code-Burler-Review rows judge them.
- Plan-Write and Discussion-Write: the Plan and Discussion Bouncer and Burler-Review rows.
- Webster card gate: the fork reads the uncommitted diff before it commits the card, by its stencil.

Rationale: every report these gates write names each failure and its place, so the whole report goes to the fixer, which reads the source itself.

### D6 — Webster's `fix-verify` runs on the shared core

`lym webster fix-verify` keeps its verb, its trigger from Merriam, its fork-first ladder, its recovery-role model, its `verify-fixer` strand role, its `recovery_timeout_min` timeout and its `VerifyFixers` state records, which are the store it hands the core.
Its spawn, wait and reply handling call `internal/gatefixer` (new), and `internal/websterengine/coldfixer.go` (removed) is deleted.
One thing changes: webster's `verify-fixer` strand is now registered with `internal/runproc` under its worktree's pair slug (D1), so ending the pair's processes stops it, beside `RemoveRecoveryStrands`.
`contracts/stencils/webster/webster-prefix-verify-fix-cold.md` (removed) becomes the shared cold frame under `contracts/stencils/gatefix/` (new), and `contracts/stencils/webster/webster-body-verify-fix.md` is the webster caller's body, still shared with the fork prefix.
Webster's other cold strands that share the wait today use the wait from `internal/gatefixer` (new).

### D7 — Writers stop looping on their own validator when the fixer is on

- The Plan-Write and Discussion-Write stencils, the Discussion chair's included, render their self-check parts from whether the row's resolved gate entry carries a fixer: the recipe element names a `fixer` setting, the setting is not empty and the worktree has a pair slug (D1).
  With the fixer on, the writer does not run its validator in a loop: it ends its turn once its files are written, and the row gate and its fixer check them.
  The Discussion writer runs its lens review right after writing; the fork boilerplate no longer says the files passed the self-check, and the lens procedure's closing validator loop is dropped.
  The Plan-Write lens procedure changes the same way: its self-check rows (`SelfCheckCommand` and `FinalSelfCheckCommand` in `internal/loomengine/twill.go`) and its order to run the lens review once `lym loom validate-plan` passes render without the validator loop, and the Plan writer runs its lens review right after writing.
  A plan or discussion fixer may then edit files its writer's lens review read; that staleness is accepted for both writers, since the fixer's edits are mechanical and D5's Bound names what judges them, and the plan gate checks the lens review record for presence only.
  With the fixer off, the text is today's.
- When the hub's `card` gate key carries a non-empty `fixer` and the fork's worktree has a pair slug (D4), the Webster fork stencil says a failed card gate call has already been through a cold fixer whose reply its report carries, and that the fork fixes what is left under its self-fix cap; otherwise the text is today's.
- The darn writer stencil is unchanged: its gate runs during its work are its work, and its row gate's failures reach the fixer through D2.

Rationale: a writer that loops on its own validator never lets the gate fail, so without this change the fixer would never run for those two writers.

### D8 — Records and observability

- A row fixer's record, report and reply live under the row's run scratch directory in `.lym`, one set per row visit and gate entry holding a record per fixer the entry spawned, written through `internal/state` and tolerant of unknown keys.
  A row visit is keyed by the row's name, the number of entries in the run's status `History` and the newest entry's `Repeats`, read from the run's status file (`shedrun.StatusFile`) when the row's call starts.
  A call that ends appends a history entry or folds into the newest one, and a goto appends one (`internal/shedengine/run.go`, `internal/shedengine/goto.go`), while an interrupted step writes none.
  A crash re-step of the same row, resume re-stepping an interrupted step, therefore keeps the same set, so the `Expired` probe and the wait-only entries see the live fixer, and so does a resume of a failed stop that appended no history entry.
  A goto, a resume of a halt that appended a history entry and every fresh row entry start an empty set and never read an earlier visit's record.
- A goto ends each role of `gatefixer`'s fixer role set (D1) under the pair through `runproc.EndRole` before it moves the run, so no fixer of the visit it leaves, a card fixer or webster's `verify-fixer` included, keeps editing the worktree under the next row.
- An agent-call fixer's record lives in the call's key directory, which outlives a run, so the next run of the key finds the live fixer of a runner that died (D4); its report and reply live in the detached run's directory, and the record holds their paths.
- Webster's records stay in its `VerifyFixers` state.
- The fixer's time shows in the gate span of the entry that waited on it, the fixer entry's on the normal path, and the strand role `gate-fixer` lets `tools/tokencount` and the measurement of R1 (`agent-turns-effect`) count fixer runs and cost apart from the writer.

Rationale: the restart path of D1 and D2 needs the record on disk, and R1 measures the card gate loop after this lands.

## Constraints

- Fact: no shipped gate closure returns `Pending` today, and a gate evaluation runs every entry from the first (`internal/shuttleengine/wait.go`, `evaluateGate`).
  Darn-Write's first entry is the clean gate, `loomshed.NewCleanGate` (`internal/loomshed/cleangate.go`), which fails on any uncommitted or untracked path.
  A pending hold would re-run it over the fixer's uncommitted edits, which is why D2 blocks instead, and a restart's finalize evaluation would reach it before the wrapped entry, which is why D2's other entries wait on a live fixer first.
- Fact: an in-process verify gate already blocks inside its closure for a whole test run, and fails on a dirty tree (`loomshed.NewVerifyGate`, `internal/loomshed/verifygate.go`).
- Fact: after an attach the loop evaluates no closure until the writer's next turn end, and asks only the entries' `Expired` probes (`internal/shuttleengine/wait.go`); D2's probe is the restart path.
- Fact: a detached gate run's key hashes the verb, the call directory and the words, and a call joins only a run whose tree identity, HEAD's tree and the dirty paths, matches (`internal/gatecli/detachfront.go`); D4 changes both for the fixer case only.
- Fact: Plan-Write sets its `plan` entry's `Forked` closure after the shared resolver returns (`internal/shedrecipe/entries_planwrite.go`).
- Fact: gate.yaml's `gates` is a per-name map, and a template entry merges key by key with a hub's (`internal/configengine/doc.go`, section "Per-name maps"), so a hub that tunes the `card` key's tiers keeps the template's `fixer`.
- Fact: the longest whole gate call in the hub's gate run records took 594 s, a `darn-write` call, measured from its first slot wait to its last tier end over the records grouped by call id (`_bolt/.lym/gate/runs`, read on 2026-11-02, every record on disk being from that day).
  No `card` call is recorded there, and the plan and discussion validators are in-process checks that write no run record.
  A fixer runs the gate at least twice, so `fixer_timeout_min`'s default of 30 stays above twice the longest call.

## Auto-mode assumptions

- `sonnet[medium]` is cheap enough and strong enough for test, lint and format failures, as the Webster and Code-Review fixers already run on it.
- The fixer's partial work after a dead end is better left in the worktree for the writer than reset.

## Open risks

- A writer whose prompt cache expires while a long fixer works pays a cache write on its next turn when the fixer fails; this cost shows only in runs where the fixer does not clear the gate.
- A pause that covers the run while a row fixer works can let the fixer reach its timeout, and the writer is then re-prompted once the cover lifts.
- Sonnet may clear fewer darn verify failures than the writer would; R1 measures it.
- No `card` gate call is on record, so the 30-minute timeout is unmeasured for the card key.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - d4-role-guard-teaches-gate-webster-role (class consistency, severity MEDIUM, open in rounds 2)
  - d8-row-visit-identity-undefined (class design, severity MEDIUM, open in rounds 2)
  - finalize-off-idle-boundary-spawns-fixer (class design, severity BLOCKING, open in rounds 2)
  - goto-does-not-stop-live-fixer (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a fixer spawned for a failure is found done once its reply exists, dead when its turn ends without one, when the timeout passes or when its strand is gone, and is stopped on every terminal result.
- D1: a caller restarted while its fixer is live attaches to that strand and spawns none, ending the pair's processes stops a live fixer, and a row or card caller with no pair slug spawns no fixer.
- D1, D6: `lym webster fix-verify` spawns, waits and records through `internal/gatefixer` (new), under its own role, timeout and state records, with its prompt rendered from the shared frame and the webster body; its one change is that its strand is registered with runproc, so ending the pair's processes stops it.
- D2: a row gate that fails with its fixer on and a pair slug spawns one fixer, re-runs the gate, and passes the row without re-prompting the writer when the re-run passes.
- D2: when the re-run still fails, the writer is re-prompted once, counting one attempt, with the remaining findings and the fixer's reply, or the dead note, in the findings.
- D2: a gate entry whose closure passes spawns no fixer, and a finalize evaluation off the writer's idle turn boundary, the deadline finalize included, spawns no fixer and only waits on a live one, so a row run outruns its deadline by at most one fixer timeout plus three runs of the inner gate.
- D2: Plan-Write's forked plan gate is wrapped as its plain one is.
- D2: a run attached while a live fixer record is held finalizes, waits on that fixer before it evaluates any entry, and passes or halts gate-failed on the re-run; on Darn-Write, a restart while the fixer holds uncommitted edits waits on the fixer and judges the re-run, never halting on the `clean` entry over a live fixer's edits; when the fixer ends leaving edits uncommitted, the `clean` findings carry its reply or dead note.
- D2: a run attached while only a terminal fixer record is held does not finalize.
- D3: each of the four callers reads its own key, defaults to `sonnet[medium]`, and with an empty value behaves exactly as today.
- D3: an unparsable `fixer` value, a `fixer` element naming a missing setting and a `fixer` on a gate.yaml key other than `card` are refused with the key and a way forward.
- D4: an agent strand's red `lym gate test --gate card` call spawns a fixer, re-runs the call, and answers with the re-run's result and the fixer's reply; a green call spawns none.
- D4: the fixer's own gate call runs in a detached run of its own, and the caller's repeated call joins its run through the fixer phase even when the fixer changed the dirty paths, and a call after the re-run's exit gets the re-run's result.
- D4: a gate call from a strand of any role in `gatefixer`'s fixer role set, a numbered one included, a call that is not capped, a call with no pair slug and a call of a key without `fixer` never spawn a fixer; a cancelled detached run stops its fixer, and the card fixer's prompt carries the no-parent directive.
- D4: a run of a key whose earlier runner died while its fixer was live attaches to that fixer or stops it before its gate run, and spawns no second fixer beside it.
- D5: each caller's rendered fixer prompt names its edit scope, its commit rule, its re-check command and the failures it must leave for the writer, and the Plan-Write fixer prompt names the lens review record as the writer's; a forked Plan-Write evaluation that fails only on the missing lens review spawns no fixer.
- D7: the Plan-Write and Discussion-Write prompts, their lens procedures included, render no validator loop and a lens review right after writing with the fixer on, and today's text with it off.
- D7: the Plan-Write and Discussion-Write prompts render today's text in a worktree with no pair slug, whatever the `fixer` setting.
- D7: the Webster fork prompt names the card-gate fixer's reply when the `card` key's `fixer` is set and the worktree has a pair slug, and renders today's text otherwise.
- D8: a row fixer's record is written under the run's `.lym` scratch directory and read back tolerant of unknown keys, and the fixer strand carries the role `gate-fixer`.
- D8: a crash re-step of a row and a resume of a failed stop that appended no history entry keep the visit's record set, and a goto, a resume of a halt that appended one and a fresh row entry start an empty one.
- D2, D8: a held re-prompt send that re-evaluates at the boundary spawns no fixer once the visit's fixers number the entry's `attempts` plus one.
- D8: a goto past a row with a live fixer leaves no fixer running.

## PATTERN changes

- `PATTERN-gate-fixer` — adds: Clearing a failed gate with a cold agent: only `gatefixer`, one fixer per failed evaluation; Go's re-run of the gate judges, never the fixer; a fixer's own gate calls spawn none.

## Notes for the plan writer

- `RenderVerifyFixColdPrompt` (removed) in `internal/websterengine/render.go` is the renderer the shared frame replaces for webster.
- `resolveGateEntry` in `internal/shedrecipe/entries_gate.go` reads the new `fixer` element key; the wrap itself runs once each row constructor has finished its spec, `planWriteGateSpec` included.
