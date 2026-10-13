# Decision record — run-window-bugs

## Goal

Fix the seven defects bundled on board entry `run-window-bugs` so none recurs, and record the weft push finding the entry asks for.

## Scope

In:

1. `shuttle-ready-blank-pane` — `start`'s startup probe reads readiness from the run's events ahead of the pane capture.
2. `shuttle-color-verified-send`, remaining bullet — the launch line carries the skill-load message when skills are named.
3. `loom-checklist-stale-render` — the shared status pane loop clears screen and history on every redraw, and a resize wakes it at once.
4. `weft-push-refused-retry` — the finding, recorded in `gitrepo`'s push retry doc comment.
5. `reed-socket-test-fd-sweep` — the remaining close-a-listener stale socket in tests moves to the datagram helper, which moves to a test kit.
6. `agent-gofmt-verb` — a `lyx loom fmt` verb and one line in the edit directive naming it.
7. "Teardown races a reed lock" — a bounded retry of the task worktree's contained removal.

Out:

- Reed's layout, `clampToFit` and a minimum pane height.
- A typed send held because its pane is too short to show the input box.
- Wait's and attach's startup probe for a run not yet started: it keeps the capture rule.
- Any change to weft push behavior or its log level.
- Killing or enumerating reed processes bound to a pair.

## Decisions

### D1 — `start`'s startup probe reads readiness from events first

In `awaitStartup`, called from `start`, a launch is ready once its events file holds a session-start signal and, after it, a turn-start signal or a tool event.
The probe checks this before it captures the pane, and a ready answer from events sets and persists `Started` exactly as a ready capture does.
The capture path stays as it is for a launch the events do not yet prove ready, so a recognized gate is still dismissed.
Rationale: a turn start or a tool call proves the provider is past every startup gate and working, which a capture of a collapsed or blank pane cannot.
After D3 every launch through `start` carries a first turn, fresh or resumed (a resume spec's launch line already carries the prompt pointer), so the events proof is available on every `start`.
Bound: the proof is read only in `start`'s probe, whose events file is created fresh in the run's new directory, so only this launch's hooks can satisfy it.
Wait's and attach's probe of a not-yet-started run, whose events file may hold an earlier incarnation's lines, keeps today's capture rule.
A trust or bypass gate holds the first turn, so no turn start reaches the events file while a gate is on screen.
An engine that does not record session starts (`RecordsSessionStarts` false) keeps the capture-only probe.
The signals are read through the engine's session-signal and event parsers, never by Claude hook name, per `PATTERN-shuttle-provider-seam`.

### D2 — The launch line carries the skill-load message

A spec that names skills launches with the engine's skill-load message as the launch argument, in place of today's empty input box.
The first load turn is then not typed: shuttle waits for its turn end in the events file, then types the color, then the retry turn for missing skills, then the prompt pointer, each through the verified send.
The first load turn is classified from the transcript as today, and its timeout and a pane death during it end as today's do: skipped skills with a warning, or a died startup.
A spec without skills launches with the prompt pointer, as today, and the color is typed after `awaitStartup` as today.
Rationale: every launch through `start` then carries its first turn, which removes one typed send and gives D1 a turn start on every launch.
The skill-load message must stay one line within `maxLaunchPromptBytes`.
This changes `PATTERN-role-skills-typed`, whose entry says lyx "types" a named skill: the entry is reworded to "lyx delivers it, on the launch line or typed", with the same no-stencil rule.
`docs/overview.md`'s skills line and the comment in `contracts/stencils/roleskills_test.go` change with it.

### D3 — The status pane clears screen and history on every redraw, and a resize wakes it

`statuspane`'s redraw writes cursor home, erase the whole screen and erase the scrollback history before the frame, in place of erasing per line and below the cursor.
A terminal resize wakes the pane loop at once, without waiting out the back-off, on platforms with a resize signal; elsewhere the loop keeps its poll.
The fix lives in `statuspane`, so the loom checklist and batten's overview pane both get it.
Rationale: a full clear leaves nothing of an earlier render on screen or in the history tmux pulls back onto a growing pane, and the immediate redraw closes the window between a resize and the next poll.

### D4 — The weft push refusal is a remote race; the retry is the intended path

`! [remote rejected] main -> main (failed)` is the remote refusing a ref update that raced another update of the weft's `main` (board writes and other runs' record pushes), not a stale local ref.
`gitrepo.PushWithRetry` (`3a627c00d`, 2026-10-11) already retries exactly that refusal inside the push call, and the 2026-10-12 16:04 sighting is its own retry log line.
No behavior or log level changes: the Warn stays, since it is the signal that surfaced the race.
The finding goes into `pushretry.go`'s doc comment, so the next reader of a retry line learns its cause there.

