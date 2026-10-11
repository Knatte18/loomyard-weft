# Advisor-1 notes — dbench-ohom

## Module boundaries

- `internal/shuttleengine` owns the turn-end wait (`wait.go`), `Spec.AwaitedShellPrefixes`, `Result.EndedShells` and the gate re-prompt text (`gate.go` `gateRepromptText`), shared by every gated run.
  Provider specifics stay in `claudeengine` (`PATTERN-shuttle-provider-seam`).
- `internal/websterengine` owns Master's spawn spec (`runlevel.go`), the verify gate entry, recovery spawn and prompt rendering (`recoverbatch.go`, `render.go`), run-write attribution (`runwrites.go`) and `State`/`BatchState`.
- `internal/webstercli` owns the `recover-batch` verb and its exit code.
- `contracts/stencils/webster/` holds Master, implementer body, recovery prefix and verify-fix stencils, read through `stencilstore`.
- `internal/planglyph` owns `DoneChecks` (Create/Delete/Rename done verdicts) and the plan-time resolve findings (`resolve.go`, `create.go`); it calls quarry, an external module (`github.com/Knatte18/quarry` v0.2.1).
- `contracts/specs/refusal-spec.md` holds the webster refusal rows.

## Findings per item

1. Since 5a8be0274 no shell expires and ended shells are recorded (warning, summary section, friction note), but a gated run's waiting turn end with both outputs present still counts at once when every outstanding task is an unawaited shell.
   Master declares only `lyx webster recover-batch` as awaited.
   The `lyx gate test --tags integration` shells in #505/#509/#517 are most likely forks' shells (the verify-fix stencil and the implementer body both background `lyx gate test`), which surface on Master's outstanding list; Master's own stencil never runs a gate.
   The verify gate reruns the full plan verify, so a `done` is still backed by a verify pass; the discarded shell's result is redundant but not reported as such.
2. Root cause in Go: `gateRepromptText` tells every gated agent "read it, fix every finding, and end your turn", which contradicts Master's stencil ("Do NOT localize or fix the failure yourself").
   A verb that spawns the fixer cannot work: forks are in-session Agent-tool spawns.
3. Root cause: `recordedSessions` reads `State.MasterSessionID` and each `BatchState.SessionID`, which is empty for a recovery record; no recovery strand's session is ever audited, so every recovery's writes read "Not written by this run".
   The recovery strand's session id is reachable through the prior record's `ShuttleRunDir` (run.json) until a done recovery removes it.
3b. `recover-batch` terminal stuck/dead digests go through `output.Ok` (exit 0); only refusals and the `batch_failed` path exit non-zero.
4. The recovery prompt is the recovery prefix plus the shared implementer body; the body already lists card and batch gates and bans only mutating `lyx fabric` verbs.
   Agents' settings refuse raw tmux-tier and llm-tier `go` runs (`claudeengine/settings.go` `steerRawGoDeny`), not `lyx gate test --tags tmux`; why the recovery could only compile-check tmux steps the code does not say.
5. #511 is almost certainly #500's cause, not an unexported-identifier limit: today `cmd/testtiming#rusageStats` and `internal/testkit/tmuxkit#tmuxAllowed` resolve `found`, and the landed tree shows the workaround (per-OS `rusage_linux.go`/`rusage_other.go` behind a single `rusageStats`; `allow_tmux.go` sets the var in `init` instead of redeclaring it).
   The `create-not-done` detail says "still does not resolve" for an `ambiguous` answer too, which is what misled the report.
6. `DoneChecks` passes Create only on found/multipart; plan-time `resolve.go` reports `glyph-ambiguous` (blocking) for an Edit/Uses target, and `create.go` reports `create-already-exists` for an ambiguous Create.
   Delete's `stillExists` treating ambiguous as present is correct and must stay.

## PATTERN entries touched

- `PATTERN-refusal-way-forward`: a changed `create-not-done` message and any new non-zero `recover-batch` exit with `ok:false` are webster refusals needing a spec row, a way forward and a test.
- `PATTERN-no-denied-recovery`: the recovery-prompt gate statement must name only allowed commands; no raw tmux-tier `go` run.
- `PATTERN-shuttle-provider-seam`: a caller-supplied re-prompt or prefix stays provider-neutral caller data on `Spec`/`GateEntry`.
- `PATTERN-completion-signal`: any change to the turn-end wait must not add a negative-verdict site without `allOutputFilesExist`.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`: stencil edits only.
- `PATTERN-glyph-conversion-chokepoint`: a build-constraint check reads candidate files quarry names; it converts no glyph, so it complies if it uses the answer's `file` field and never derives a path from a glyph.
- `PATTERN-sole-parsers`: untouched unless plan parsing changes.
- `PATTERN-test-economy`, `PATTERN-test-speed`, test budget (`cmd/lyx/testdata/test-budget.yaml`): new tests extend existing table tests where possible.
- Docs in the same commit: `shuttleengine/doc.go`, `websterengine/doc.go`, `planglyph/doc.go`.

## Board entries touched

- `proc-child-lifetime`: gate child lifetime; not a conflict, but lyx still cannot kill a provider shell (no pid).
- `gate-test-hygiene`: Publish fix route for verify failures; no conflict.
- `kick-start-pack`, `enclose-kickstart`: quarry consumers, no conflict.
- `dbench-*` entries are benchmark copies of this task.

## Conflicts seen

None between the board entry and PATTERN.
Item 5's premise (unexported identifiers do not resolve) is contradicted by the tree; the item reduces to the refusal message plus item 6.
