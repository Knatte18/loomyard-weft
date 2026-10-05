# Decision record — selfreport-issues-say-nothing

## Goal

lyx's Go code stops filing GitHub issues.
Self-reported issues are written only by the Tier 2 reflection agent (`Friction-Reflect` and the halt reflection), which can say what happened, why it is abnormal and what to do.
The Go-side anomaly detection stays, but its output becomes friction notes the reflection reads, carrying the full facts the empty Tier 1 issues (#366–#382) lacked.
A deliberate resume through `lyx loom start`, which covers a migration between machines, is no anomaly.

## Scope

In:

- Remove Tier 1 GitHub filing from `internal/loomcli`: the filing pass, its filed-title marker with its pending retry list, the filing seam on the CLI struct, and the `selfreport` key in `loom.yaml`'s template and config struct.
- Remove the issue-title and issue-body rendering from `internal/loomengine` (`anomaly.go`'s titles, `anomalybody.go`), and the marker path accessors.
- Keep the pure anomaly classification (crash-resume and the three halt kinds) and route each detected anomaly into a Go-authored friction note.
- Make `lyx loom start` vouch for the next entry observation, so the resume it performs never reads as a crash.
- Update the reflection stencil, `PATTERN.md` and `pattern/PATTERN-friction-capture.md`, `internal/friction/doc.go`, the `halt.go` file comment, and `docs/overview.md`'s selfreport line.

Out:

- `lyx selfreport create` and its default `bug` label for manual use; the board-intake calls in `selfreportengine/inbox.go`.
- Webster refusal notes and the agent-written friction notes.
- `frictionengine`'s scan, archive and re-entrancy mechanics.
- Editing any live `_lyx/config/loom.yaml`, including main-weft's `selfreport: false`; `lyx config reconcile` drops the stale key when the operator runs it.
- Closing or editing issues #366–#382.

## Decisions

### D1 — No Go code path files an issue

Decision: delete `detectAndFileAnomalies`, `runFilingPass`, `collapseAnomaliesByTitle`, the `selfreportFiledMarker`/`pendingAnomaly` types and their read/write helpers, the `selfreportDeps.FileIssue` seam and `loomCLI.fileIssue`, `loomengine.LoomSelfreportFiled`/`LoomSelfreportFiledLock` and their file-name constants, the anomaly `Title` field with its renderers, and `RenderAnomalyBody`.
`loomcli` no longer imports `selfreportengine`.
The only production caller of `selfreportengine.CreateIssue` is `internal/selfreportcli`, so an issue is filed only when an agent (or the operator) runs `lyx selfreport create`.

Rationale: the Go filer can only print fields it holds, so its issues carry keys and blanks; the reflection agent reads the notes and the trace and can explain the event.

### D2 — The `selfreport` config key is removed

Decision: drop `selfreport` from `internal/loomengine/template.yaml` and `Selfreport` from the config struct.
Every gate that read `cfg.Selfreport` now reads whether Tier 2 is on: a non-empty friction directory, the existing single "off" signal of `internal/friction`.
A `loom.yaml` that still carries `selfreport` must keep loading; if the loader rejects unknown keys, the plan handles the retired key so that existing configs load unchanged.

Rationale: with no filing there is nothing for the knob to gate; anomaly notes are Tier 2 input and follow Tier 2's own on/off.

### D3 — Anomalies reach the reflection as friction notes

Decision: the existing input channel of `Friction-Reflect` and the halt reflection, the friction directory, carries the anomalies.
The pure classifier stays in `internal/loomengine` with its four kinds: `crash-resume`, `escalation-to-human`, `bounce-budget-exhausted` and `producer-hard-failure`.

- Halt kinds: the existing `loom-halt` note gains the anomaly kind line and the run's history rows (producer, outcome, at), alongside the producer, state, reason, history length and trace file it already carries.
  No separate note per halt kind; one halt writes one note, as today.
  The `TierOneFiled` field and its "Tier 1 anomaly filing files the halt event itself" line are removed.
- Crash-resume: a new Go-authored note, id `loom-crash-resume`, written through `friction.NotePath` at the entry observation itself, under `run` and `step` alike.
  It carries the slug, the parent, the observed state and current producer, the history length, the last history rows, the verb that resumed (`run` or `step`) and the trace file.
- Writing at entry replaces the deferred post-run filing, so the cancelled-context exception and the busy-skip ordering that existed to keep the observation alive until filing go away.
  A busy probe (run lock held) is still no anomaly, as today.
- A crash-resume note is not reflected on immediately; it waits in the friction directory for the next reflection (the `Friction-Reflect` row on done, or the next halt's reflection), as every other note does.
- Notes are written only when Tier 2 is on; a note-write failure is warned and never changes the verb's exit or envelope (`PATTERN-friction-capture`).

Rationale: the friction directory is already the reflection's one input and is already archived only after a reflection covers it, so no new channel or retry mechanism is needed.

### D4 — A resume through `lyx loom start` is no anomaly

Decision: the step clean-handoff marker becomes a general handoff voucher with two writers: a completed `lyx loom step` (as today) and `lyx loom start` right before it spawns a driver.
Start records the persisted history length and state it saw; the driver's first entry observation consumes the voucher one-shot and, on a match, reports no crash-resume.
Every resume path (operator, orch, batten's `lyx loom start --no-attach`, a migration between machines) goes through start, so a crash-resume note fires only when a bare `lyx loom run` or `lyx loom step` finds a dead driver's run with no voucher, such as a step killed mid-producer that its own driver re-issues.

Bound: the voucher suppresses at most one entry observation, and only one whose history length and state equal what start or the completed step recorded; it is machine-local under `.lyx`, so it never crosses machines or survives a fabric re-wire.
A driver that start spawned and that then dies mid-run is not noted on the next start; its evidence is the driver log and trace, which the halt or operator path already surfaces.

Rationale: on-disk state cannot tell a migration from a crash; the verb that resumes can, and every deliberate resume uses start.

### D5 — The reflection decides whether an anomaly is worth an issue

Decision: `contracts/stencils/friction/friction-template-reflection.md` lists `loom-crash-resume` among the Go-written note kinds and drops the Tier 1 sentence.
For a `loom-halt` or `loom-crash-resume` note, the agent files an issue only when the notes, the reason or the trace show a lyx problem behind the event; a halt caused by the task's own code, or a review segment that legitimately did not converge, is dropped.
Every issue body states what happened, why it is abnormal and what to do, then the existing provenance line.
The agent picks the label through `lyx selfreport create`'s own `--label` guidance; the stencil says that a process signal (a spent budget, an escalation) is not a defect, and points at the verb's contract rather than restating it (`PATTERN-producer-pointer-rule`).

Rationale: the board entry assigns the judgement to the reflection agent, which is the only reader with the context to make it.

### D6 — The invariant is pinned in PATTERN

Decision: add a `PATTERN.md` entry, `PATTERN-agent-filed-issues`: filing a GitHub issue from lyx goes only through `lyx selfreport create`, run by an agent or the operator; no other production package calls `selfreportengine.CreateIssue`. (test)
Rewrite `pattern/PATTERN-friction-capture.md`'s selfreport bullet: Go-detected anomalies become friction notes, and a halt note carries its anomaly kind.

Rationale: the bug was Go code acquiring a filing path; a test-enforced entry keeps it from coming back.

## Constraints

- `PATTERN-friction-capture`: note writes go through `friction.NotePath`, never change the verb's outcome, and no note is archived before a reflection covers it.
- `PATTERN-durable-vs-ephemeral-state`, `PATTERN-lyxdirs-single-declarer`: the voucher stays under `.lyx` through `loomengine`'s accessor.
- `PATTERN-config-strictness`: loom keeps its current loader; the retired key must not break it.
- `PATTERN-producer-pointer-rule`: the stencil points at `lyx selfreport create`'s contract.
- `PATTERN-parent-directive`: the reflection stencil keeps its parent directive marker.
- `PATTERN-sole-parsers`, `PATTERN-shed-producer-seam`: untouched; no new parser or shed import.
- `PATTERN-test-economy`, `PATTERN-test-tier-purity`, `PATTERN-test-parallel`: the classifier and note rendering stay untagged table tests; tests for removed code are deleted, not adapted.
- `PATTERN-documentation-lifecycle`, CLAUDE.md "Docs land in the same commit": doc.go, `docs/overview.md` and PATTERN change with the code.

## Auto-mode assumptions

- Every resume of a loom run goes through `lyx loom start`; batten calls `lyx loom start --no-attach` and orch's documented loop does the same.
- `issues #366–#382` are left for the hub's intake to close.
- The stale machine-local `selfreport-filed.json` marker files are left on disk; nothing reads them after this change.
- The `bug` default label of `lyx selfreport create` stays for manual filing.

## Open risks

- With the voucher, a driver that start spawned and that dies mid-run is never noted as a crash-resume; only the trace and the driver log record it.
- If the config loader turns out strict on unknown keys, main-weft's `loom.yaml` breaks on deploy unless the plan keeps the key loadable.
- The reflection agent's judgement replaces a deterministic filer, so a real lyx fault behind a halt may go unfiled if the agent misreads it.

## Acceptance criteria

- No `lyx loom run` or `lyx loom step` path calls the GitHub API: `loomcli` has no filing seam, and a test fails if a production package other than `internal/selfreportcli` calls `selfreportengine.CreateIssue`.
- `loom.yaml`'s template has no `selfreport` key, and a `loom.yaml` that still carries `selfreport: false` loads without error.
- A `blocked` halt with the bounce-budget prefix, a `blocked` halt with any other reason, and a `failed` halt each write one `loom-halt` note naming its anomaly kind, producer, state, reason, history rows and trace file.
- A `run` or `step` entry that observes state `running`, a free run lock, a non-empty history and no matching voucher writes one `loom-crash-resume` note carrying slug, parent, state, current producer, history length, last history rows, resuming verb and trace file.
- No crash-resume note when: `lyx loom start` vouched for the observed history length and state; a completed `lyx loom step` vouched; the run lock is held; the state is not `running`; the history is empty; Tier 2 is off.
- A voucher is consumed by the first entry observation, so a second observation of the same state without a new voucher writes a note.
- A note-write failure leaves the verb's exit and envelope unchanged.
- The reflection stencil lists `loom-crash-resume`, has no Tier 1 sentence, and states the file-only-a-lyx-problem rule and the what-happened/why-abnormal/what-to-do body.
- `PATTERN.md` carries `PATTERN-agent-filed-issues`; `pattern/PATTERN-friction-capture.md`, `internal/friction/doc.go`, `internal/loomcli/halt.go`'s file comment and `docs/overview.md` no longer describe Go-side filing or the `selfreport` knob.

## Notes for the plan writer

- Filing code: `internal/loomcli/selfreport.go` (also holds `observeEntry` and the step-handoff marker helpers, which stay), `internal/loomcli/arm.go` (`anomalyDeps`, `loomPreRun`, `observeStepEntry`, `loomPostRun`), `internal/loomcli/halt.go`, `internal/loomcli/cli.go` (`fileIssue`).
- Detector: `internal/loomengine/anomaly.go`, `anomalybody.go`, `config.go` (marker accessors, `Selfreport` field, `LoomStepHandoff`), `template.yaml`.
- Tests to delete or rework: `selfreport_test.go`, `selfreport_github_test.go`, `anomaly_test.go`, `anomalybody_test.go`, the `Selfreport` cases in `config_test.go`, `halt_test.go` and `step_test.go`.
- `lyx loom start`'s driver spawn lives in `internal/loomcli/start.go`; the voucher write goes right before the spawn, after the status read start already performs.
- Unknown-key handling: `internal/configengine/config.go` `load` and `internal/yamlengine`; `lyx config reconcile` preserves or drops orphan keys per `internal/configcli`.
- The `recurring-finding` kind was already removed in `a0dae77c3`.
