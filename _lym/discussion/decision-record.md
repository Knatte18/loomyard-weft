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
- `lym gate card-verify` lines, which run after the card's commit and are arbitrary commands.
- The darn writer's own `lym gate test` runs during its work.
- Webster's plan-level verify gate ladder (fixer fork first, cold fixer last), which keeps its trigger and order.

## Decisions

### D1 — One shared cold fixer core in `internal/gatefixer` (new)

The package owns the life of one cold fixer strand: spawn-or-attach, the bounded wait and the stop.
It is lifted from `internal/websterengine/coldfixer.go` and the spawn half of `internal/websterengine/verifyfixer.go`.

- The four new callers run the strand under the reed role `gate-fixer`, one constant declared once; a caller may name its own role, as webster does (D6).
  The strand starts through shuttle's `Starter`, so it is an interactive reed strand like every other, and it is registered with `internal/runproc` under its run, so ending the run's processes, by cancel, goto or reset, stops it.
- Its prompt is the shared cold frame stencil plus one caller body stencil (D5), read through `internal/stencilstore` at call time, with the parent, edit, pattern and friction directives rendered as the webster cold fixer renders them today.
  Skills go on the launch spec.
- Its shuttle output file is its reply file.
  The wait reads the fixer done once the reply exists, and dead when its newest turn ended without one, when the caller's timeout has passed since the spawn, or when its strand is gone, in that order, as `awaitColdFixer` does today.
- Every terminal result stops the strand through the stop seam; done also reads the reply and removes the shuttle run dir.
- The record of a live fixer, with strand, session, events path and offset, spawn time and the paths of its report and reply, is kept through a store the caller supplies (D8), saved before the wait, so a caller that restarts attaches to the live strand instead of spawning a second.
  A spawned strand whose run cannot be resolved, or whose record cannot be saved, is stopped at once.
- A fixer never judges the gate: the caller re-runs its own gate after every fixer and acts on that result alone.

Rationale: the webster code already solves spawn, wait, dead classification and crash attach.

### D2 — A row gate entry with a fixer holds its own evaluation while the fixer works

A row gate element whose `fixer` setting (D3) holds a model-spec has its closures wrapped.
The wrap applies to the entry's final `Gate` and `Forked` closures, after the row's own constructor has set them, so Plan-Write's lens-review `Forked` closure, which default runs evaluate, is wrapped too.

1. The wrapped closure runs the inner closure; a pass or a pending result is returned as it is.
2. On a failure it writes the findings text to a report file, spawns or attaches to the fixer with that report, and blocks until the fixer is terminal.
   No gate slot is held while it waits: the inner gate run has ended and released its slots, so the fixer's own gate calls can take them.
3. It runs the inner closure again and returns that result.
   When it still fails, its `Findings` text is the new findings followed by the fixer's reply, or by a note that the fixer ended dead with its reason and that any uncommitted change it left is in the worktree.
   The shuttle loop then re-prompts the writer exactly as today, counting the entry's attempt, with the row's own `Reprompt` line where it has one, such as Darn-Write's; Darn-Write's kept findings, which a resumed writer is pointed at, carry the same text.

The closure blocks rather than returning pending, as `loomshed.NewVerifyGate` already blocks for a whole test run; no `shuttleengine` change is made.
Only the wrapped entry is re-run after the fixer: an earlier entry's pass from the same evaluation stands, so a fixer's edit scope must stay inside what the wrapped gate itself checks.
On Darn-Write this holds because the verify gate fails on a dirty tree itself.

The wrapped entry also sets the `Expired` probe, read-only: it answers true when the store holds a fixer record from before an attach.
A driver restarted while the closure blocked thus finalizes the run at once, and the finalize evaluation attaches to that fixer, waits and judges; a still-failing gate halts the run gate-failed with the row's resume way forward, and the writer's unspent attempts are not used.

