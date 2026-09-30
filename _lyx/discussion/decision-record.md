# Decision record — refusal-audit

## Goal

Every refusal a `lyx webster`, `lyx shed` or `lyx loom` run can reach leaves a documented way forward, so no run needs a hub code fix and deploy to continue.
A guard that does not protect correctness warns and records instead of halting.
A false stop is treated as seriously as a wrong landing.

## Scope

In:

- Every refusal or halt reachable through the verbs of three modules:
  - webster: `internal/websterengine`, `internal/webstercli`;
  - shed: `internal/shedengine`, `internal/shedverbs`, `internal/shedrun`, `internal/shedcli`;
  - loom: `internal/loomcli`, plus the packages its gates are raised in (`internal/loomshed`, `internal/loomengine`, `internal/preflight`, `internal/preflightshed`, and the Webster adapter in `internal/shedadapters`).
- Gate families named by the board: preflight, coherence, fingerprint, fork audit, head-SHA, report-exists.
- Classification of every refusal, a fix for each one without a way forward, and a test for each reaching it from a realistic state.
- A refusal table, one section per module, in a new durable contract doc.
- The fork-audit disposition change for policy-class findings.
- Two new verbs: `lyx shed goto` and `lyx webster rebaseline`.
- A new CONSTRAINTS.md invariant and the doc updates it implies.

Out:

- Refusals in landing (`internal/landingshed`), batten, orch, fabric, burler and board verbs.
  They are a separate audit under the same principle, not part of this task.
- The fast-forward merge-in limit that `webster-parent-merge-midrun` left in place: it already names a git recourse, which counts as a way forward under the definition below.

## Decisions

### D1 — What counts as a way forward

Decision: a refusal has a way forward when its message names one of:

1. a `lyx` verb that makes progress from the refused state;
2. the state transition the run takes by itself (a re-step, the next Master retry, a failure ladder rung);
3. a plain `git` command on the warp worktree, when no `lyx` verb covers it.

Hand-editing a lyx-owned state file (`state.json`, `status.json`, `seed.json`, report files) never counts.
A code fix plus deploy never counts.
Waiting for a lock holder counts when the message names how to see the holder (`status`).

Rationale: every 2026-09-30 wedge had a "way forward" only through a hub code fix or hand-edited state.
The definition makes the audit decidable per row.

### D2 — Refusal classification

Decision: every refusal is placed in one class, and the class fixes its disposition:

| Class | Meaning | Disposition |
|---|---|---|
| correctness halt | proceeding could land wrong code, forge a judgment, or corrupt run state | halt; message names the way forward |
| policy guard | steering or hygiene; proceeding lands nothing wrong | warn and record; never halt |
| transient | a condition that clears by itself or by a named action (lock held, merge in progress, I/O) | halt without mutating state; message names the retry |
| wiring guard | nil deps, invalid producer list, empty paths — unreachable from any on-disk state a run can produce | grouped table row; no per-row test |

Raw I/O failures (`stat`, `mkdir`, `write`) form one grouped transient row.

Rationale: the board's principle names two outcomes (halt with way forward, or warn); the grouped rows keep the table readable and keep tests on refusals a run can reach.

### D3 — Refusal table lives in a new contract doc

Decision: create `contracts/specs/refusal-spec.md` with one section per module (webster, shed, loom).
Columns: Refusal, Trigger, Class, Way forward.
The `websterengine` package doc, `shedengine` package doc, the loom package doc that owns its gates, and `contracts/specs/webster-spec.md` each link to it rather than restate rows.
The ly-drive skill references it as the lookup for an escalation's way forward.
The file is plain markdown and is not registered in `contracts/specs/specs.go`.

Rationale: the table spans three modules and is consumed by the ly-drive driver and the operator, which makes it a cross-module contract, the class `contracts/specs/` holds.

### D4 — Fork-audit class split

Decision:

- Correctness class: `fork-contract-write`, and `parent-write` of a path in the warp's tracked tree or anywhere under the run's `_lyx` content (reports dir, `state.json`, plan dir).
- Policy class: `fabric-reference`, `named-spawn`, `nested-agent`, and every other `parent-write` (a path outside the worktree, or a git-ignored warp path outside `_lyx`).

The correctness rule is checked first and wins, since `_lyx` is itself excluded from warp git.

Rationale: a nested Agent call cannot run inside a fork, so it changes nothing.
A Master scratch write outside the tracked tree and outside `_lyx` lands nothing.
A contract-file write by a fork forges the run's terminal judgment;
a Master write into the warp tree bypasses the audited delta, and one under `_lyx` forges a report or corrupts run state.

### D5 — Policy-class findings become recorded warnings

