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
- Any change to the closed `step` refusal-kind vocabulary or the step envelope key set.

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

- Policy class: `fabric-reference`, `named-spawn`, `nested-agent`, and `parent-write` of a path outside the worktree or under a git-ignored path.
- Correctness class: `fork-contract-write`, and `parent-write` of a tracked-tree path or a path in the reports directory.

Rationale: a nested Agent call cannot run inside a fork, so it changes nothing.
A Master scratch write outside the tracked tree lands nothing.
A contract-file write by a fork forges the run's terminal judgment, and a Master write into the tree or the reports dir bypasses the audited delta or forges a report.

### D5 — Policy-class findings become recorded warnings

Decision: when record-batch or recover-batch finds only policy-class violations on a batch whose report is `OK`, it re-runs each of the batch's cards' `verify:` commands in-process on the worktree as it stands.
All pass (a card with no `verify:` passes) → the batch is recorded done, and each finding becomes a warning.
Any fails, or a verify cannot run → the findings are treated as correctness class (D6).

A recorded warning is:

- appended to the batch's state in `state.json`, once per finding identity (a retry never duplicates it);
- returned on the verb's envelope under the existing `warnings` key;
- appended by Go at run exit to `summary.md` as an `## Audit warnings` section, after the pattern of the integration-triage section.

Rationale: board requirement; the verify re-run replaces trust in the fork's own account with evidence, and runs only when a policy finding exists.

### D6 — A report rejected on its merits fails the batch instead of wedging it

Decision: when record-batch rejects an `OK` report on its merits — a correctness-class audit finding, a failing card done-check, or a blocking handle-bind finding about the batch's own cards — it:

1. marks the offending fork transcripts and parent findings as seen;
2. archives the report with webster's existing archive-never-refuse primitive;
3. records the batch terminal-failed with the reasons as its digest;
4. returns an error naming `lyx webster recover-batch NN` as the way forward.

`recover-batch NN` then proceeds on that batch: no OK report remains to refuse over.

Rationale: today each of these leaves an OK report on disk and the batch non-terminal, so begin-batch (report exists), record-batch (same finding again) and recover-batch (OK report) refuse each other.
The recovery strand continues from the committed tree, as it already does for a stuck fork.

### D7 — An unattributable report is archived and the batch re-driven

Decision: when record-batch cannot attribute a report — no begin-batch record (`ErrNoBeginRecord`), or zero fork transcripts after the settle window (`ErrNoForkTranscripts`) — it archives the report, leaves the batch re-drivable, and returns an error naming `lyx webster begin-batch NN`.
The manual "move the report out of the reports dir" recourse is removed from `ErrNoForkTranscripts`'s text.

Rationale: both cases form the same three-verb ring as D6; an unattributable report is discarded rather than trusted, whether Master forged it or the transcript lives on another machine.
The existing settle window already guards the transcript flush race.

### D8 — Every audit finding is dispositioned once

Decision: a parent-transcript finding (`named-spawn`, `parent-write`, parent `fabric-reference`) is consumed by the first record-batch or recover-batch that reports it, and never refuses or warns again.
Fork transcripts are marked seen whether they passed, warned, or failed.

Rationale: the parent audit covers Master's whole session, so today one parent finding refuses every later batch of the run.

### D9 — A finding about a later card never fails the current batch