### D5 — One stale-socket helper, in a test kit

The datagram-built stale socket (`bindStaleUnixSocket`, private to `reedengine`'s tests) moves to a test kit per `PATTERN-testkit`.
`tmuxkit`'s dead-socket test builds its dead socket with it instead of closing a listener, and drops the fork-inheritance wait it carries for that.
Rationale: a closed listener is the form a parallel test's forked child can keep answering; that test is the one remaining instance.

### D6 — `lyx loom fmt` formats named Go files

`lyx loom fmt <file>...` rewrites each named `.go` file in place to its `go/format` form, in-process, with no `gofmt` binary.
It resolves each path, symlinks and junctions included, before checking that it is a `.go` file inside the current worktree.
It checks and formats every named file before writing any: a path outside the worktree, a non-`.go` path or a file that does not parse refuses the whole call with nothing written, and the envelope names each refused file.
It requires at least one file, writes a file only when its content changes, keeps its mode, and reports the changed files in its JSON envelope.
The edit directive (`contracts/stencils/edit/edit-directive.md`) gains one line: a Go file's formatting is fixed with `lyx loom fmt <file>`.
Rationale: `cmd/lyx/gofmt_test.go` fails a misformatted file, and the edit directive forbids every rewriting tool, so an agent had no sanctioned way to fix formatting; an in-process verb works on every OS and is narrower than allowing `gofmt -w`.
Bound: the verb rewrites only `.go` files the caller names inside its own worktree, and only to their `go/format` form.
`loom` is its home beside `lint-comments`, the other agent-facing Go source verb; a role in a repo with no Go files never needs the line.

### D7 — The task worktree's contained removal retries a directory a late writer refilled

When the contained removal of the task worktree directory, after `git worktree remove`, fails because the directory is not empty, the removal waits briefly and removes the same target again, at most two more times.
The retry lives inside that one removal call in `fabricengine`'s destroy path, after the ownership and dirtiness checks have passed for the target; it repeats only the removal primitive on the same contained target.
The failure is matched by its error value (not-empty directory), not by message text, on every OS.
Rationale: the reed session has ended before removal starts, so the writers left are short-lived processes a tmux hook or the run window's attach started as the session closed (the 2026-10-12 sighting's two empty lock files), and waiting them out is reliable.
A re-entered `Remove` cannot finish this state, since it treats a leftover directory that is no longer a registered worktree as a stray path and leaves it on disk.
Bound: only the not-empty failure is retried, only on the target whose checks already passed, with an injected sleep; a failure after the last attempt is returned as today.

## Constraints

- `PATTERN-shuttle-provider-seam`: the skill-load launch argument and every Claude-specific event name stay under `claudeengine`.
- `PATTERN-completion-signal`: every negative startup answer still consults the output files first; D1 only adds a positive answer.
- `PATTERN-pane-text-injection`, `PATTERN-guarded-interrupt`: the sends that stay typed keep using the verified send.
- `PATTERN-role-skills-typed`: changed by D2, reworded in `PATTERN.md` in the same commit.
- `PATTERN-fabric-destruction-chokepoint`, `PATTERN-pair-teardown`: D7's retry stays in `fabricengine`'s destroy path, after the session end.
- `PATTERN-edit-directive`, `PATTERN-no-denied-recovery`: the directive still renders through `editdirective`, and `lyx loom fmt` is not denied by the agents' settings.
- `PATTERN-cli-cobra`, `PATTERN-cwd-resolution`: `lyx loom fmt` is a cobra subcommand in `loomcli` resolving its root through `lyxcwd`.
- `PATTERN-testkit`: the stale-socket helper exists once, in a kit, imported only from tests.
- `PATTERN-test-speed`, `PATTERN-daemon-wakeups`: injected clocks and sleeps; the pane loop's idle back-off stays.
- Same-commit docs: the `doc.go` of `shuttleengine`, `claudeengine`, `statuspane`, `loomcli`, `gitrepo` and `fabricengine` where behavior changes, `docs/overview.md`, `PATTERN.md`, and the operator command index for `lyx loom fmt`.