Bound: one fixer per failed evaluation, so at most the entry's `attempts` plus one fixers per attach of the writer's run, each bounded by the fixer timeout.
A finalize evaluation runs the wrapped closure too, so it can outrun the row's deadline by at most one fixer timeout plus one run of the inner gate.
While the closure blocks, the loop reads no pause, cover or liveness signal, as during a verify run today.

Rationale: the writer sits idle at its turn boundary, so the fixer edits a worktree nobody else is touching, and the writer hears only what the fixer left.

### D3 — Callers and config: one model-spec key per caller, `sonnet[medium]` by default

- Row gates opt in through a new optional key of a recipe gate element, `fixer`, naming a `<module>.<key>` setting that holds a model-spec; it resolves through `Env.Setting` at build, as the `parts` gate reads `tapestry.parts_max`.
  - Darn-Write's `verify` element: `fixer: darn.gate_fixer`, a new key in `internal/darnengine/template.yaml`.
  - Plan-Write's `plan` element: `fixer: tapestry.plan_gate_fixer`, a new key in `internal/loomengine/tapestry.yaml`.
  - Discussion-Write's `discussion` element: `fixer: loom.discussion_gate_fixer`, a new key in `internal/loomengine/template.yaml`.
- The agent-run gate call opts in through a new optional field `fixer` of a gate key in gate.yaml, a model-spec; the template in `internal/gateslot/template.yaml` sets it on the `card` key only.
- Each of the four defaults to `sonnet[medium]`; a model-spec carries both model and effort.
  An empty value turns that caller's fixer off, and the caller behaves exactly as today.
- One timeout, `fixer_timeout_min` in gate.yaml, default 30, bounds the wait of the four callers' fixers; webster's `fix-verify` keeps `recovery_timeout_min`.
- A `fixer` value that does not parse as a model-spec, or a `fixer` element key naming a setting that does not exist, refuses the build or the config load with the key and the way forward to set it.

Rationale: rows already look values up by setting key, and a gate key already owns its tiers and scope, so each caller's key sits with what it configures.

### D4 — The Webster card gate: an agent's gate call hands a red verdict to the fixer

The card gate is the Webster fork's own `lym gate test --gate card --card <id>` call; it is no shuttle row gate.
When an agent strand calls `lym gate test --gate <key>` and the key's `fixer` is set, a red verdict does not end the run:

1. Once the gate run has ended and released its slots, the detached runner (`internal/gatecli/detachrun.go`) spawns the fixer with the gate's failure report, the gate call and the card ids the call names.
2. It waits for the fixer, then runs the same gate call again, and the run's result is that re-run's, with the fixer's reply added to its report.
3. The caller's repeated calls join the run and answer `done: false` while it lasts.
   While the run is in its fixer or re-run phase, a call with the same key joins it whatever the tree identity it sees, since the fixer is what changed the tree; at any other phase the tree-identity rule of `joinable` stands.

Bound:

- Only a call from an agent strand hands off.
- A call from a fixer strand, of role `gate-fixer` or webster's `verify-fixer`, never hands off, and its detach key includes its strand name, so it never joins the run waiting on it.
- A detached run cancelled on its lapsed lease stops its fixer.
- At most one fixer per red call: a card gets at most the fork's self-fix cap plus one fixers, and a webster recovery strand's re-run of the card's gate adds one per red call of its own.

Rationale: the fork stays idle on its gate call while the fixer works in the same worktree, and it sees only the post-fix result.
The trigger is the gate key, never webster or the card scope, so the gate verb learns nothing of webster.

### D5 — The line between the fixer and the writer

The fixer is handed every failure the report lists; Go does not sort findings by code.
Its body stencil tells it to leave, and name in its reply, any failure whose fix would add, remove or change a decision, an acceptance criterion, a PATTERN change, a card or a card's scope, or a test's intended assertion, or would need a choice the decision record, plan or card does not make.
Go's re-run after the fixer then hands exactly what is left to the writer.

Every fixer: reads the report and the source behind each failure itself, fixes every failure it can in one pass, never deletes, skips or weakens a test, re-runs the failed packages or the validator first and then the whole gate call, and writes its reply as its final action.
Each caller's body sets its edit scope:

