# Round 2 fixer-report — discussion

Target: `_lym/discussion/decision-record.md`.
The support log is unchanged.

## Changes

- F1 (BLOCKING), fixed as proposed: D4's fence is now max(3, one longer than the longest backtick run in the body) backticks long, D4's Bound matches, and the D4 acceptance row adds a body with no backticks and one with inline code only, each holding `## Decisions` and `## Open risks`.
- F2 (MEDIUM), fixed as proposed, then extended by the self-check: Record-Write writes each file to a sibling temporary file and renames it into place, and it removes a leftover temporary file before it writes, so the discussion commit, which stages the whole discussion directory, never takes one. The D4 acceptance row covers the torn-temporary-file resume.
- F3 (MEDIUM), fixed as proposed: D13's check runs in `PrepareRunSettings` on each module's raw object, after `moveRunSettings` and before `module.Migrate`. The keys are pruned through the `RetiredKeys` reconcile prune alone, and never added to `retiredLoomKeys` or dropped by darn's `MigrateConfig`.
- F4 (MEDIUM), fixed as proposed, then extended by the self-check: in D6, with no parent, a `Decision for <slug>:` note does not settle a stop. The stencil drops its note-into-record route, and its stop report names `lym loom decision add` then `lym loom resume`. A D6 acceptance row covers a no-parent run with a note and no `Added after Discussion` entry halting again, plus the stop report's way forward.
- F5 (MEDIUM), fixed differently. Proposed: all three darn refusals name `goto --to Record-Write` then `step`. Not made: the self-check found that this route leaves a run past Darn-Write on Drift-Baseline after code was committed, which D4's in-flight bullet forbids. It would also commit an edit made outside `decision add`, which D6 forbids, and it never restores a dropped entry. Instead:
  - a record not committed at HEAD, which only a crash inside Record-Write leaves, names `lym loom resume`;
  - a record that differs from its committed copy names restoring the committed copy in the `_lym` weft repository;
  - a committed record missing a historical entry names restoring and committing the entry there;
  - `decisionWayNoRecord`'s darn text drops the Darn-Write session and the board-note route;
  - `decisionWayEntryLost` drops the Darn-Write session.

  The D4 acceptance row pins these.
- F6 (MEDIUM), fixed as proposed, form 1: the no-commit route now says the earlier pass's scope record and capture stand. It adds which scope that is: from the board entry on a run that passed Drift-Baseline before landing, or from no subject on one resumed without the goto.
- F7 (MEDIUM), fixed as proposed: `missingSections` counts a required heading only on a line outside a fence, by the same CommonMark rule. The D5 acceptance row adds the case of a body holding `## Open risks` with the real heading removed. The Scope bullet for the discussion spec names it. D5 states that the stricter check applies to tapestry records too.
- F8 (MEDIUM), fixed as proposed, form 2: D13's Bound, placed after its Rationale, says a hub base value is dropped and pruned with no carry-over. The orch moves it into the loom recipes' profile blocks under `shed`, never into `shed.yaml`, since `shed.yaml` would widen it to every recipe. The same risk is in Open risks.
- F9 (LOW), fixed differently. Proposed: the agent "may record a scope from the board entry alone". Not made: under `subject: decision-record`, `subjectFields.reading()` (`internal/driftengine/seats.go`) gives the agent only the record's path and no board route. D4 and Open risks now say the agent either halts Stuck with drift's ways forward, which loop on the missing record, or records a scope from no subject, and that the way forward is the goto to Record-Write.
- F10 (LOW), fixed differently. Proposed: a row for a run at Preflight after `lym loom goto --to Preflight`. Not made: Tapestry-Preflight already waived that re-entry (`waivesHalfFinished`), so the row pins no new edge. The D10 row instead pins a run at Preflight whose status, edited outside the seed, resume and goto routes, holds history past Preflight, a set `start_sha` or `pause_requested`, and proceeds.

## Self-check

- fix-claims, MEDIUM, the agent pointed at a missing record has no board route: fixed (D4 Drift-Baseline bullet, Open risks).
- fix-claims, LOW, darn has no Migrate drop list: fixed as proposed (darn's `MigrateConfig` named).
- fix-consistency, MEDIUM, `decisionWayNoRecord` names the Darn-Write session and a board note: fixed.
- fix-consistency / fix-way-forward, MEDIUM (merged), the stop report still names a board note: fixed (D6 and its acceptance row).
- fix-consistency / fix-way-forward / fix-proposal, MEDIUM (merged), the `goto --to Record-Write` then `step` route strands a run past Record-Write and commits an edit made outside `decision add`: fixed with per-refusal ways forward.
- fix-way-forward, LOW, the restore-entry way forward runs into the differs-from-committed refusal: fixed, since the way forward now restores and commits the entry.
- fix-consistency, MEDIUM, the no-commit bullet's justification is false for the no-goto path: fixed.
- fix-consistency, LOW, D13's Bound sits mid-decision and the rationale reads as unbounded: fixed (Bound moved after Rationale, and the rationale bounded to profile, run settings and local file).
- fix-proposal / fix-way-forward, MEDIUM (merged), the torn temporary file is committed: fixed (Record-Write removes it before writing).
- fix-proposal, LOW, the D10 row pins no new edge: fixed (row rewritten).
- fix-proposal, LOW, moving tuned values into `shed.yaml` widens them: fixed (profile blocks only).
- fix-proposal, LOW, the stricter section check on tapestry is unstated: fixed (D5 sentence).

Every fork returned a report.

## Disputed

None.

## Deferred

None.
