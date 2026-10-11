# Shuttle reads agent state from hooks first, the pane only where no hook exists

`shuttle-hook-first` · Task · enhancement, bug, shuttle, loom · running · Describe

## Seen (2026-10-11, main at a35c8c2b1)

Claude Code state reaches shuttle through three channels: the recording hooks `buildSettings` installs (`internal/shuttleengine/claudeengine#buildSettings`, settings.go:265-320), pane captures read through reed, and process or file state (reed's pane liveness, Claude's `~/.claude/sessions` registry, transcript files).
The installed hooks are `Stop`, `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd` on every run, `PreToolUse(AskUserQuestion)` on interactive runs, and `SessionStart` only for a run whose spec sets a context-after-compaction command (settings.go:290-297, `compact` matcher only).
Each hook writes a stamp line naming the hook from lyx's own command, then the payload (`internal/shuttleengine/claudeengine#Claude.ParseSessionSignals`, signals.go:57-118; `#signalFromPayload`, signals.go:130-172).
`SessionFold` (`internal/shuttleengine#SessionFold.settle`, sessionstate.go:243-266) folds them into busy, idle-done, idle-stalled, asking, dead or unknown, and an unknown reading falls back to the pane.

The pane is read for state in these places:

- `internal/shuttleengine#Run.checkLivenessTick` (wait.go:965-1039) captures the pane and calls `internal/shuttleengine/claudeengine#Claude.Startup` (startup.go:90-99), which reads a gate from needles plus option-line evidence and readiness from `❯` or `shortcuts`; `#Claude.TrustDismissSequence` (startup.go:325-343) walks the caret from the same capture.
- `internal/shuttleengine#requireReadyAgentPane` (run.go:931-958) gates `Runner.Interrupt` and the send path of an engine without `SessionCycler` on `Startup` alone.
- `internal/shuttleengine#busyReading` (send.go:146-176) falls back to `Startup` plus `#Claude.IdleSession` (session.go:61-75) when the reading is unknown.
- `internal/shuttleengine#readBox` (readiness.go:45-58) reads `#Claude.InputBoxText` (session.go:81-94) and `#Claude.PaneTooShort` on every readiness reading.
- `internal/shuttleengine#turnStartHold.holds` (send.go:231-257) and `#Run.startHoldReleased` (send.go:645-661) release an unmatched turn start after the pane reads idle for `turnStartIdleOverride`.
- `internal/shuttleengine#Runner.SessionIdle` (session.go:138-177) reads the pane alone for a strand with no run record, which the orch watcher (`internal/orchengine/watcher.go:481`) reaches through `orchcli`.
- The paste confirmation (`internal/shuttleengine#confirmPasted`, send.go:409-441) reads the box after the paste; only `turnStartedSince` (send.go:587) uses a hook.
- reed's session-name repair reads the pane through the namer's `SessionIdle` (`internal/reedengine/namerepair.go:171`), outside shuttle and with no hook reading.

Why #549 failed: before the start prompt, `run.json`'s `promptOffset` is 0 and the events file holds no signal, because no hook fires before the first typed turn.
So `#readRunSessionState` (sessionread.go:64-93) settles unknown with cause no-signal, and the color send and the skill-load send after `#Run.awaitStartup` (run.go:535-549) fall back to the pane in `busyReading`.
`IdleSession` then read the stray `%` in the box as a draft, and the start was torn down.
A `SessionStart` line would have been the first signal and would have kept those sends off the pane fallback.

A Claude crash does not kill the pane: reed starts the pane as a login shell and sources the launch script into it (`internal/reedengine/spawn.go:74-79`, `panebin.go:63-73`), so the shell outlives `claude`.
The run loop's death check is reed's `Live` (`pane_dead`) in `checkLivenessTick`.
The fold's `dead` state (from `SessionEnd` or the session registry, `internal/shuttleengine/claudeengine#Claude.ProcessLiveness`, prober.go:24-66) only holds sends (readiness.go:77) and is shadow-logged (waitshadow.go:90); `Run.Wait` does not act on it.
So a crashed Claude leaves the run waiting until its deadline.

Hook facts are from the Claude Code hooks reference (code.claude.com/docs/en/hooks), read through a summarizing fetch; only the `SessionStart` and `SessionEnd` schemas were confirmed word for word.
`SessionStart` carries `source` (`startup`, `resume`, `clear`, `compact`, `fork`), and its hooks run in the background while typed input is held until they finish; the docs do not tie it to the input box being drawn.
Settings-file hooks are held back until the workspace trust dialog is accepted, and no hook fires for the trust or bypass dialog.
No hook fires for typing without submitting, nor for an Esc interrupt; `Stop` does not run on an interrupt.
`PermissionRequest` fires when a tool call needs a permission decision, whereas `Notification(permission_prompt)` fires about 6 s later and a keystroke defers it.
`StopFailure` is the only API-error hook.

