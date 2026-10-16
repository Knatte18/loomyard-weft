# Fixer report — plan, round 1

## Changes

- **F1** (MEDIUM, scope): fixed differently in placement.
  The premise holds: `internal/orchcli/start.go:72-73` sets `AllowAgentTool` and `ForkSubagents` on the orch spec, and `internal/shuttleengine/claudeengine/signals.go:124` takes each hook line's `session_id` unfiltered.
  - `03-start-continue.md`: the proposed clause was not spliced into the middle of the `Bound:` sentence, where it would be followed by the unconditional "so it takes over no session orch did not launch, adopt or bind".
    Instead that claim now reads "so, within the reliance below, it takes over no session…", and the reviewer's reliance text follows as its own sentence, so the hedge qualifies the guarantee instead of being contradicted by it.
  - `02-watcher-records-session-id.md`: the `Watcher.recordSessionID` bullet gains that it accepts whatever id the newest id-naming signal of the strand's own run names, bounded to that run's events file, with the same provider reliance; the `internal/orchengine/doc.go` paragraph gains that `# Open risks` records this risk.
- **F2** (LOW, scope): fixed as proposed in `04-status-session-id.md`.
  Dropped `internal/orchcli#TestStop` from the Edit list and its Tests bullet, dropped "and the record outlives `lyx orch stop` (D4)" from the Intent's first line, ended the stop bullet with the pointer to card 3's continue after stop row, and ended the ImpactSummary with "stop's behavior is unchanged".

`lyx loom validate-plan` passes on the edited plan.

## Disputed

None.

## Deferred

None.