- Darn-Write: source only, never `_lym`; each fix committed `fix: <summary>`; the worktree left clean.
- Plan-Write: only the files under `_lym/plan/`; no commit, since the row commits them; re-checked with `lym loom validate-plan`.
- Discussion-Write: only the three discussion files; never an `### Added after Discussion` entry, a suffixed `## PATTERN changes` bullet or a carry-over block; no commit; re-checked with `lym loom validate-discussion`.
- Webster card gate: source in the worktree, reading the card and the uncommitted diff; no commit, since the fork commits the card.

Rationale: every report these gates write names each failure and its place, so the whole report goes to the fixer, which reads the source itself.

### D6 — Webster's `fix-verify` runs on the shared core

`lym webster fix-verify` keeps its verb, its trigger from Merriam, its fork-first ladder, its recovery-role model, its `verify-fixer` strand role, its `recovery_timeout_min` timeout and its `VerifyFixers` state records, which are the store it hands the core.
Its spawn, wait and reply handling call `internal/gatefixer` (new), and `internal/websterengine/coldfixer.go` is removed.
`contracts/stencils/webster/webster-prefix-verify-fix-cold.md` becomes the shared cold frame under `contracts/stencils/gatefix/` (new), and `contracts/stencils/webster/webster-body-verify-fix.md` is the webster caller's body, still shared with the fork prefix.
Webster's other cold strands that share the wait today use the wait from `internal/gatefixer` (new).

### D7 — Writers stop looping on their own validator when the fixer is on

- The Plan-Write and Discussion-Write stencils, the Discussion chair's included, render their self-check parts from whether the row's resolved gate entry carries a fixer: the recipe element names a `fixer` setting and the setting is not empty.
  With the fixer on, the writer does not run its validator in a loop: it ends its turn once its files are written, and the row gate and its fixer check them.
  The Discussion writer runs its lens review right after writing; the fork boilerplate no longer says the files passed the self-check, and the lens procedure's closing validator loop is dropped.
  With the fixer off, the text is today's.
- The Webster fork stencil says a failed card gate call has already been through a cold fixer whose reply its report carries, and that the fork fixes what is left under its self-fix cap.
- The darn writer stencil is unchanged: its gate runs during its work are its work, and its row gate's failures reach the fixer through D2.

Rationale: a writer that loops on its own validator never lets the gate fail, so without this change the fixer would never run for those two writers.

### D8 — Records and observability

- A row fixer's record, report and reply live under the row's run scratch directory in `.lym`, one set per gate entry, written through `internal/state` and tolerant of unknown keys.
- An agent-call fixer's record, report and reply live in the detached run's directory.
- Webster's records stay in its `VerifyFixers` state.
- The fixer's time shows in the gate entry's gate span, and the strand role `gate-fixer` lets `tools/tokencount` and the measurement of R1 (`agent-turns-effect`) count fixer runs and cost apart from the writer.

Rationale: the restart path of D1 and D2 needs the record on disk, and R1 measures the card gate loop after this lands.

## Constraints

- Fact: no shipped gate closure returns `Pending` today, and a gate evaluation runs every entry from the first (`internal/shuttleengine/wait.go`, `evaluateGate`); a pending hold would re-run Darn-Write's clean gate over the fixer's uncommitted edits, which is why D2 blocks instead.
- Fact: an in-process verify gate already blocks inside its closure for a whole test run, and fails on a dirty tree (`loomshed.NewVerifyGate`, `internal/loomshed/verifygate.go`).
- Fact: after an attach the loop evaluates no closure until the writer's next turn end, and asks only the entries' `Expired` probes (`internal/shuttleengine/wait.go`); D2's probe is the restart path.
- Fact: a detached gate run's key hashes the verb, the call directory and the words, and a call joins only a run whose tree identity, HEAD's tree and the dirty paths, matches (`internal/gatecli/detachfront.go`); D4 changes both for the fixer case only.
- Fact: Plan-Write sets its `plan` entry's `Forked` closure after the shared resolver returns (`internal/shedrecipe/entries_planwrite.go`).
- Fact: gate.yaml's `gates` is a per-name map, and a template entry merges key by key with a hub's (`internal/configengine/doc.go`, section "Per-name maps"), so a hub that tunes the `card` key's tiers keeps the template's `fixer`.

