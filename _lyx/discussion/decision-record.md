# orch-live-ready — decision record

## Goal

Make `lyx orch` able to host the real hub orchestrator, and prove it against a live Claude Code install.
The hub today runs in bypass permission mode, spawns review and fix agents through the Agent tool, and is already running when orch would take it over;
orch launches prompt-gated, denies the Agent tool, can only start fresh or from a handoff file, and cycles only at one hard threshold.
The idle probe may also never pass on a real session, which would mean no cycle ever runs.

## Scope

In:

- A per-run permission mode in shuttle's `Spec`, realized by `claudeengine`, with orch defaulting to bypass through `orch.yaml`.
- A per-run Agent-tool allowance in shuttle's `Spec`, set by orch together with `ForkSubagents`.
- Prompt suggestions switched off in every shuttle-launched Claude session, so the idle probe can pass.
- `lyx orch start --adopt <session-id>`: resume an existing Claude session as the orch strand, with the watcher bound to it.
- A soft threshold that cycles at a natural break, with the existing threshold kept as the hard cap.
- Extending and running the orch smoke test against a live Claude install, and recording what it showed.
- The docs those changes touch: the `internal/orchengine` package doc (orch's module doc), the `internal/shuttleengine` package doc for the new `Spec` fields, and `orch.yaml`'s template comments.

Out:

- Any new flag on `lyx shuttle run`; the new `Spec` fields have orch as their only production caller.
- Exposing the Agent-tool allowance as an `orch.yaml` key.
- Killing, removing or renaming any strand or process orch did not launch, including the operator's own `claude` strand.
- Changing the hard-cap cycle rules, the four-phase cycle, or the `--handoff` migration path.

## Decisions

1. **Permission mode is a `Spec` field in engine vocabulary.**
   `Spec.PermissionMode` is a string the engine alone validates, the same contract `Effort` has.
   `claudeengine` accepts three values: empty (the run mode's default — autonomous runs bypass, interactive runs prompt), `bypass` (adds `--dangerously-skip-permissions`), and `prompt` (adds nothing).
   Any other value, and `prompt` on an autonomous run, is a hard error from the engine, since an autonomous run that prompts stalls with nobody to answer.
   The launch command and the resume command reed replays carry the same mode.
   Bound: this removes the permission-prompt guard only for a run whose caller sets `bypass`;
   orch is the only production caller that does, and the operator turns it back on with `permission_mode: prompt`.
   Subagents and forks run in their parent's permission mode, so under `bypass` every agent the orch session spawns also runs every tool with no prompt.
   This applies to an adopted session too (Decision 5), which comes back under orch's mode whatever mode it ran in before.
   Rationale: the permission decision belongs to the caller, not to the interactive/autonomous split it is coupled to today.

2. **`orch.yaml` gains `permission_mode`, template default `bypass`.**
   orch passes the value to the `Spec` verbatim and does not validate it itself, so the engine stays the single validator.
   Rationale: the hub runs in bypass mode, and the board asks for an option with bypass as the default.

3. **The Agent tool is allowed per run through `Spec.AllowAgentTool`.**
   When true, the engine installs no Agent `PreToolUse` deny and adds no Agent sentence to the deny notice, whatever shuttle's `claude_deny_agent_tool` says.
   orch's spec sets `AllowAgentTool` and `ForkSubagents` both true, so the hub can spawn typed subagents and forks.
   Bound: the allowance is per run and only orch sets it;
   every other run keeps shuttle's config-driven deny, and the fork-context `lyx webster` guard that comes with `ForkSubagents` stays installed.
   The allowance has no operator switch: no `orch.yaml` key turns the Agent tool off for orch.
   The operator's remaining lever is `permission_mode: prompt`, under which the orch session's subagents and forks prompt in the orch pane.
   Rationale: the hub's review and fix work runs through the Agent tool, and `ForkSubagents` alone admits forks only.

4. **Prompt suggestions are off in every shuttle-launched Claude session.**
   `claudeengine` writes `promptSuggestionEnabled: false` into each run's settings file.
   `IdleSession` keeps its fail-closed reading of a non-empty box.
   Rationale: a capture carries no styling, so a greyed suggestion reads as a draft and the probe never passes;
   switching the suggestion off at the source fixes every interactive run and the session-name watchdog that shares the probe.

5. **`lyx orch start --adopt <session-id>` resumes an existing session as the orch strand.**
   `Spec.ResumeSessionID`, when set, makes the engine launch that session with `--resume` instead of minting a new id, with this run's own settings file (so the Stop hook feeds the watcher) and the launch prompt as the first message.
   reed's resume command names the same id.
   The watcher binds to the adopted transcript through the turn-end payload's transcript path, so it needs no adopt-specific code.
   The launch prompt is a new orch stencil, `orch-template-adopt.md`, telling the session it is now hosted by `lyx orch` and how handoff requests reach it;
   the envelope's `prompt_source` reads `adopt`, and state is reset for a fresh launch with `LastHandoff` kept.
   Refusals, each naming its way forward:
   - `--adopt` together with `--handoff`.
   - `--adopt` while the orch strand is live (`lyx orch stop` first), the same rule `--handoff` has.
   - From the engine: a malformed session id; no transcript for that id under the prime's Claude project directory;
     and a live Claude process holding the id, read from Claude's own session registry with liveness through `internal/proc` (exit that session first).
   A registry that is absent or unreadable logs a warning, carries it on the envelope, and proceeds, since only a positively identified live holder refuses.
   Bound: adopt resumes only a session whose cwd was the prime, never takes over a session a live process is positively identified as holding, and never kills, removes or renames any process or strand.
   An absent or unreadable registry lets an unconfirmed holder through, and two processes then drive one session.
   That case is bounded by adopt being an explicit operator verb naming one id, the warning on the log and envelope, and the transcript check still confining adopt to a prime session.
   Rationale: the hub is already running with context worth keeping, and a handoff file loses more than a resume does.

6. **A soft threshold cycles at a natural break; `threshold_tokens` stays the hard cap.**
   New `orch.yaml` keys: `soft_threshold_tokens` (template 300000) and `soft_idle_s` (template 300), each floored to its template default when non-positive, like the existing knobs.
   In phase idle the watcher cycles on the soft trigger when all hold:
   the reading is at or over the soft threshold and below `threshold_tokens`, the newest event is a turn end (`EventStop` or `EventWaiting`), that event was read at least `soft_idle_s` ago, no soft deferral is pending (below), and the idle probe passes.
   The soft trigger sends its own handoff request, a new stencil `orch-template-handoff-soft.md`, which lets the session decide whether this is a natural break:
   long-lived watchers (background shells, Monitors) are not in-flight work and go into the handoff to be re-armed;
   work a `/clear` would lose (a subagent or fork mid-task, an edit half done) is, and then the session replies with the single word `DEFER` and writes no handoff file.
   The watcher reads the reply from the turn end's final assistant message: a turn end carries `DEFER` when that message, trimmed of whitespace, equals `DEFER` exactly.
   In a soft cycle:
   - A `DEFER` turn end with no handoff file written aborts the cycle back to idle with `LastAbortReason` `deferred`.
     The deferral is pending until `soft_idle_s` after that turn end was read; only then may the soft trigger fire again.
   - A written handoff file wins over `DEFER`: the cycle proceeds as today, since the handoff it needs exists.
   - A turn end with neither keeps today's behaviour: the watcher waits for the file until `handoff_timeout_s`, then aborts with "handoff timed out".
   The hard trigger (reading at or over `threshold_tokens`, the existing idle grace, `EventStop` or `EventWaiting`, the existing handoff stencil) and the `lyx orch cycle` request are unchanged and cannot be deferred:
   in a hard or requested cycle the watcher ignores `DEFER` and waits for the file until the timeout as today.
   A reading at or over `threshold_tokens` is always a hard trigger, whatever `idle_grace_s` and `soft_idle_s` are set to, so a cycle started at a hard-cap reading is never soft;
   a soft threshold at or above the hard one therefore never fires.
   State records which trigger started the current or last cycle (`soft`, `hard` or `requested`), and `lyx orch status` reports it alongside both thresholds and the last deferral.
   Bound: the soft trigger only adds an earlier, deferrable entry into the existing handoff-first cycle, so it can never clear a session without a written handoff.
   Rationale: the real hub keeps a watcher in the background almost all the time, so "nothing in the background" would never hold;
   only the session knows whether what runs in the background survives a handoff, so it judges the break and the hard cap stays the backstop.

7. **Live verification is part of this task.**
   The build-tagged orch smoke test is extended to cover bypass, the Agent tool, a threshold-triggered cycle through the soft path, an adopted session's cycle, and the adopted strand's name, parentage and SendMessage reachability, and is run against this machine's Claude install.
   The naming needs no new code: an adopt launch goes through orch's existing spec and launch lines, which already carry the agent name, so the smoke run only verifies it.
   Its outcomes, including the two observations it already logs (a background task across `/clear`, a `SendMessage` address across `/clear`), replace "unverified, pending a smoke run" in the orchengine package doc's Open risks.
   Rationale: the board entry's risk is settled only by a live session, and orch has never run one.

## Constraints

- Shuttle Provider-Seam Invariant: every Claude specific — the `--dangerously-skip-permissions` flag, `--resume`, the settings key, the session registry, the project-directory layout — lives in `internal/shuttleengine/claudeengine` only.
- Told-Geometry Invariant: `orchengine` and `shuttleengine` stay told; the registry and project-directory lookups resolve from paths `claudeengine` already derives for its transcript audit, not from `lyxcwd`.
- Config Strictness Invariant: `orchengine` stays on `LoadOrTemplate`; the new keys enter through the embedded template.
- Live-Substrate Spawn Observability: the adopt launch logs its spawn like the existing launch.
- CLI/Cobra Invariant: the new flag carries help text, and orch's help-tree test covers it.
- The cycle's existing guarantee holds: nothing is typed into the pane unless the idle probe passed on the same tick, and no session is cleared without a written handoff.
- Documentation lands in the same commit as the behaviour (CLAUDE.md, Documentation Lifecycle); no new cross-cutting invariant is introduced.

## Auto-mode assumptions

- No operator answered; every pick below and in the support log's Question ledger is the writer's own, except Decision 6's natural-break rule, which the hub orchestrator's review revised.
- The permission vocabulary is `bypass`/`prompt`, so the knob reads the same for a later non-Claude engine.
- The Agent-tool allowance has no `orch.yaml` key: the hub always needs it.
- Adopt takes an explicit session id; the operator reads it from Claude's `/status`.
- The soft trigger's natural break is "quiet for `soft_idle_s`" plus the session's own judgment through `DEFER`, with 300 seconds as the template value.
- Disabling prompt suggestions applies to every shuttle run, not only orch, because the idle probe is shared.

## Open risks

- `claude --resume <id>` with a positional prompt is expected to resume and submit the prompt; if the live smoke shows otherwise, the plan falls back to resuming without a prompt and sending the adopt stencil (one line) once startup clears.
- Resuming a large session may raise a startup dialog shuttle's startup classifier does not know; startup then fails closed with its capture kept, and the smoke run is where it shows.
- The session registry, `promptSuggestionEnabled`, the transcript layout and the Stop payload are Claude Code internals; each degrades (registry: warn and proceed; setting: the idle probe fails closed as today) rather than breaking a launch.
- A session that answers `DEFER` on every soft request cycles at the hard cap only; the last deferral in `status` shows it.
- A watcher re-armed from the handoff after `/clear` restarts from the current state, so a change it would have reported during the cycle is reported on its first poll instead.
- Background tasks and subagents launched before a cycle may not survive `/clear`; the smoke run records the observed behaviour, and the handoff stencil's re-arm instruction stays the mitigation.

## Acceptance criteria

- `claudeengine` unit tests: an interactive run with `bypass` gets `--dangerously-skip-permissions` on both the launch and resume commands; empty mode keeps today's behaviour in both run modes; `prompt` on an autonomous run and an unknown value are engine errors.
- `claudeengine` unit tests: `AllowAgentTool` removes the Agent deny hook and its notice sentence with `claude_deny_agent_tool` on, with and without `ForkSubagents`, and leaves the fork `lyx webster` guard in place when `ForkSubagents` is set.
- Every generated settings file carries `promptSuggestionEnabled: false`.
- `ResumeSessionID` produces a `--resume` launch command carrying the prompt, settings, model, effort and permission mode, and a resume command naming the same id.
- The adopt preflight refuses a malformed id, a missing transcript, and an id held by a live process in a fixture registry; a dead pid in the registry proceeds, and an absent registry proceeds with its warning on the envelope.
- `lyx orch start` unit tests: `--adopt` with `--handoff` refuses; `--adopt` with a live orch strand refuses naming `lyx orch stop`; `--adopt` with a dead or absent strand launches with `prompt_source` `adopt` and a `Spec` carrying the id, `bypass`, `AllowAgentTool` and `ForkSubagents`.
- Watcher unit tests with the fake session: the soft trigger fires on `EventStop` and on `EventWaiting` past `soft_idle_s` and not before; a `DEFER` reply with no handoff file aborts to idle with `deferred` and holds the next soft attempt until `soft_idle_s` after it was read; a reply that only contains `DEFER` among other text does not defer; `DEFER` with a written handoff file proceeds to clearing; `DEFER` on a hard or requested cycle is ignored and the cycle waits for the file until the timeout; the hard trigger still fires on `EventWaiting` after the idle grace; a reading at or over `threshold_tokens` starts a hard cycle even when `idle_grace_s` exceeds `soft_idle_s`; a soft threshold at or above the hard one never fires; the recorded trigger is `soft`, `hard` or `requested` accordingly and `status` reports it.
- Config tests: the new keys load from the template, and non-positive values floor to their defaults.
- The extended smoke test passes against this machine's live Claude install: a Bash tool call runs with no permission dialog, a subagent spawns through the Agent tool, the idle probe passes after a turn, a soft-threshold cycle completes (handoff written, `/clear`, resume prompt) with a background shell running and re-armed after it, and an adopted session resumes with its prior context and completes a cycle.
- Live smoke, verifying behaviour shipped with agent-naming: after adopt, the orch strand carries the hub's full name `<shortname>:orch`, a loom run started from it records that name as `parent`, and a driver's escalation notice reaches it through SendMessage.
- The orchengine package doc describes `--adopt`, the permission mode, the Agent-tool allowance, the soft trigger, and the recorded smoke outcomes; the prompt-suggestion residual is replaced by the settings fix.

## Notes for the plan writer

- `claudeengine` already locates a session's project directory and transcript for its fork audit; the adopt preflight can reuse that lookup.
- Claude's session registry is `~/.claude/sessions/<pid>.json`, each entry naming `pid`, `sessionId` and `cwd`; the registry directory should be told to the check so tests run on a fixture.
- `internal/proc.IsAlive` exists on both platforms.
- The operator's own hub session in the prime runs in a reed strand named `claude` from the folderOpen chain; after adopt that strand stays the operator's to remove.