Decision: when record-batch finds only policy-class violations on a batch whose report is `OK`, it re-runs each of the batch's cards' `verify:` commands in-process on the worktree as it stands.
All pass (a card with no `verify:` passes) → the batch is recorded done, and each finding becomes a warning.
Any fails → the findings are treated as correctness class (D6).
A verify command that fails to start, exits non-zero, or times out fails.
A failure before any verify command starts (reading the plan or the cards) is D2's grouped transient I/O row: it halts and mutates nothing.

A recorded warning, whether record-batch or run exit (D8) raises it, is:

- appended in `state.json` once per finding identity (a retry never duplicates it): to the batch's state for a record-batch warning, to a run-level warning list for a run-exit one;
- returned on the raising verb's envelope under the existing `warnings` key (record-batch's, or `lyx webster run`'s through the run result's warnings);
- appended by Go at run exit to `summary.md` as an `## Audit warnings` section, after the pattern of the integration-triage section.

Rationale: board requirement; the verify re-run replaces trust in the fork's own account with evidence, and runs only when a policy finding exists.

### D6 — A report rejected on its merits fails the batch instead of wedging it

Decision: when record-batch rejects an `OK` report on its merits — a correctness-class audit finding, a failing card done-check, or a blocking handle-bind finding about the batch's own cards — it:

1. marks the offending fork transcripts and parent findings as seen;
2. archives the report with webster's existing archive-never-refuse primitive;
3. records the batch terminal-failed with the reasons as its digest, naming each correctness-class `parent-write` path;
4. returns an error naming `lyx webster recover-batch NN` as the way forward.

`recover-batch NN` then proceeds on that batch: no OK report remains to refuse over.
The recovery strand reads the digest, so it treats each named `parent-write` path as suspect and reverts or re-derives it.

Rationale: today each of these leaves an OK report on disk and the batch non-terminal, so begin-batch (report exists), record-batch (same finding again) and recover-batch (OK report) refuse each other.
The recovery strand continues from the committed tree, as it already does for a stuck fork.

### D7 — An unattributable report is archived and the batch re-driven

Decision: when record-batch cannot attribute a report — no begin-batch record (`ErrNoBeginRecord`), or zero fork transcripts after the settle window (`ErrNoForkTranscripts`) — it archives the report, leaves the batch re-drivable, and returns an error naming `lyx webster begin-batch NN`.
The manual "move the report out of the reports dir" recourse is removed from `ErrNoForkTranscripts`'s text.

Rationale: both cases form the same three-verb ring as D6; an unattributable report is discarded rather than trusted, whether Master forged it or the transcript lives on another machine.
The existing settle window already guards the transcript flush race.

### D8 — Every audit finding is dispositioned once

Decision: a parent-transcript finding (`named-spawn`, `parent-write`, parent `fabric-reference`) is consumed by the first record-batch that reports it, and never refuses or warns again.
Fork transcripts are marked seen whether they passed, warned, or failed.
The dispositions are recorded in `state.json` per finding identity.

The run-exit whole-session audit cross-check applies the same classification and passes every finding already dispositioned there (a recorded warning, or a finding consumed by a D6-failed batch).
A finding no record-batch dispositioned — from the integration fork's transcript, or from Master's session after the last record-batch — is dispositioned at run exit:

- policy class: a recorded warning (D5), and the run keeps its outcome;
- correctness class: Master's `done` is demoted to `stuck`, as the integration stage demotes a regression, with a stuck reason naming each finding and each suspect path;
  way forward: revert or re-derive the named paths on the warp with `git`, then re-step the Webster row.

Its audited-transcript count check is unchanged.

Rationale: the parent audit covers Master's whole session, so today one parent finding refuses every later batch of the run.
Record-batch and run exit are the only two audit sites, and recover-batch runs no audit;
a run-exit check blind to dispositions would fail the run on every finding record-batch already settled.
The integration fork runs after every batch is recorded, so its findings reach only run exit.
A bare re-step would spawn a fresh Master whose session carries no finding, so the suspect paths are named for revert before it.

### D9 — A finding about a later card never fails the current batch

