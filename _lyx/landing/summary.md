# orch: host the real hub — bypass, Agent tool, --adopt, soft cycle

`lyx orch` could not take over the real hub orchestrator.
The hub runs in bypass permission mode and spawns review and fix agents through the Agent tool.
It is already running when orch would take it over, and its context is worth keeping.
orch launched prompt-gated, denied the Agent tool, could only start fresh or from a handoff file, and cycled only at one hard threshold.
This change closes those gaps and runs orch against a live Claude Code install for the first time.

## What changes

- **Permission mode.**
  `shuttleengine.Spec` gains `PermissionMode`, validated by `claudeengine` alone.
  Values are empty (the run mode's default), `bypass` (adds `--dangerously-skip-permissions`) and `prompt`.
  Anything else, and `prompt` on an autonomous run, is an engine error.
  The launch command and the resume command reed replays carry the same mode.
  `orch.yaml` gains `permission_mode`, template default `bypass`, passed through unvalidated.
- **Agent tool.**
  `Spec.AllowAgentTool` removes the Agent `PreToolUse` deny and its notice sentence for that run, whatever `claude_deny_agent_tool` says.
  orch sets it together with `ForkSubagents`.
  The fork-context `lyx webster` guard stays installed.
- **Prompt suggestions off.**
  Every generated settings file carries `promptSuggestionEnabled: false`, in every shuttle-launched session.
  A greyed suggestion read as a draft and could keep the idle probe from ever passing.
- **`lyx orch start --adopt <session-id>`.**
  `Spec.ResumeSessionID` makes the engine launch that session with `--resume`, under this run's settings file and with the new `orch-template-adopt.md` stencil as the first message.
  The watcher binds to the adopted transcript through the turn-end payload, so it needs no adopt-specific code.
  State is reset for a fresh launch with `LastHandoff` kept, and `prompt_source` reads `adopt`.
  `shuttleengine.Runner` checks for an optional `SessionResumer` capability before a resume.
- **Soft threshold.**
  New keys `soft_threshold_tokens` (300000) and `soft_idle_s` (300).
  Between the soft and hard thresholds, the watcher sends the new `orch-template-handoff-soft.md` request once the last turn end is `soft_idle_s` old and the idle probe passes.
  The session may answer `DEFER`, which aborts the cycle to idle with `LastAbortReason` `deferred` and holds the next soft attempt for `soft_idle_s`.
  `lyx orch status` reports the trigger (`soft`, `hard` or `requested`), both thresholds and the last deferral.
- **Idle probe fix.**
  Found by the live run: Claude draws the session name into the input box's top rule, and `isBoxRule` rejected it.
- **Docs.**
  The `orchengine` and `shuttleengine` package docs and `orch.yaml`'s template comments are updated.
  The live smoke outcomes replace "unverified, pending a smoke run" in the orchengine Open risks.

## Design decisions

- The permission decision belongs to the caller, so it is a `Spec` field rather than a function of interactive versus autonomous.
  Under `bypass`, subagents and forks run in the parent's mode, so every agent orch spawns runs every tool unprompted; the operator's lever is `permission_mode: prompt`.
- The Agent allowance has no `orch.yaml` key, because the hub always needs it.
- The soft trigger only adds an earlier, deferrable entry into the existing handoff-first cycle, so it cannot clear a session without a written handoff.
  A written handoff file wins over `DEFER`.
  `DEFER` is ignored on hard and requested cycles, and a reading at or over `threshold_tokens` is always a hard trigger.
  A soft threshold at or above the hard one never fires.
  The session judges the natural break because the real hub nearly always has a watcher in the background, so "nothing in the background" would never hold.
- Adopt refuses with `--handoff`, with a live orch strand, on a malformed id, with no transcript under the prime's project directory, and when Claude's session registry shows a live process holding the id.
  An absent or unreadable registry warns on the log and envelope and proceeds, so two processes could then drive one session.
  That case is bounded by adopt being an explicit verb naming one id and by the transcript check.
  Adopt never kills, removes or renames a process or strand, including the operator's own `claude` strand.
- All Claude specifics stay in `claudeengine`, and the registry and project-directory lookups derive from paths it already uses.

## To check by hand

- The live smoke suite (build-tagged) was run and its four tests passed, but it is not part of CI; rerun it before relying on adopt against the real hub.
- The soft-cycle smoke test logs the background-shell count instead of asserting it, so whether a background shell survives `/clear` is read from the log and the orchengine doc.
- Open finding, not fixed here: a visible plain run shrinks the orch pane to 3 rows, which stalls the idle probe.
- Adopting the real hub session, and the `<shortname>:orch` name, loom-run `parent` and SendMessage reachability after adopt, are covered by the smoke test only.
- This change adds stencils, so a production deploy (`update-plugins.sh`) is needed before the board copies stop reporting drift.
