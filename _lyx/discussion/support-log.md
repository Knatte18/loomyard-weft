# Support log — hub-orch-strand

## Interview

The session ran autonomously with no operator present; every turn below is a question put to the codebase and a self-made pick.

1. **Scope source.**
   The board task names the three open points: threshold, adopting the operator's terminal session, and cross-session messaging.
   Its proposal asks for a `lyx hub orch` whose name is open, and describes the cycle as usage read, then idle check, then `/scribe:handoff`, then `/clear`, then resume.
2. **Where it sits.**
   The codebase shows every agent launches through proc → reed → shuttle, the Claude engine already writes a Stop hook into an events file, and shuttle already exposes `Send`/`Inject` and a `ModelSwitchSequence` slash-command precedent.
   The session therefore becomes a shuttle run (D3), with provider grammar kept in claudeengine (D7, D13).
3. **Watcher host.**
   Candidates were reed's per-hub watchdog, a Claude Stop hook, and a detached lyx process.
   Picked the detached lyx process (D4).
4. **Idle.**
   `manifest/HANDOFF.md` shows the orchestrator keeps one background watcher per live run, so a Stop-only idle rule would starve.
   Picked turn end including `EventWaiting`, plus a grace period and an input-box probe (D9).
5. **Threshold.**
   The transcript does not state the window size.
   Picked an absolute, configurable 150000 (D8).
6. **Handoff location.**
   The prime is on `main`, `update-plugins.sh` needs a clean tree, and `manifest/HANDOFF.md` is operator-updated only.
   Picked `.lyx/orch/handoffs/` (D12).
7. **Adoption.**
   The operator's VS Code terminal session is not in tmux, so lyx cannot drive it.
   Picked no adoption, with a manual one-time migration (auto-mode assumptions).
8. **Messaging.**
   Claude Code's native `ListAgents`/`SendMessage` already reaches any local session.
   Picked no lyx work (D16), recording the `/clear` gap and address stability as risks.
9. **Failure safety.**
   Settled on a persisted phase machine (D10) and the rule that nothing clears without a written handoff (D11).

## Rejected alternatives

- **`lyx hub orch` as the CLI path** — "hub" already names geometry (`hubgeom`, `hubforge`, the suffix); a `hub` parent holding one child adds a level for nothing.
- **A weaving-themed module name** — no weaving term maps onto "orchestrator session", and the operator already calls it "orch".
- **Hosting the watcher in reed's per-hub watchdog** — reed sits below shuttle and must stay provider-blind; reading Claude transcripts there breaks the Shuttle Provider-Seam Invariant.
- **Driving the cycle from a Claude Stop hook inside the session** — a hook runs synchronously inside the turn and cannot `/clear` its own session; it would need a detached helper anyway, and it spreads orch logic into provider settings.
- **Threshold as a percentage of the window** — the window size is not in the transcript; guessing it per model string is brittle.
- **Stop-only idle (excluding `EventWaiting`)** — the orchestrator's long-lived background watchers would block cycling for hours.
- **Writing cycle handoffs to `manifest/HANDOFF.md`** — it is tracked, which dirties the prime and blocks `update-plugins.sh`, and the operator updates it only on request.
- **Adopting the operator's existing session via `claude --resume` into a strand** — it moves a context that is already large and needs the current session ID; the one-time manual migration costs a single command.
- **Auto-relaunch or provider-resume after process death** — the session ID changes on every `/clear`, so tracking the current one adds state for a rare event; the relaunch from the last completed handoff loses at most one cycle's worth.
- **A Shed recipe for the orchestrator** — it is not a phased run with producers; a plain state machine is smaller.
- **Pruning old handoffs** — a few small files a day; retention logic is unmotivated.
- **Adding a hub-workspace auto-attach task in `lyx ide`** — belongs with the existing `ide-hub-workspace-followups` note, not this task.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Module name?** Resolved (auto-pick): `lyx orch`, D1.
- **Q2 — Which worktree hosts it?** Resolved (auto-pick): the warp prime only, D2.
- **Q3 — What launches the session?** Resolved (auto-pick): a shuttle interactive run, D3.
- **Q4 — Where does the watcher run?** Resolved (auto-pick): a detached `lyx orch watch`, D4.
- **Q5 — Threshold?** Resolved (auto-pick): absolute tokens, default 150000, configurable, D8.
- **Q6 — What counts as idle?** Resolved (auto-pick): Stop or Waiting, plus the grace period and the input-box probe, D9.
- **Q7 — Can the operator's own terminal session be adopted?** Resolved (auto-pick): no; manual migration, as in auto-mode assumptions.
- **Q8 — Cross-session messaging to the strand-hosted session?** Resolved (auto-pick): native `SendMessage`, no lyx work, D16. Two sub-points stay open risks: the `/clear` gap, and address stability across `/clear`.
- **Q9 — Where do cycle handoffs go?** Resolved (auto-pick): `.lyx/orch/handoffs/`, kept, D12.
- **Q10 — What if the handoff never appears?** Resolved (auto-pick): timeout aborts with no `/clear`, D11.
- **Q11 — Start prompt when no handoff is given?** Resolved (auto-pick): last completed cycle handoff, else the plain start stencil; `manifest/HANDOFF.md` only via `--handoff`, D6.
- **Q12 — Manual trigger?** Resolved (auto-pick): `lyx orch cycle`, D14.
- **Q13 — Do background tasks survive `/clear`?** Open: unverified, listed under open risks; the handoff focus text mitigates it, and the smoke test verifies it and records the outcome.
- **Q14 — What happens to a mid-cycle phase when the strand is stopped or dies?** Resolved (review round 1): a fresh launch resets it to `idle` and records it; only a watcher restarted against the same strand resumes it, D10.
- **Q15 — Which handoff does a fresh launch resume from?** Resolved (review round 1): the last completed handoff recorded in state, never merely the newest file, D6.
- **Q16 — How does a watcher know a turn is running when no event marks a turn start?** Resolved (review round 1): the idle-session probe classifies a turn in progress, D9.
