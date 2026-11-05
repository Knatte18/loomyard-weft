# Add the Codex pane and load trial report

Adds `docs/research/codex-engine-trial-pane-load-2026-11-03.md`, the report of parts 5 and 7 of project `codex-engine-trial`, run against real Codex sessions on `luna[medium]` launched through `lym shuttle`.
No code or config changes.

## What the report finds

- Pane readings, 4,000-character sends, a Codex exit and three sessions at once through ten turns each, with an interrupt and a send during a busy turn, match `codexengine`'s and `creelengine`'s docs.
- A 40-line send is refused by the single-line rule of the guarded send, so it was checked only as a bracketed paste, which lands whole.
- A creel notice and a reminder reach a Codex run's composer as documented, but the session cannot run `lym creel read` or `post --reply` from its sandbox: the socket connect is refused.
- An opened Codex session is never read idle, so creel types nothing into it.
- Case 6, the end-to-end darn run in a sandbox hub, was not run: `lym fabric clone` is refused from a task session. The orch takes it as a separate step.

## Design decisions

- Faults are board notes B3 (`opened-codex-never-idle`), B4 (`codex-state-asking-after-exit`) and B5 (`codex-sandbox-no-creel-socket`), one finding each; the hub's config is unchanged.
- The cases needed sends, idle probes and interrupts that no `lym` verb offers, so a scratch program under `.scratch/` called the same runner methods creel's delivery calls; it is not committed.
- The parent agreed to leave case 6 out of this report and not file the clone refusal as a bug.
