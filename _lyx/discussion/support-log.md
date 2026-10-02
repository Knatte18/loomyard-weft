# orch-live-ready — support log

## Interview

The session ran autonomously, so each turn below is a question the writer put to itself after reading the code, with the pick it made.

1. **Is this the right thing to build?**
   The board entry lists four gaps plus one live risk, and each one blocks the real hub from moving under orch: without bypass the hub stalls at dialogs, without the Agent tool it cannot review or fix, without adopt it loses its running context, and without a passing idle probe nothing cycles.
   Pick: build all five; the idle-probe risk is treated as a probable defect to fix, not only a thing to observe.

2. **Where does the permission decision live?**
   `claudeengine` adds `--dangerously-skip-permissions` exactly when a run is autonomous, on both the launch and resume lines.
   Pick: a `Spec.PermissionMode` string in engine vocabulary (the `Effort` contract), validated by the engine, with orch's value coming from `orch.yaml`.
   `prompt` on an autonomous run is rejected because such a run stalls with no one to answer.

3. **How does orch get the Agent tool?**
   The Agent deny comes from shuttle's global config key, and `ForkSubagents` narrows it to forks only.
   Pick: a per-run `Spec.AllowAgentTool` that skips the deny and its notice sentence, set by orch alongside `ForkSubagents` so the fork env flag stays on.
   No `orch.yaml` key, since the hub always needs the tool.

4. **Is the idle-probe risk real?**
   `IdleSession` reads a capture with no styling and fails closed on any text inside the box, and its own doc names the greyed prompt suggestion as a case it misreads.
   The installed Claude Code (2.1.287) exposes a `promptSuggestionEnabled` settings key ("When false, prompt suggestions are disabled") and a `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` env var.
   Pick: write `promptSuggestionEnabled: false` into every shuttle run's settings file, keep the probe fail-closed, and confirm live in the smoke run.

5. **What does adopting a running session mean, given two processes cannot share a session?**
   Claude Code keeps a per-process registry under `~/.claude/sessions/` naming `pid`, `sessionId` and `cwd`, and the operator's hub session in the prime runs in a reed strand named `claude` from the folderOpen chain.
   Pick: adopt resumes an existing session id in a new shuttle-managed strand; it refuses while a live process holds the id, and the operator exits the original first.
   Unreadable registry warns and proceeds, so only a positively identified holder blocks.
   The watcher needs no change: the Stop payload names the transcript, so the context reading follows the adopted session.

6. **What is a natural break for the soft threshold?**
   The watcher already sees whether the newest turn end had background work outstanding (`EventWaiting`) and when it arrived.
   Pick: at or over the soft threshold, cycle when the newest event is a plain `EventStop`, it is at least `soft_idle_s` old (300 s), and the idle probe passes.
   The hard cap keeps today's rule, including cycling on `EventWaiting`, so a hub that always runs a Monitor still cycles at 400k.

7. **How is "verify live" closed?**
   The smoke test exists but has never run against a live install, and the orchengine doc carries two "unverified" open risks.
   Pick: extend the smoke test to the new paths, run it on this machine, and record outcomes in the package doc.

8. **Edge cases checked.**
   `--adopt` with `--handoff`, `--adopt` with a live strand, a malformed id, a missing transcript (session ran from another cwd), a held id, a dead pid in the registry, an absent registry, soft at or above hard, non-positive config values, and a reed rebirth replaying the adopted id.

9. **Security.**
   Bypass mode removes the permission guard for the orch session and its subagents; it is bounded to orch's own run and switchable back through `orch.yaml`.
   The adopt id is validated for shape before it reaches a command line.

## Rejected alternatives

- **Claude's own `--permission-mode` vocabulary in `Spec`** (`bypassPermissions`, `acceptEdits`, `plan`).
  Rejected: it leaks provider vocabulary into the knob's values and offers modes nobody asked for.
- **A `--permission-mode` or `--allow-agent-tool` flag on `lyx shuttle run`.**
  Rejected: no caller needs it (YAGNI).
- **Flipping shuttle's global `claude_deny_agent_tool` for the hub's worktree.**
  Rejected: it would open the Agent tool to every run in the prime, not only orch.
- **Reading styled captures (`capture-pane -e`) and treating dim text in the box as empty.**
  Rejected: it needs a new reed capture mode and encodes a guess about Claude's styling, while a documented settings key removes the suggestion outright.
- **Disabling prompt suggestions only in orch's runs.**
  Rejected: the idle probe is shared with the session-name watchdog in every run.
- **Adopt waiting for the holding process to exit, or killing it.**
  Rejected: waiting needs a detached helper for a session that cannot exit itself, and killing a live session can destroy an operator draft.
- **Adopt fails closed when the registry cannot be read.**
  Rejected: the registry is a Claude internal; refusing on its absence would make adopt break on a shape change even when no other process holds the id.
- **Soft threshold as an injected notice asking the session to run `lyx orch cycle` at its own break.**
  Rejected: it spends a turn, relies on the session complying, and the session cannot see its own token count to judge.
- **Excluding `EventWaiting` from the hard trigger too.**
  Rejected: the hub keeps long-running Monitors, so it would never cycle.
- **Renaming `threshold_tokens` to `hard_threshold_tokens`.**
  Rejected: existing `orch.yaml` files would fail to carry their value over, for a cosmetic gain.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Build all five board items? — resolved, auto-pick: yes.
- Q2 Permission mode as a `Spec` field in engine vocabulary? — resolved, auto-pick: yes, values empty/`bypass`/`prompt`.
- Q3 Orch default permission mode? — resolved, auto-pick: `bypass` (board-stated).
- Q4 Agent tool per run or via config? — resolved, auto-pick: per-run `AllowAgentTool`, orch only, no `orch.yaml` key.
- Q5 Fix or only observe the idle-probe risk? — resolved, auto-pick: fix via `promptSuggestionEnabled: false` in every run, verify live.
- Q6 Adopt semantics with a live holder? — resolved, auto-pick: refuse with way forward; never wait or kill.
- Q7 Adopt with unreadable registry? — resolved, auto-pick: warn and proceed.
- Q8 How does the operator find the session id? — resolved, auto-pick: explicit id from Claude's `/status`; no discovery.
- Q9 Soft-threshold natural break? — resolved, auto-pick: `EventStop` quiet for `soft_idle_s` (300 s) at or over 300k.
- Q10 Keep `threshold_tokens` as the hard-cap key? — resolved, auto-pick: yes.
- Q11 Record the cycle trigger in state and status? — resolved, auto-pick: yes, so a live run shows which path fired.
- Q12 Is running the smoke test in scope? — resolved, auto-pick: yes, outcomes recorded in the orchengine package doc.
- Q13 Does `claude --resume <id>` accept a positional prompt? — open: verified by the smoke run; fallback recorded in the decision record's Open risks.
- Environment: nothing was broken; `lyx board get` returned the task.