## Auto-mode assumptions

- `sonnet[medium]` is cheap enough and strong enough for test, lint and format failures, as the Webster and Code-Review fixers already run on it.
- A 30-minute fixer timeout covers a fixer's own full gate run on the largest gate key.
- The fixer's partial work after a dead end is better left in the worktree for the writer than reset.

## Open risks

- A writer whose prompt cache expires while a long fixer works pays a cache write on its next turn when the fixer fails; this cost shows only in runs where the fixer does not clear the gate.
- A pause that covers the run while a row fixer works can let the fixer reach its timeout, and the writer is then re-prompted once the cover lifts.
- Sonnet may clear fewer darn verify failures than the writer would; R1 measures it.

## Acceptance criteria

- D1: a fixer spawned for a failure is found done once its reply exists, dead when its turn ends without one, when the timeout passes or when its strand is gone, and is stopped on every terminal result.
- D1: a caller restarted while its fixer is live attaches to that strand and spawns none, and ending the run's processes stops a live fixer.
- D1, D6: `lym webster fix-verify` spawns, waits and records as before, through `internal/gatefixer` (new), under its own role, timeout and state records, with its prompt rendered from the shared frame and the webster body.
- D2: a row gate that fails with its fixer on spawns one fixer, re-runs the gate, and passes the row without re-prompting the writer when the re-run passes.
- D2: when the re-run still fails, the writer is re-prompted once, counting one attempt, with the remaining findings and the fixer's reply, or the dead note, in the findings.
- D2: a gate entry whose closure passes spawns no fixer, and a finalize evaluation runs the fixer at most once.
- D2: Plan-Write's forked plan gate is wrapped as its plain one is.
- D2: a run attached while a fixer record is held finalizes, waits on that fixer and passes or halts gate-failed on the re-run.
- D3: each of the four callers reads its own key, defaults to `sonnet[medium]`, and with an empty value behaves exactly as today.
- D3: an unparsable `fixer` value or a `fixer` element naming a missing setting is refused with the key and a way forward.
- D4: an agent strand's red `lym gate test --gate card` call spawns a fixer, re-runs the call, and answers with the re-run's result and the fixer's reply; a green call spawns none.
- D4: the fixer's own gate call runs in a detached run of its own, and the caller's repeated call joins its run through the fixer phase even when the fixer changed the dirty paths.
- D4: a gate call from a fixer strand, a call outside an agent strand and a call of a key without `fixer` never spawn a fixer; a cancelled detached run stops its fixer.
- D5: each caller's rendered fixer prompt names its edit scope, its commit rule, its re-check command and the failures it must leave for the writer.
- D7: the Plan-Write and Discussion-Write prompts render no validator loop with the fixer on and today's text with it off; the Webster fork prompt names the card-gate fixer's reply.
- D8: a row fixer's record is written under the run's `.lym` scratch directory and read back tolerant of unknown keys, and the fixer strand carries the role `gate-fixer`.

## PATTERN changes

- `PATTERN-gate-fixer` — adds: Clearing a failed gate with a cold agent: only `gatefixer`, one fixer per failed evaluation; Go's re-run of the gate judges, never the fixer; a fixer's own gate calls spawn none.

## Notes for the plan writer

- `RenderVerifyFixColdPrompt` in `internal/websterengine/render.go` is the renderer the shared frame replaces for webster.
- `resolveGateEntry` in `internal/shedrecipe/entries_gate.go` reads the new `fixer` element key; the wrap itself runs once each row constructor has finished its spec, `planWriteGateSpec` included.