| State | Read today | Hook that could carry it | Must stay on pane? |
|---|---|---|---|
| Startup: provider up, TUI drawn | Pane: `Claude.Startup` (startup.go:90) in `Run.checkLivenessTick` (wait.go:1009), every 10 s from `Run.awaitStartup` (wait.go:500); `requireReadyAgentPane` (run.go:931) for Interrupt | `SessionStart` (`startup`, `resume`): proves the process loaded its settings and that trust was accepted; not installed today outside the compaction case | Partly: the docs do not say the box is drawn at `SessionStart`, so a pane confirmation stays until the corpus pins the order |
| Startup: trust or bypass gate shown | Pane: gate needles plus option-line evidence (startup.go:90-99), dismissed by `TrustDismissSequence` (startup.go:325) | None; hooks are held until trust is accepted, so a `SessionStart` line proves the trust gate passed | Yes, for detecting and dismissing the gate |
| Idle before the first typed turn (color, skill load) | Pane fallback in `busyReading` (send.go:157-175), since the fold reads unknown/no-signal before any hook fires; the #549 path | `SessionStart(startup)` with no later turn start: idle, never typed into | No for idle; the box read below stays |
| Idle or busy mid-run | Hook fold (`UserPromptSubmit` busy, `Stop` idle) in `sessionReadiness` (readiness.go:95); pane idle releases an unmatched turn start after `turnStartIdleOverride` (send.go:231, 645) | Already hooks; the pane release exists because an interrupted turn writes no `Stop` | Only the interrupt release, see Esc interrupt |
| Turn end, run done | `Stop` line through `ParseEvents` (events.go:22) in `Run.pollEventsTick` (wait.go:637), plus the output files; outstanding work from the payload's `background_tasks`, the transcript when absent | Already `Stop` | No |
| Held at a turn end | `Stop` or ask with output files missing, held by `pollEventsTick`; notice from hold.go | Already hooks | No |
| Waiting on a permission prompt or question | `PreToolUse(AskUserQuestion)` (interactive only, settings.go:307); `Notification` `permission_prompt`, `elicitation_dialog` (signals.go:151-158); the pane shows the dialog as no box (`IdleSession`, readiness.go:85) | `PermissionRequest` fires at once rather than 6 s later and carries `tool_name`; newer `Notification` types (`agent_needs_input`, `elicitation_url_dialog`) are skipped today | Only for a picker or menu the operator opens (`/model`), which no hook reports |
| Operator draft in the input box (pane-inject-safety guard) | Pane: `InputBoxText` via `readBox` (readiness.go:45) and in `confirmPasted` (send.go:409); operator activity through reed's tmux `client_activity` (send.go:180) | None; no hook fires on typing | Yes |
| Pane too short, no box on screen | Pane: `PaneTooShort`, `readBox` (readiness.go:83-86) | None | Yes |
| Send landed | Box read after the paste; a turn start past the pre-send offset (`turnStartedSince`, send.go:587) | `UserPromptSubmit` already for text; `SessionStart(clear)` and `SessionStart(compact)` or `PostCompact` could confirm `/clear` and `/compact`; nothing for `/color` or `/reload-plugins` | For slash commands that start no turn and have no hook |
| Esc interrupt | Transcript `[Request interrupted by user` marker (`Claude.TurnStartInterrupt`, prober.go:69-95); else the pane idle release | None; `Stop` does not run on an interrupt | The pane release stays as the backstop |
| Crash or death | reed `Live` (`pane_dead`) in `checkLivenessTick` (wait.go:978) and `requireLiveStrand` (run.go:968); fold `dead` from `SessionEnd` or the registry (prober.go:24), used only to hold sends and in shadow logs | `SessionEnd` for a graceful exit; a crash fires none, so the registry pid stays the source | Not the pane: the pane shell outlives Claude, so pane liveness misses it |
| API error | `StopFailure` (signals.go:139) and the transcript's `isApiErrorMessage` (`Claude.TurnEndActivity`, activity.go:22); batten through `ReadAgentActivity` (activity.go:55) | Already `StopFailure` | No |
| Stalled or quiet (batten notices) | Newer of the transcript's last write and the events file's newest `Stop` or ask time (activity.go:90-155), wait markers | `PostToolUse` could stamp a heartbeat in place of the transcript mtime | No |
| reed name-repair idle | Pane through the namer's `SessionIdle` (`internal/reedengine/namerepair.go:171`) | None in reed, which is provider-blind | Yes, unless shuttle answers for reed |

Task `shuttle-startup-ready` is changing these rows now: its card 1 makes `IdleSession` and `InputBoxText` read a lone `%` as an empty box (the idle-before-first-turn and draft rows), card 3 adds the pane's last lines to a never-ready teardown, and card 4 reworks `Run.awaitStartup`, `Run.checkLivenessTick` and `playInputs` to probe from one poll interval with backoff, keeping a dismissal gap for gate keys (the two startup rows).
It changes no hook and no fold rule, so everything below is additive to it.

## Idea

Hook-first: the hook state answers whenever it settles anything, and the pane is read only for what no hook carries, which is the startup gates, the input box and anything the operator opened.
The pane stays as a guard on top of the hook reading, never as the reading itself, except for an engine without hooks.

What moves, in order:

