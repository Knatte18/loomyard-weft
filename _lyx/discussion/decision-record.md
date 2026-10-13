# Decision record — shuttle-gate-early-done

## Goal

Shuttle must not take a writer's turn as over while the model is still working on it, and a twill Plan-Write must not end as Done before its lens review is recorded.
In both bench runs (`twb-ftp-r-1`, `twb-ftp-s-1`) a task-notification turn that started right after a turn boundary was released by the 10 s idle-pane override (`turnStartIdleOverride`) while the model was thinking, and the Plan gate passed; the transcript's newest main-chain entry was the unanswered notification throughout.

## Scope

In:

- The unmatched-turn-start hold in `internal/shuttleengine` (`turnStartHold`), which the gate boundary (`startHoldReleased`), every verified send's idle wait and `Runner.SessionIdle` (the orch watcher's idle probe) all decide through.
- A transcript reading in `internal/shuttleengine/claudeengine` that tells a turn in flight from a turn that is over.
- The Plan-Write row's gate under twill: it refuses Done until `lens-review.md` holds non-whitespace text in the plan directory, plus the parity mode `lyx loom validate-plan --lens-review`.
- A run-record field in `internal/shuttleengine` (`RunState`) holding the spec's fork-subagent authorization at spawn, tolerant of its absence on an older record, which reads as not twill; the Plan-Write gate and the fork audit at finalize (`Run.finalize`) both read it.
- The twill directive's final self-check step names that mode for the Plan host.
- Docs: `internal/shuttleengine/doc.go`, `claudeengine`'s doc, `internal/loomengine/plan.go`'s file doc, `docs/overview.md` (the `validate-plan` modes and the twill writer paragraph), and `pattern/PATTERN-gate-self-check-parity.md`, whose Plan-Write row is rewritten, not only extended.

Out:

- The pane classifier (`Claude.IdleSession`, `RunningTurn`) is not changed; the fix does not depend on how it reads a thinking model.
- No mechanical lens-review check for twill Discussion-Write: its disposition is a turn inside the support log, a file written before the forks run, so presence cannot be told from the file.
- No change to `turnStartIdleOverride`'s length, the gate's re-prompt budget, or the Plan spec's `OutputFiles`.
- No backfill of a lens review for a landed run.

## Decisions

### D1 — An in-flight turn vetoes the idle-pane override

Decision: `turnStartHold.holds` releases a turn start by the idle-pane override only when the engine does not read that turn as in flight.
The order is: a reported interrupt releases at once (unchanged); a turn the engine reads in flight holds and resets the idle timer, whatever the pane shows; otherwise the existing pane-idle timer and `turnStartIdleOverride` decide.
The veto does not apply when a session-start signal of any source lies after the turn start in the events file: a resumed, cleared or restarted session cannot still be running that turn, so such a start falls to today's override.
A reading the engine cannot make (no capability, no transcript path in the turn start's payload, an unreadable transcript) also leaves today's behaviour.
The veto logs nothing per poll; the existing release Warn stays the only line.
Rationale: Claude Code writes a turn's prompt to the transcript at turn start and the model's reply only when a message completes, so the transcript tells a running turn from an orphaned turn start where the pane and the hook-derived state cannot.
Bound: the veto only lengthens a hold and never lets a send, a gate evaluation or an idle report through earlier.
A turn read in flight forever (a hung model) holds until a turn end, a reported interrupt, a later session start, the verified send's `send_ready_timeout_s` (`ErrSessionBusy`, retried on later ticks without spending the gate's budget, as today) or the run deadline, whose classification is unchanged.
`Runner.SessionIdle` reports busy for as long as the turn reads in flight, where today it reports idle after 10 s.

### D2 — The in-flight reading lives in `claudeengine`

