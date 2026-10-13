# shuttle: hold in-flight turns; twill Plan-Write needs a lens review

Shuttle no longer takes a writer's turn as over while the model is still working on it, and a twill Plan-Write no longer ends as Done before its lens review is recorded.

In two bench runs (`twb-ftp-r-1`, `twb-ftp-s-1`), a task-notification turn that began right after a turn boundary was released by the 10 s idle-pane override (`turnStartIdleOverride`) while the model was thinking.
The Plan gate then passed with no `lens-review.md` written.

## What changes

- **In-flight veto.**
  `turnStartHold.holds` in `internal/shuttleengine` now releases an unmatched turn start by the idle-pane override only when the engine does not read that turn as in flight.
  A reported interrupt still releases at once.
  A session-start signal after the turn start lifts the veto, since a resumed, cleared or restarted session cannot still be running that turn.
  When the engine cannot make a reading (no capability, no transcript path, unreadable transcript), today's override applies.
  The gate boundary, every verified send's idle wait and `Runner.SessionIdle` all decide through this one place, so `SessionIdle` now reports busy while the turn reads in flight.
  The veto only lengthens a hold.
- **Transcript reading.**
  `claudeengine` gains an optional engine capability beside `SessionProber` that reads the transcript named in the turn start's `UserPromptSubmit` payload, backward over complete lines.
  The newest main-chain entry decides: a user entry other than the interrupt marker, or an assistant entry with stop reason `tool_use`, is in flight; `end_turn` or `stop_sequence` is over; anything else, including a null or unlisted stop reason, reads unknown.
  An assistant entry is classified by its message's stop reason, never by its last content block.
  Sidechain entries, non-message entries and unparsable lines are skipped.
- **Twill lens-review gate.**
  The Plan-Write row's plan gate adds one blocking finding when the run was spawned under twill and `lens-review.md` is absent or whitespace-only in the plan directory.
  The finding names the file's path and is a validation finding, so `refusal-spec.md` gains no row.
  Plan-Burler-Fix, PR-Rework and a non-twill Plan-Write keep today's check set.
- **Parity mode.**
  `lyx loom validate-plan --lens-review` runs the full plan check plus the lens-review check through the same `loomshed` function the gate calls; the flag-less mode ignores `lens-review.md`.
  The twill directive's final self-check step names this mode for the Plan host; the pre-ready step and the Discussion host's prompt are unchanged.

## Design decisions

- `RunState` records the spec's fork-subagent authorization at spawn, and an older record without the field reads as not twill.
  The Plan-Write gate and the fork audit at finalize both read this recorded value, so a `plan_producer` edit between spawn and a later attaching Call changes nothing and one run has one answer to whether it was authorized to fork.
- The lens-review check tests presence only, never content or timing; a file written before the forks return passes.
- A writer that never records the review spends the gate's existing re-prompt budget.
  A relaunched writer keeps an existing `lens-review.md`, so a resumed run is not re-prompted.
- The pane classifier is unchanged, as are `turnStartIdleOverride`'s length, the re-prompt budget and the Plan spec's `OutputFiles`.
- Twill Discussion-Write gets no mechanical lens-review check: its disposition is a turn in the support log, written before the forks run.

Docs updated: `internal/shuttleengine/doc.go`, `claudeengine`'s `doc.go`, `internal/loomengine/plan.go`, `internal/loomcli/doc.go`, `internal/loomshed/doc.go`, `docs/overview.md`, and the Plan-Write row of `pattern/PATTERN-gate-self-check-parity.md`.

## To check by hand

- Run a real twill Plan-Write and confirm a task-notification turn that starts after a turn boundary holds the gate until its turn end.
  The change is tested against fake clocks and transcript fixtures cut from the bench transcripts, not against a live Claude Code.
- An assistant message split over several transcript lines: a thinking line that already carries a turn-ending stop reason, read before the text line lands, reads as over.
  Release then still needs the pane to read idle for the whole override, so the exposure is a long final text block under an idle-looking pane.
- A `UserPromptSubmit` that fires before the prompt line reaches the transcript reads the previous turn's end as newest, so the turn reads over until the line lands; this falls back to today's override.
- A Claude Code release that renames entry types or stop reasons turns the reading unknown or wrong, and a second engine without the capability keeps today's early release.
- This change and the `twill-fork-timing` board entry both edit the twill directive stencil and `internal/loomengine/twill.go`; whichever lands second rebases onto the other.