Decision: a drift or plan-references-deleted-symbol finding that concerns a card other than the ones being recorded becomes a warning on the current batch (D5's recording path).
The later card's own begin-batch dispatch validation refuses it, with the way forward "edit the plan, then `lyx webster rebaseline`".

Rationale: the refusal lands on what caused it; recovering the current batch cannot fix a stale later card.

### D10 — `lyx webster rebaseline`

Decision: a new webster verb that accepts the on-disk plan as the run's plan: it restamps the plan fingerprint in `state.json` and keeps every batch record.
It serializes with the existing state-mutation lock.
It refuses when, under the edited plan, any begun or terminal batch number covers a different card set than the one its record was begun with (a removed, renumbered or regrouped card);
way forward: restore those cards, or reset the branch to the run's start and use `lyx webster run --fresh`.
`ErrFingerprintMismatch` messages at run entry and begin-batch name `lyx webster rebaseline` first, and `--fresh` only with the branch-reset precondition.

Rationale: a foreign plan edit mid-run today leaves only `--fresh`, whose fresh state re-validates landed create cards as create-already-exists and refuses.
A standalone verb composes with loom: re-stepping the Webster row after it resumes normally.

### D11 — `lyx shed goto`

Decision: a new generic shed verb, `lyx shed goto [<run-id>] --to <producer>`, in `internal/shedverbs`, so the shed, loom and batten subtrees all gain it.
It sets the run's current producer to the named row with state `paused`, so the next `step` or `run` resumes there through the existing resume write.
It appends one history entry with the new history-only outcome `goto`, naming the target producer.
The bounce-budget count treats a `goto` entry as an episode boundary for every producer sharing the target's `Segment` (the target alone when `Segment` is empty), which gives that segment a fresh budget.
Loom's seed-coherence outcome check accepts `goto`.
goto places no restriction on targets.
An entry gate that refuses a re-entered run is an ordinary refusal-table row with its own way forward;
for `Loom-Preflight`'s fresh-start (half-finished) check, that is `goto` the row after it, or seed a new run.
It refuses:

- while the run lock is held (way forward: `lyx shed pause`, or wait and check `status`);
- a producer name not in the recipe (the message lists the valid names);
- a `done` run (way forward: seed a new run).

It is an operator and orchestrator verb; ly-drive never calls it on its own and names it in an escalation.
It adds no step refusal kind.

Rationale: `current_producer names no producer in the list` is reachable whenever a deploy renames or removes a recipe row mid-run, and an exhausted bounce budget re-blocks on every re-step; neither has a way forward today short of hand-editing `status.json`.
The budget is per-producer and counted from history back to the producer's own last `done`, so only a history entry can reset it without a second source of truth.
A `running` write with the lock free reads as a crashed driver to loom's entry observation;
`paused` does not.

### D12 — New invariant: Refusal Way-Forward Invariant

Decision: add to CONSTRAINTS.md:

- it binds the modules that have a section in `contracts/specs/refusal-spec.md` (webster, shed and loom after this task);
  another module joins when its own audit adds its section;
- every refusal reachable through a bound module's verbs names its way forward (D1) in its message and has a row in its section;
- a guard in a bound module that does not protect correctness warns and records rather than halts;
- a new refusal in a bound module lands with its row and its reaching test in the same commit.

Enforcement is review discipline plus the per-row tests; no scan.
The Shed Verb-Set Invariant's generic verb list gains `goto`.

Rationale: the principle is cross-cutting and outlives this audit; CLAUDE.md requires the invariant in the same commit.

## Constraints

- Shed Verb-Set Invariant: `goto`'s body lives in `internal/shedverbs`; it derives no path and imports no resolver.
- The closed `step` refusal-kind vocabulary and the step envelope key set are unchanged.
- Transient Stop Invariant: no new transient-mark site; `goto` and the new dispositions set no mark.
- The `goto` history outcome changes the status-file history contract: `internal/shedengine` declares it, no producer returns it, and every reader that validates outcomes (loom's seed-coherence check) accepts it.
- Shed Producer-Seam Invariant: `internal/shedengine` still imports only stdlib, `state`, `lock`.
- Told-Geometry and Cwd Resolution: `rebaseline` and the archive paths use webster's told geometry accessors.
- websterengine stays fabric-blind; any fabric commit of `state.json` or an archived report happens in `internal/webstercli` at the verb boundary, as today.
- hubforge Fabric-Fixture Invariant: every hub fixture in the new tests is built by `internal/hubforge`.
- CLI/Cobra Invariant: both new verbs carry a non-empty `Short`, JSON errors via `internal/output`, `clihelp.ShouldAbort` first, and appear in the help-tree tests.
- Existing behaviour when nothing is refused is unchanged.
- The fork audit stays a steering guard, not a security boundary.

## Auto-mode assumptions

- The board's "table in the webster module doc" is met by a new contract doc (D3).
- `nested-agent` is policy class, though the board named only `fabric-reference` and `named-spawn` (D4).
- `parent-write` is split by path rather than kept whole in one class (D4).
- The board's "commits pass verify" is read as a re-run of the cards' own `verify:` commands (D5).
- The module scope is chosen by verb surface (Scope).
- The new verbs' names (D10, D11).
- `goto` does not re-open a done run (D11).

## Open risks

- The Plan's enumeration is the audit: a refusal it misses keeps its current behaviour.
  The per-row test requirement makes a missed row visible only when the refusal is hit.
- D6's recovery continues on the fork's committed tree; a fork that committed bad work before a correctness violation leaves the recovery strand to repair it, not revert it.
- D7 re-drives a batch whose cards may already be committed; the new fork finds the work done, which relies on begin-batch's begun-card validation scoping staying correct.
- A consumed correctness-class `parent-write` (D8) is not re-audited: if the recovery strand keeps a suspect path the D6 digest names, that content can land.
- Landing and batten refusals may still wedge runs until their own audit.

## Acceptance criteria

- `contracts/specs/refusal-spec.md` exists with webster, shed and loom sections; every refusal in scope is a row or belongs to a grouped wiring or I/O row.
- Every non-grouped row's message names its way forward per D1.
- Each non-grouped row has a test that reaches it from a realistic on-disk state (real git fixture, state files as the verbs write them):
  - a halting row asserts the message names the way forward, takes it, and asserts the refused verb, or the verb the way forward names, then succeeds;
  - a policy row asserts the warning is recorded and the verb succeeds.
- Must-cover scenarios:
  - a `fabric-reference` finding on an OK report whose cards' verify passes → batch recorded done, one warning in `state.json`, on the envelope, and in `summary.md`'s `## Audit warnings`; a second record-batch call adds no duplicate;
  - the same finding with a failing card verify → batch recorded failed, report archived, `recover-batch` proceeds;
  - a `named-spawn` parent finding at batch 2 → batch 2 records with a warning, batch 3 records with no refusal and no repeated warning, and the run-exit audit cross-check passes;
  - a `fabric-reference` in the integration fork's transcript → the run ends `done`, with one run-level warning in `state.json`, on `lyx webster run`'s envelope, and in `summary.md`'s `## Audit warnings`;
  - a Master `parent-write` into the warp's tracked tree after the last batch → the run ends `stuck` with the path named in the stuck reason; reverting it with `git` and re-stepping the Webster row ends `done`;
  - a Master `parent-write` to `_lyx/webster/state.json` → correctness class, batch failed, the path named in the digest;
  - `fork-contract-write` → batch failed, report archived, `recover-batch` proceeds;
  - OK report with a failing done-check → batch failed, `recover-batch` proceeds (the ring no longer forms);
  - report present with no begin record → report archived, `begin-batch` succeeds;
  - report present with zero fork transcripts after the settle window → report archived, `begin-batch` succeeds;
  - foreign plan edit mid-run → begin-batch refuses naming `rebaseline`; `lyx webster rebaseline` then lets begin-batch proceed with prior batch records intact;
  - `rebaseline` over a plan that removes a recorded card, or regroups a begun batch's cards → refused with the restore/reset way forward;
  - a shed run whose `current_producer` is absent from the recipe → step refuses; `lyx shed goto --to <row>` leaves the run `paused` at that row, and step then proceeds;
  - a segment with an exhausted bounce budget → `goto` into it gives every producer in the segment a fresh budget;
  - a loom run after `goto` → the next `lyx loom run` files no crash-resume anomaly;
  - `goto` to `Loom-Preflight` on a run past it → the half-finished check refuses naming `goto` to the next row;
  - `goto` while the run lock is held, to an unknown producer, and on a done run → each refused with its way forward;
  - regression tests for the three 2026-09-30 incidents (fork-audit false positive, begun-but-unrecorded batch at run entry and begin-batch, the parent-merge ring) exist and pass.
- CONSTRAINTS.md carries the Refusal Way-Forward Invariant and the Shed Verb-Set list includes `goto`.
- The websterengine package doc describes D5–D9 and links the table; ly-drive's skill references the table and `goto`.
- `go test ./...` passes.

## Notes for the plan writer

Candidate wedges seen during discussion, unverified and not exhaustive:

- record-batch returns on audit violations before appending to `SeenForkTranscripts`, and the parent audit is whole-session, so both repeat on every retry.
- record-batch's `ErrCardNotDone` returns (done-checks, bind-blocking, drift-blocking) appear to leave the OK report and a non-terminal batch in place.
- `ErrNoForkTranscripts` and `ErrNoBeginRecord` both leave a report begin-batch refuses to overwrite.
- run entry's `--fresh` path builds empty state, so dispatch validation over landed cards refuses.
- shedengine's `current_producer ... names no producer in the list` refusal, and bounce-budget exhaustion re-blocking on re-step.
- loom's fresh-start coherence check runs only in the `Loom-Preflight` row; it is the one known entry gate a `goto` re-entry reaches.
- webster already has archive primitives and an integration-triage summary appender to model the new ones on.