## Auto-mode assumptions

- Claude Code fires `UserPromptSubmit` for the positional first turn, and fires no hook while a trust or bypass gate is on screen.
- A positional skill-load message runs as a normal first turn, so `ClassifySkillLoad` finds it in the transcript as it finds a typed one.
- The stacked checklist render came from earlier frames pulled back out of the pane's history, or left on screen across a resize before the next poll; tmux honours `ESC [ 3 J` as clear history.
- The three Publish-time sightings on 2026-10-12 were `PushWithRetry`'s own retries, like the 16:04 one, and cost no row or round.
- The late lock writer finishes within a few seconds of the session end.

## Open risks

- If the stacked render has another cause, D3 may not stop it.
- Clearing history on every redraw discards an operator's copy-mode position in a status pane.
- A run-window launcher that re-runs `lyx reed attach` in a loop would outlast D7's bounded retry; the removal then fails as today.

## Acceptance criteria

- A `start` launch whose events file holds a session start, then a turn start or a tool event, is ready while every capture is blank.
- A `start` launch with a session start, no turn start and blank captures stays pending and dies at the startup deadline.
- A capture showing a trust gate, with no turn start recorded, is dismissed and not reported ready.
- A run with skills launches with the skill-load message on the launch line, types no load message for the first turn, and types the color, the retry turn and the prompt pointer after the load turn ends, in that order.
- A long-running launch-carried load turn delays the color without losing it.
- A load turn that times out skips its skills with a warning, and a pane that dies during it ends the startup died.
- A run without skills launches with the prompt pointer, unchanged.
- Every status pane redraw begins with cursor home, erase screen and erase history, so a shorter, narrower frame drawn over a longer, wider one leaves no line of the old.
- A resize redraws the pane without waiting out the back-off.
- `tmuxkit`'s dead-socket test passes with the shared helper and no wait, and `reedengine`'s socket tests use the same helper.
- `lyx loom fmt` rewrites a misformatted `.go` file to its `go/format` form, leaves a formatted one untouched and unreported, and refuses a whole call naming a non-`.go` path, a path outside the worktree, a symlink resolving outside it or an unparsable file, writing nothing.
- The rendered edit directive names `lyx loom fmt`.
- A task worktree removal whose contained removal first meets a file recreated in the directory succeeds on a retry; a persistent not-empty failure is returned after the last attempt; any other removal failure is not retried.

## Notes for the plan writer

- Startup probe: `internal/shuttleengine/wait.go` (`awaitStartup`, `checkLivenessTick`, `sessionStartRecorded`); launch argument: `claudeengine/claudeengine.go` and `command.go`; skill turns: `run.go` (`start`, `loadSkillsThenPrompt`, `loadSkillTurn`).
- Status pane loop: `internal/statuspane/pane.go`; its users are `loomcli/checklist.go` and `battencli/overview.go`.
- Push retry: `internal/gitrepo/pushretry.go`.
- Stale socket: `internal/reedengine/socketfile_test.go`, `internal/testkit/tmuxkit/tmuxkit_test.go`.
- Neighbouring verb: `internal/loomcli/lintcomments.go`.
- Removal: `removeWarpWorktreeDir` in `internal/fabricengine/remove.go` and its fallback through `removePath` in `destroy.go`.
- `go-driver-assistant` is running and spawns its sessions through shuttle `start`, so it edits the same `shuttleengine` files.
