# Fixer report — Discussion-Review round 1

All eight findings are fixed; `lyx loom validate-discussion` exits 0 after the edits.

## Changes

- **F1 (MEDIUM), `strand_name` retirement.**
  `decision-record.md` Constraints: the bullet that cited a nonexistent "retired-key mechanism" under the Config Strictness Invariant now states the behaviour.
  The key leaves `Config` and both templates, a `reed.yaml` that still carries it loads with the key ignored, and `lyx config reconcile` removes it.
  Acceptance criteria gain a `reed config` bullet pinning that load.
- **F2 (MEDIUM), session-name provider degradation.**
  Mirrors: the watchdog decision now says a missing or unreadable transcript logs that the name could not be read and counts as no drift.
  Open risks: the `custom-title` bullet now gives the real degradation, a `/rename` retyped and logged on each idle tick.
  The provider criterion separates "no `custom-title` entry" (drift) from "missing or unreadable transcript" (logged, no drift).
- **F3 (MEDIUM), "the ly hub workflow documents it".**
  Parentage: the clause is gone. The bullet now says the env-prefix path is not a deliverable of this task, since no repo doc carries it and the hub session applies it from this record.
- **F4 (LOW), the `self` seed.**
  The Parentage decision says the default run is run-id `self`, resolved by `shedrun` to the slug, with its seed at `_lyx/shed/<slug>/seed.json`, read through `shedrun`.
  The matching Auto-mode assumption and the reed acceptance criterion now name the default-run seed.
- **F5 (LOW), support log against the hub review.**
  Interview turn 15 keeps its original pick and gains a line pointing to the hub review that superseded it in part.
  The `--parent` rejection now gives the env prefix as its reason.
  Q13 now gives the revised resolution.
- **F6 (LOW), criteria gaps.**
  The uncoded-hub refusal criterion now also covers the refusal for a task worktree whose name fails the slug grammar.
  The docs criterion now names the reed and fabric package docs.
- **F7 (NIT), restated seed criterion.**
  The last acceptance criterion, which repeated the hand-started hub seed case, is removed. The Seed criterion already covers it.
- **F8 (NIT), dated history.**
  The "(hub review, 2026-10-02)" attribution is dropped from the Parentage rationale. The Delivery rationale drops its date and keeps its evidence: SendMessage already addresses sessions by name.

## Deferred

None.
