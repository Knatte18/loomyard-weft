# Test creel stage 1 in a closed sandbox hub

`creel-test` · Task · enhancement, creel, reed, shuttle

## Goal

Test creel stage 1 (`internal/creelengine`, `internal/creelcli`, `lyx creel`, landed in `993ff3fc5`) thoroughly, in a closed environment, before anything that works today is switched over to it (`creel-notices`, `creel-agents`).
Fix the defects the testing finds, on this task's branch.

Operator decision 2026-10-15: creel is tested in its own task, in its own worktree, in a closed environment.
Operator decision 2026-10-15 (earlier): build a working creel first, in parallel, tested thoroughly before anything that works today is switched over.

## Closed environment

- Every live check runs in a sandbox hub (the `tools/sandbox` model, `lyx fabric clone` into a scratch location), never in the live hub.
  A separate hub path gives a separate creel daemon, since the socket name hashes the hub path.
- Its tmux sessions run on their own tmux server, never the live hub's server or the prime's session `loomyard`; no window opens on the operator's desktop.
- Nothing the run starts posts, injects or reads in the live hub's creel, and no live agent receives a line from it.
- The sandbox hub, its daemon and its tmux server are torn down at the end.

## What to test

- The two end-to-end tests the landing left unrun, through `lyx gate test` with their tags: `TestSmokeCreelFakeProviderPane` (tmux tier) and `TestSmokeCreelClaudeStrand` (llm tier).
- The landing's "to check by hand" list:
  - `read` sets and `inbox` in `Store.Read`: plain `read` (oldest unread plus every other unread from its sender), `read --from`, `read --all`, and the `inbox` sequence;
  - a full `lyx gate test ./...` and `--tags integration` run green.
- Every verb against a live strand added by `lyx reed add --cmd claude`: `post` (`--reply`, `--wait-reply`, `--timeout`), `inject` (`--interrupt`, `--ttl`, `--wait`), `interrupt`, `forward`, `read`, `inbox`, `log`, `hold`, `release`.
- Delivery rules:
  - a reminder is typed only at idle, never into a busy turn, never twice;
  - the reminder backs off after an unanswered turn;
  - an inject expires by TTL counted only while the receiver is live and unpaused;
  - an unlanded inject is journaled unconfirmed and never retyped;
  - mail to an orch strand is stored and never typed.
- Daemon life: started by the first verb, survives a client drop (an unacked `read` leaves mail unread), exits when its binary is replaced and comes back on the next verb, and the journal folds back to the same pool after a restart, including day rollover and retention carry-forward.
- Identity: sender derived from process ancestry on Linux, a disagreeing `LYX_STRAND_NAME` refused, EXTERNAL from a plain terminal.
- Pause: a `lyx tender pause` on the sandbox hub holds delivery, and resume lets it go on.

## Output

- A creel sandbox suite (scenarios with Watch and Verdict, in the `tools/sandbox` format) that can be re-run, beside the existing S9 scenario.
- Automated tests for every defect found, and its fix.
- A test report in the PR description: each scenario with OK / WARN / FAIL.

## Out of scope

Wiring any existing module to creel (that is `creel-notices` and `creel-agents`), and native Windows.