1. Verify the payloads first.
   The `shuttle-hook-payload-corpus` note is the prerequisite: run its capture scenario, extended to `SessionStart` (`startup`, `resume`, `clear`, `compact`) and `PermissionRequest`, and record when each line lands relative to the trust gate, the bypass dialog and the first `❯` the pane draws.
   A rule keyed on the hook name alone (a `Stop` line, a `UserPromptSubmit` line, any `SessionStart` line) is already reliable, since the stamp line carries the name lyx's own command wrote.
   A rule keyed on a payload field (`source`, `reason`, `notification_type`, `background_tasks`, `error`) is only as good as an unverified shape, so it waits for the corpus.
   The one `StopFailure` fixture in the corpus (`testdata/corpus/fork-killed-api-error`) is reconstructed, not captured.
2. Install `SessionStart` for every run, recording only, with the compaction command kept as the `compact` entry's second command.
   Teach the fold that a `SessionStart` with no later turn start reads idle with a cause of its own, so the color and skill-load sends read the hook state instead of the pane fallback.
   This removes the #549 path at its source; `readBox`'s draft check still runs, as it must.
3. Let `awaitStartup` end on a `SessionStart` line: before it, the pane probe keeps detecting and dismissing gates; once it lands, the trust gate is proven passed and no gate key is ever sent again.
   Whether the bypass dialog precedes it, and whether the box is drawn by then, is what step 1 must settle; until then readiness also needs the pane's `StartupReady`.
4. Map `PermissionRequest` to an ask, and the newer `Notification` types to ask or skip deliberately, so a permission wait reads asking without the 6 s delay.
5. Confirm `/clear` and `/compact` by `SessionStart(clear)` and `SessionStart(compact)` past the pre-send offset, as `turnStartedSince` confirms text, with the box read kept as the fallback.
6. Let `Run.Wait` act on a fold `dead` reading (a process-ending `SessionEnd`, or a registry pid proven gone), after the output-file check that PATTERN-completion-signal requires, since the pane shell outlives Claude.
   Run it in the shadow log first (waitshadow.go already pairs the loop's classification with the fold), then switch.

What stays on the pane: gate detection and dismissal before `SessionStart`, the input-box draft and too-short reads, a menu or picker the operator opened, the operator-activity query, the idle release of an interrupted turn the transcript does not mark, and reed's name-repair idle check.
The pane fallback for an unknown reading stays for an engine without `SessionSignalParser`.

## Open

- Does `SessionStart(startup)` land before or after the bypass-permissions dialog, and before or after the input box is drawn? The docs say input typed while `SessionStart` hooks run is held, not when a paste is safe.
- Does the hold of settings-file hooks until trust apply to hooks from the `--settings` file lyx passes, or only to project settings?
- Should `Run.Wait` end a run as died on the fold's `dead` reading, and on which evidence: a process-ending `SessionEnd` alone, or the registry too, which reads unproven after a normal exit?
- Which of the newer `Notification` types (`agent_needs_input`, `elicitation_url_dialog`, `elicitation_response`) mean asking for a shuttle run?
- Can `idle_prompt`, which fires after about 60 s of quiet with an untouched box, serve as positive proof of an empty box, or only as the contradiction marker it is now?
- Should reed's name repair take shuttle's readiness reading, or stay pane-only because reed is provider-blind?
- Does a resumed session (`Spec.ResumeSessionID`, Attach) write `SessionStart(resume)` with the same `session_id`, so the fold's liveness lookup keeps working?

## Prerequisite folded in: real hook payload fixtures (was note shuttle-hook-payload-corpus)

`shuttle-state` added recording hooks for `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd`, with an `llm`-tier capture scenario that was compiled but never run.
The payload shapes those hooks parse are unverified against a real session (an Open risk in that task).
Run the scenario once and commit the captured payloads as fixtures, so the parsers are tested against real shapes.

Do this first in the task: rules keyed on payload fields wait until the captured fixtures confirm the shapes; rules keyed only on hook names do not.

## Folded in: a stray character in an idle input box blocks resume (was note stray-draft-blocks-resume, issue #554)

Seen 2026-10-11 in `small-fixes-oct11`: after a Discussion-review escalation was accepted, `lyx loom resume` refused to wake the parked driver, retries included.
The driver's pane showed an idle Claude Code UI whose input box held one stray character, `❯ +`, typed by nobody; resume read it as an operator draft, and the run stayed `awaiting` until the operator attached and deleted it.
Same class as #549's stray `%`.

The operator-draft check stays on the pane after this task, so it needs its own rule:
- a lone stray non-alphanumeric character in an otherwise idle input box does not block a resume or a start for good: treat it as no draft, or clear it after a bounded wait with no operator activity;
- every refusal for an operator draft quotes the text it found, so the orch can act without opening the pane.

## Folded in: bound Merriam's fork transcript read (was note fork-transcript-tail-read)

Seen in the `fork-wait-noise` PR #555 review: `internal/shuttleengine/claudeengine` `holdsPendingToolCall` parses each fork transcript in full at every turn-end activity read of Merriam, so a Master with many batches or large fork transcripts pays that parse on every read.
Read each fork transcript backward from its tail and stop at the first tool result or pending call, so the cost is bounded by the tail, not the transcript length.