Decision: the reading is a new optional engine capability beside `SessionProber`, so an engine without it keeps today's behaviour with no change to existing implementers or fakes.
`shuttleengine` sees only in flight, over, or unknown.
`claudeengine` reads the transcript the turn start's `UserPromptSubmit` payload names, backward from its end over complete lines only, as `TurnStartInterrupt` does.
The turn is in flight when the newest main-chain user or assistant entry is a user entry that is not the interrupt marker (a prompt, a task notification, a tool result), or an assistant entry whose message's stop reason is `tool_use`.
It is over when that entry is an assistant entry whose stop reason is `end_turn` or `stop_sequence` (which Claude Code's synthetic API-error entry carries).
An assistant entry whose stop reason is absent or null, or is any value other than `tool_use`, `end_turn` or `stop_sequence`, reads unknown; a message cut off mid-stream leaves its lines with a null stop reason.
An assistant entry is classified by its message's stop reason, never by its last content block: every line of a completed message split over several transcript lines carries the message's final stop reason, so a thinking or text line ahead of its `tool_use` line already reads in flight.
The interrupt marker reads unknown, since D1 releases on a reported interrupt first.
Sidechain (subagent) entries and non-message entries (`system`, attachments, summaries) are skipped; a line that does not parse is skipped.
Rationale: `PATTERN-shuttle-provider-seam` keeps transcript shape knowledge in `claudeengine`; reusing the backward reader keeps one transcript-reading path.

### D3 — The twill Plan-Write gate refuses Done without a lens review

Decision: the Plan-Write row's gate, when that row's spawn ran with the twill directive, adds one blocking finding when `planparser.LensReviewFileName` is absent from the plan directory or holds only whitespace.
The finding names the file's absolute path and says to record the lens review there as the directive describes; it is a validation finding, not a refusal, so it adds no `refusal-spec.md` row.
The run record (shuttle's `RunState`) records the spec's `ForkSubagents` at spawn, which `PlanSpec` sets exactly under twill.
The Plan-Write gate reads it from the run it judges, attached or spawned, and never from the spec composed for that Call, so a `plan_producer` edit between spawn and gate changes nothing.
The run's fork audit at finalize reads the same recorded authorization instead of the attaching Call's spec, so one run has one answer to whether it was authorized to fork.
The gate evaluation receives that run's recorded authorization, since the `GateSpec` itself is fixed at wire time.
The Plan-Burler-Fix row shares the `plan` gate kind today; the wiring tells the Plan-Write row apart, so Plan-Burler-Fix, PR-Rework and a non-twill Plan-Write keep today's check set.
The check is one exported `loomshed` function that the gate closure and `lyx loom validate-plan --lens-review` both call; that mode runs `ValidatePlan`'s full set plus the lens-review check, and the flag-less mode ignores `lens-review.md`.
Rationale: a twill plan without a recorded lens review fails at the gate instead of passing silently.
Bound: it adds a refusal only, and checks presence, not content or timing; a lens review written before the forks return passes.
A writer that never records it spends the gate's existing re-prompt budget and ends as that budget ends today.
A relaunched writer keeps an existing `lens-review.md` (the plan-dir rotator moves it only on a fresh spawn), so a resumed run is not re-prompted for it.

### D4 — The twill Plan writer runs the parity mode last

Decision: the twill directive gains a second self-check value for its final step (step 8): `lyx loom validate-plan --lens-review` for the Plan host; the pre-ready step (step 3) keeps `lyx loom validate-plan`, and the Discussion host uses `lyx loom validate-discussion` for both.
Rationale: the writer meets a missing lens review in its own turn instead of in a gate re-prompt; the pre-ready check cannot require a file written only after the forks return.

## Constraints

- `PATTERN-shuttle-provider-seam`: transcript entry shapes, stop reasons and the interrupt marker stay in `claudeengine`.
- `PATTERN-gate-self-check-parity`: D3's verb mode and its rewritten `## Pairs` row land in the same task.
- `PATTERN-completion-signal`: no negative-outcome path changes.
- `PATTERN-sole-parsers`: `lens-review.md` is checked for presence only, never parsed.
- `PATTERN-tolerant-records`: the transcript reading tolerates unknown keys and unknown entry types.
- `PATTERN-test-speed`, `PATTERN-test-economy`: fake clocks; the unmatched-turn-start rows in `run_test.go` and the existing gate tests are extended before new top-level tests; transcript fixtures are files.
- The twill directive's marker set is pinned by stencil tests (`markers_test.go`, `claims_test.go`); D4 updates them.
- `twill-fork-timing` (board) also edits the twill directive stencil and `internal/loomengine/twill.go`; D3 and D4 hold under either spawn point, and whichever lands second rebases onto the other.

## Auto-mode assumptions

- Claude Code writes the turn's prompt line to the transcript at or right after `UserPromptSubmit`, and a completed assistant message's lines carry its final stop reason; a message cut off mid-stream leaves a null stop reason on its lines.
## Open risks

- An assistant message split over several transcript lines: a thinking line that already carries a turn-ending stop reason, read before the text line lands, reads as over; the pane must still read idle for the whole override with no turn end, so the exposure is a long final text block under an idle-looking pane.
- A `UserPromptSubmit` that fires before the prompt line reaches the transcript reads the previous turn's end as newest, so the turn reads over until the line lands; this falls back to today's override.
- A Claude Code release that renames entry types or stop reasons turns the reading unknown or wrong, and a second engine without the capability inherits today's early release.

## Acceptance criteria

- An unmatched turn start whose transcript reads in flight holds the gate boundary and a verified send against a pane that reads idle past `turnStartIdleOverride` (fake clock); a later turn end then lets the gate evaluate at that boundary.
- The bench incident shape — a turn end, then a turn start from a task notification whose user entry is the transcript's newest main-chain entry — never reaches gate evaluation before its turn end.
- `Runner.SessionIdle` reports busy while that turn reads in flight.
- An unmatched turn start followed by a resume, clear or startup session start, with the dead turn's prompt still newest in the transcript, releases by today's override.
- An unmatched turn start whose transcript's newest main-chain entry is an assistant entry ending the turn still releases after `turnStartIdleOverride` with one Warn.
- A reported interrupt still releases at once, ahead of the in-flight reading.
- A turn start whose payload names no transcript, or names one that cannot be read, keeps today's override.
- `claudeengine`'s reading over transcript fixtures cut from the bench transcripts: newest entry a prompt, a task notification, a tool result, an assistant tool use → in flight; an assistant end of turn → over; the interrupt marker → unknown; a thinking or text line whose stop reason is `tool_use`, with no `tool_use` line yet → in flight; a thinking line whose stop reason is `end_turn` → over; a synthetic API-error entry → over; an assistant line with a null stop reason, and one with an unlisted stop reason such as `max_tokens` → unknown; trailing sidechain and `system` entries, a malformed last line and a line with no trailing newline are skipped.
- A twill Plan-Write gate over a valid plan with no `lens-review.md`, an empty one or a whitespace-only one does not pass, and its finding names the file's path; with text in the file it passes.
- A non-twill Plan-Write gate, a Plan-Burler-Fix gate and a PR-Rework gate pass a valid plan with no `lens-review.md`.
- A twill-spawned Plan-Write run attached by a later Call whose `plan_producer` now reads single still gets the lens-review check; a single-spawned run attached by a later Call whose `plan_producer` now reads twill does not.
- A twill-spawned run attached by a Call whose spec does not set `ForkSubagents` is still fork-audited when it ends Done; a single-spawned run attached by a Call whose spec sets it is not.
- `lyx loom validate-plan --lens-review` and the twill gate report the same findings over the same plan directory; `lyx loom validate-plan` ignores `lens-review.md`.
- The twill Plan prompt's step 8 names `lyx loom validate-plan --lens-review` and its step 3 `lyx loom validate-plan`; the Discussion twill prompt is unchanged.

## Notes for the plan writer

- The incident traces and transcripts are under the hub prime's `.scratch/twill-fork-timing-plan/after/` (`ftp-r-1`, `ftp-s-1`); the bench pairs themselves will be torn down.
- `turnStartHold.holds` is the one place the gate boundary, the send's idle wait and `SessionIdle` decide a release; `unmatchedTurnStart` pairs only turn starts and turn ends.
