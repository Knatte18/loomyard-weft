# loom: stop filing issues from Go; anomalies become friction notes

lyx's Go code no longer files GitHub issues.
The Tier 1 filer produced issues that said nothing: a ledger key, blank state and producer fields, an empty history, and the `bug` label on process signals.
It also reported a deliberate migration of runs between machines as a crash-resume.
Issues are now written only by the Tier 2 reflection agent, which can say what happened, why it is abnormal and what to do.

## What changes

- `internal/loomcli`: the filing pass, the filed-title marker with its retry list and the filing seam on the CLI struct are deleted.
  `loomcli` no longer imports `selfreportengine`.
- `internal/loomengine`: the issue-title and issue-body rendering and the marker path accessors are deleted.
  The pure anomaly classification stays, with its four kinds: `crash-resume`, `escalation-to-human`, `bounce-budget-exhausted` and `producer-hard-failure`.
- Each detected anomaly becomes a Go-authored friction note, written through `friction.NotePath`.
  - A halt still writes its one `loom-halt` note, which now also carries the anomaly kind and the recent history rows.
  - A crash-resume writes a new `loom-crash-resume` note at the `run` or `step` entry observation, with the slug, parent, state, current producer, history length, recent history rows, resuming verb and trace file.
    Writing at entry replaces the deferred post-run filing, so the cancelled-context exception and the busy-skip ordering are gone.
  - Notes wait in the friction directory for the next `Friction-Reflect` or halt reflection, like every other note.
- `lyx loom start` is no anomaly: the step clean-handoff marker is now a general handoff voucher (`handoff-voucher.json`, machine-local under `.lyx`).
  A completed `lyx loom step` and `lyx loom start` (right before it spawns a driver) both write it.
  The driver's first entry observation consumes it one-shot and, when history length and state match, reports no crash-resume.
- The reflection stencil lists `loom-crash-resume`, drops the Tier 1 sentence, and tells the agent to file only when the notes, reason or trace show a lyx problem.
  The issue body states what happened, why it is abnormal and what to do, and the label follows `lyx selfreport create`'s own guidance.
- `PATTERN.md` gains `PATTERN-agent-filed-issues`, enforced by a scan test in `cmd/lyx/agentfiledissues_test.go`: no production package other than `internal/selfreportcli` calls `selfreportengine.CreateIssue`.
  `pattern/PATTERN-friction-capture.md`, `internal/friction/doc.go`, `internal/loomcli/halt.go`'s file comment and `docs/overview.md` follow.

## Design decisions to check

- The `selfreport` key is removed from `template.yaml` and the config struct.
  Anomaly notes follow Tier 2's own on/off, a non-empty friction directory.
  The loom loader is non-strict, so a `loom.yaml` still carrying `selfreport` loads unchanged; a test pins this.
  No live `_lyx/config/loom.yaml` is edited, and `lyx config reconcile` drops the stale key when the operator runs it.
- The voucher suppresses at most one entry observation, and only one whose history length and state match what was recorded.
  `start` writes it only after its refusals pass, only with Tier 2 on, and each write replaces an earlier one.
  A voucher whose spawn then failed readiness stays on disk and suppresses at most one later matching observation.
  The parked-driver resume spawns nothing and writes none, since a non-`running` state is already no crash-resume.
- A driver that `start` spawned and that then dies mid-run is not noted as a crash-resume on the next start; the driver log and trace are its evidence.
- Judgement moves from a deterministic filer to the reflection agent, so a real lyx fault behind a halt can go unfiled if the agent misreads it.
  The note still names the trace file, the run's status and history stay on disk, and `lyx selfreport create` remains available for manual filing.
- `lyx selfreport create`, its default `bug` label and the board-intake calls in `selfreportengine/inbox.go` are untouched.
  Stale machine-local `selfreport-filed.json` files stay on disk; nothing reads them.
- `internal/loomcli/gotoentry_test.go` and a few other tests changed only to match the new `DetectCrashResume` return.

## To check by hand

- Not run against a live run: a `lyx loom start` resume after moving a run between machines should write no `loom-crash-resume` note.
  A bare `lyx loom run` or `lyx loom step` on a run killed mid-producer should write one.
- Issues #366–#382, filed by the retired filer, are not closed or edited here; they are left for board intake.