Decision: a drift or plan-references-deleted-symbol finding that concerns a card other than the ones being recorded becomes a warning on the current batch (D5's recording path).
The later card's own begin-batch dispatch validation refuses it, with the way forward "edit the plan, then `lyx webster rebaseline`".

Rationale: the refusal lands on what caused it; recovering the current batch cannot fix a stale later card.

### D10 — `lyx webster rebaseline`

Decision: a new webster verb that accepts the on-disk plan as the run's plan: it restamps the plan fingerprint in `state.json` and keeps every batch record.
It serializes with the existing state-mutation lock.
It refuses when the edited plan removes or renumbers a card whose batch is begun or terminal; way forward: restore that card, or reset the branch to the run's start and use `lyx webster run --fresh`.
`ErrFingerprintMismatch` messages at run entry and begin-batch name `lyx webster rebaseline` first, and `--fresh` only with the branch-reset precondition.

Rationale: a foreign plan edit mid-run today leaves only `--fresh`, whose fresh state re-validates landed create cards as create-already-exists and refuses.
A standalone verb composes with loom: re-stepping the Webster row after it resumes normally.

### D11 — `lyx shed goto`

Decision: a new generic shed verb, `lyx shed goto [<run-id>] --to <producer>`, in `internal/shedverbs`, so the shed, loom and batten subtrees all gain it.
It sets the run's current producer to the named row with state `running` and a fresh bounce budget for that row's segment, and appends a history entry recording the jump.
It refuses:

- while the run lock is held (way forward: `lyx shed pause`, or wait and check `status`);
- a producer name not in the recipe (the message lists the valid names);
- a `done` run (way forward: seed a new run).

It is an operator and orchestrator verb; ly-drive never calls it on its own and names it in an escalation.
It adds no step refusal kind.

Rationale: `current_producer names no producer in the list` is reachable whenever a deploy renames or removes a recipe row mid-run, and an exhausted bounce budget re-blocks on every re-step; neither has a way forward today short of hand-editing `status.json`.

### D12 — New invariant: Refusal Way-Forward Invariant

Decision: add to CONSTRAINTS.md:

- every refusal a run can reach names its way forward (D1) in its message and has a row in `contracts/specs/refusal-spec.md`;
- a guard that does not protect correctness warns and records rather than halts;
- a new refusal lands with its row and its reaching test in the same commit.

Enforcement is review discipline plus the per-row tests; no scan.
The Shed Verb-Set Invariant's generic verb list gains `goto`.

Rationale: the principle is cross-cutting and outlives this audit; CLAUDE.md requires the invariant in the same commit.

## Constraints

- Shed Verb-Set Invariant: `goto`'s body lives in `internal/shedverbs`; it derives no path and imports no resolver.
- The `step` refusal-kind vocabulary stays at five values; the step envelope key set is unchanged.
- Transient Stop Invariant: no new transient-mark site; `goto` and the new dispositions set no mark.
- Shed Producer-Seam Invariant: `internal/shedengine` still imports only stdlib, `state`, `lock`.
- Told-Geometry and Cwd Resolution: `rebaseline` and the archive paths use webster's told geometry accessors.
- websterengine stays fabric-blind; any fabric commit of `state.json` or an archived report happens in `internal/webstercli` at the verb boundary, as today.
- hubforge Fabric-Fixture Invariant: every hub fixture in the new tests is built by `internal/hubforge`.
- CLI/Cobra Invariant: both new verbs carry a non-empty `Short`, JSON errors via `internal/output`, `clihelp.ShouldAbort` first, and appear in the help-tree tests.
- Existing behaviour when nothing is refused is unchanged.
- The fork audit stays a steering guard, not a security boundary.

## Auto-mode assumptions

- The board's "table in the webster module doc" is met by a new `contracts/specs/refusal-spec.md` linked from the webster docs (D3).
- `nested-agent` is policy class, though the board named only `fabric-reference` and `named-spawn` (D4).
- `parent-write` splits by path: tracked tree or reports dir is correctness, anything else is policy (D4).
- "Commits pass verify" means the batch's cards' own `verify:` commands re-run in-process on the current worktree (D5).
- Landing, batten, orch, fabric, burler and board refusals are outside this task.
- The new verbs are named `goto` and `rebaseline`.
- A done run is not re-openable through `goto`.

## Open risks

- The Plan's enumeration is the audit: a refusal it misses keeps its current behaviour.
  The per-row test requirement makes a missed row visible only when the refusal is hit.
- D6's recovery continues on the fork's committed tree; a fork that committed bad work before a correctness violation leaves the recovery strand to repair it, not revert it.
- D7 re-drives a batch whose cards may already be committed; the new fork finds the work done, which relies on begin-batch's begun-card validation scoping staying correct.
- `goto` can re-enter a row whose entry gate expects a fresh seed (loom's coherence check); the audit must decide each such gate's behaviour on re-entry.
- Landing and batten refusals may still wedge runs until their own audit.

## Acceptance criteria

- `contracts/specs/refusal-spec.md` exists with webster, shed and loom sections; every refusal in scope is a row or belongs to a grouped wiring or I/O row.
- Every non-grouped row's message names its way forward per D1.
- Each non-grouped row has a test that reaches the refusal from a realistic on-disk state (real git fixture, state files as the verbs write them), asserts the message names the way forward, and then takes that way forward and asserts the run progresses.
- Must-cover scenarios:
  - a `fabric-reference` finding on an OK report whose cards' verify passes → batch recorded done, one warning in `state.json`, on the envelope, and in `summary.md`'s `## Audit warnings`; a second record-batch call adds no duplicate;
  - the same finding with a failing card verify → batch recorded failed, report archived, `recover-batch` proceeds;
  - a `named-spawn` parent finding at batch 2 → batch 2 records with a warning, batch 3 records with no refusal and no repeated warning;
  - `fork-contract-write` → batch failed, report archived, `recover-batch` proceeds;
  - OK report with a failing done-check → batch failed, `recover-batch` proceeds (the ring no longer forms);
  - report present with no begin record → report archived, `begin-batch` succeeds;
  - report present with zero fork transcripts after the settle window → report archived, `begin-batch` succeeds;
  - foreign plan edit mid-run → begin-batch refuses naming `rebaseline`; `lyx webster rebaseline` then lets begin-batch proceed with prior batch records intact;
  - `rebaseline` over a plan that removes a recorded card → refused with the restore/reset way forward;
  - a shed run whose `current_producer` is absent from the recipe → step refuses; `lyx shed goto --to <row>` then lets step proceed;
  - a segment with an exhausted bounce budget → `goto` into it gives a fresh budget;
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
- loom's fresh-start coherence check is the gate most likely to misfire on a `goto` re-entry.
- webster already has archive primitives and an integration-triage summary appender to model the new ones on.
