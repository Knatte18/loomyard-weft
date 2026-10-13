# Support log — logging-cleanup

## Interview

Autonomous mode: no operator answered, and every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry and the related entries: `run-window-bugs` (owns `weft-push-refused-retry`), `spawn-audit-log` (owns `logger-file-level`), `go-driver-assistant` (running Webster, touches the shed driver path), `arbiter-fixes` and `orch-split`.
2. Explored the reed resize bug live.
   The `go-driver-assistant` window holds four panes, the failing layout string names every pane but `loom-status`, and reed's recorded pane ids match the window.
   The hub watchdog process (started 2026-10-08) runs a deleted binary, `AnchorTop` landed on 2026-10-12, and the first failure in the trace is 2026-10-12T14:06.
   tmux refuses the layout with "have 4 panes but need 3".
   The failures began after the `AnchorTop` landing, before `reed-multi-window` landed, and do not depend on it.
   The failing layout and the four-pane window were read from `_board/.lyx/logs/trace-20261008T181526Z-f2e3f7df172f3d30-3854.log` and `go-driver-assistant/.lyx/reed.json` in the hub.
   Conclusion: a stale-binary daemon, not a lost pane binding (D1).
3. Read the battenshed wait reporting: `report` already dedupes per reason, but some reasons carry counters, and `exemptEnd` mixes ordinary state changes with real failures (D5).
4. Read the orch watcher: the idle-disagreement line already dedupes per pair, and its repeats come from two benign combinations flipping (D8); the draft text enters through shuttle's readiness reason (D9).
5. Checked the logger sink: it opens lazily, so a Debug-only process writes no trace file (D4).
   It already switches its durable half off under `testing.Testing()`, so the stderr half can take the same test-time default (D13).
6. Checked teardown and paths: `pairteardown` is the single teardown route, the hub's `.lyx` is `_board/.lyx`, verify logs sit in the pair's verifytree directory, and batten per-slug logs live in the prime (D14).
7. Checked the land notice and the orch stencil wording that pins "newly ready" (D16, D17).
8. Lens review (six forks, all returned):
   - goal-scope BLOCKING, the live pre-task daemon never runs D2: fixed, D18 adds a one-time operator step with an acceptance row.
   - goal-scope MEDIUM, scenarios LOW: D11 had no finish line: fixed, every Warn site gets a plan verdict, with an acceptance row.
   - goal-scope LOW, board intake missing from D17: fixed.
   - goal-scope LOW, `reed-multi-window` left implicit: fixed in D1.
   - goal-scope LOW, PATTERN entry unasked: rejected, the CLAUDE.md rule requires it and D12 states why.
   - premises MEDIUM, draft text also enters through `send.go`'s busy error: fixed in D9 and its acceptance row.
   - premises LOW, pattern-fit outside-lens: D10's seen-set lifetime and concurrency: fixed, keyed per provider value under a mutex.
   - pattern-fit MEDIUM, archive copy before the run-lock wait: fixed, D14 copies after the run-lock wait, right before `Topology.Remove`.
   - pattern-fit MEDIUM, archive root has no owning accessor: fixed, one `fabricengine` accessor beside `HubLogsDir`.
   - pattern-fit LOW, suppress source in the daemon unnamed: fixed in D2.
   - pattern-fit LOW, PATTERN entry placement and enforcement: fixed in D12.
   - pattern-fit LOW, D2 vs `PATTERN-binary-change-reconcile`: fixed with one sentence in D2.
   - pattern-fit LOW, marker literal change could resend a notice: fixed, D16 keeps the existing literal.
   - pattern-fit LOW, logger tests under the discard: fixed in the acceptance row.
   - pattern-fit outside-lens, doc updates unnamed: fixed in Constraints.
   - scenarios MEDIUM, held-back landing notice lost: fixed, D16 logs Warn and does not retry, with the bound stated.
   - scenarios MEDIUM, prune age basis and foreign entries: fixed in D15.
   - scenarios LOW, partial copy and same-second collision: fixed in D14.
   - scenarios LOW, verify-log size unmeasured: fixed as an open risk; no verify logs exist on the hub to measure.
   - scenarios LOW, episode state lifetime: fixed in D3.
   - scenarios LOW, chained retirements during a multi-step deploy: fixed in D2's bound.
   - scenarios LOW, no `-v` escape after the discard: fixed, the trade-off is stated in D13.
   - board-fit MEDIUM, `run-window-bugs` shuttle overlap unnamed: fixed in Open risks.
   - board-fit LOW, `orch-split` tension: resolved; the landing notice is the signal a later run driver (`orch-split`) would take over, so it does not add to what that note plans to move.
   - board-fit LOW, `go-driver-assistant`'s assistant reads logs: fixed in Open risks.
   - board-fit LOW, `spawn-audit-log` sampler: rejected, no conflict and nothing to change.
   - board-fit LOW, name `shedverbs.binaryChanged` in the rejected alternative: fixed below.
   - leanness MEDIUM, D1 forensic detail: fixed, moved here (turn 2).
   - leanness MEDIUM, auto-mode assumptions restated decisions: fixed, cut to one line.
   - leanness LOW, Constraints repeated decision citations: fixed, trimmed.
   - leanness LOW, tallies in rationale: fixed, cut from D3, D5, D8 and D9.
   - leanness LOW, render acceptance row pinned existing behavior: fixed, cut.
   - leanness LOW, code locations in notes: partly fixed, trimmed to the pointers the decisions lean on.
   - leanness LOW, D13 import-cycle reasoning: fixed, moved to Constraints.
   - leanness outside-lens, blocked child at Info vs the board: fixed, D5 says why.
   - leanness outside-lens, D4 premise unverified: rejected, premises confirmed the lazy sink and the zero-exit path.

## Rejected alternatives

- **Watchdog: the spawner asks a stale daemon to retire** (compare builds when the lock is held, signal the holder): needs a pid or IPC channel the daemon does not have; self-check on the existing tick is simpler.
- **Watchdog: compare `buildvcs` identity instead of file stat** (as `shedverbs.binaryChanged` does for step loops): a running process cannot read the identity of the binary now on disk without running it, and a dirty dev rebuild keeps the same revision, while a stat catches any replacement.
- **Transition: a spawner retires a legacy lock holder automatically**: the pre-task daemon writes no pid or build record, and telling it from a fresh daemon that has not yet written one would risk killing a healthy daemon; a one-time operator step (D18) is bounded and safe.
- **Transition: a new lock file name for the new daemon**: the old daemon would keep running and failing beside the new one.
- **Watchdog: exit without respawning**: leaves the hub with no resize self-heal until the next `up`/`attach`/`resume`.
- **Reed: reconcile untracked panes in the watcher's re-apply**: the panes were not untracked; the layout code was old.
- **Resize failure: dedupe on the error text**: the layout string's checksum and size change with the window, so the text is not stable; an episode is.
- **Batten waits at Debug** (the inventory's suggestion): Debug never reaches the default trace, so a run study loses the wait history; once-per-change Info costs little.
- **Change shuttle's readiness reason to drop the draft for every consumer**: loom's `driver_busy` refusal names the draft on purpose.
- **Orch idle disagreement at Debug**: it is the one trace of a probe misreading a session; at Info once per episode it stays visible.
- **A static scan that forbids Warn in loops**: loop context and state changes cannot be judged mechanically with useful precision.
- **Archive retention as a config key**: needs a hub-wide config module for a value nobody asked to tune.
- **Move the pair's logs instead of copying**: a refused removal would leave the pair without its logs.
- **Archive shuttle run directories**: not asked for; trace logs carry shuttle's engine log lines.
- **Archive batten per-slug logs**: they live in the prime, which is never torn down.
- **Landing fallback: compare the board against the landings the orch handled**: nothing records the handled set, so a refresh, compact or handoff loses it.
- **Landing fallback: read the hub notice log's landed lines since the orch's last deploy**: the time of that deploy is recorded nowhere either; `origin/prod`, which `update-plugins.sh` moves, records it as a commit.

## Review rounds

- Round 1 (BLOCKING): D13 moved from `tmuxkit.Main` into the logger, since a tmuxkit import of the logger closes an import cycle in `internal/dotgit`'s tests; D11 bounded to listed loop and polled sites with a code-observable acceptance row; D16 and D17 keep a next-wake fallback for a dropped landing notice; D5 splits the reason-file write from the log dedupe and has callers pass the key; forensic and board-fit narrative moved here.
- Round 2 (APPROVED): D17's dropped-notice fallback keys on `origin/main` holding a commit `origin/prod` lacks, with D16 and the acceptance row naming the same baseline; D11 and the Scope bullet narrowed to Warn and Info-or-above lines in a loop or polled path.

## Question ledger

- Q1 (resolved, auto-pick): which pane the resize layout leaves out. Answer: `loom-status`, the top-anchored strand, dropped by a watchdog running a binary older than `AnchorTop` (D1).
- Q2 (resolved, auto-pick): whether reed's recorded pane list and the live window disagree. Answer: no; they agree (D1).
- Q3 (resolved, auto-pick): how to stop a stale watchdog. Answer: self-check of its executable's file identity on the existing tick, then retire and respawn (D2).
- Q4 (resolved, auto-pick): level for batten waits. Answer: Info once per state; Warn only for a failed step (D5).
- Q5 (resolved, auto-pick): how to fix the orch idle-disagreement repeats. Answer: treat the two benign combinations as agreement, Info once per episode (D8).
- Q6 (resolved, auto-pick): how to keep draft text out of orch logs. Answer: a draft kind on the idle probe and a constant orch reason (D9).
- Q7 (resolved, auto-pick): how far the "other modules" Warn check reaches. Answer: every Warn or Info-or-above line in a loop or polled path, against stated criteria; a Warn site in no loop is out of the audit (D11).
- Q8 (resolved, auto-pick): whether the policy refinement needs a PATTERN entry. Answer: yes, `PATTERN-log-levels`, because agents read PATTERN and not `logger.go` (D12).
- Q9 (resolved, auto-pick): archive retention period and config. Answer: 30 days, a Go constant (D15).
- Q10 (resolved, auto-pick): whether batten per-slug logs and verify logs join the archive. Answer: verify logs yes; batten per-slug logs no, since the prime keeps them (D14).
- Q11 (resolved, auto-pick): the archive path. Answer: `<hub>/_board/.lyx/log-archive/<slug>/<UTC timestamp>/` (D14).
- Q12 (resolved, auto-pick): the landing notice's wording and the orch's handling. Answer: `<slug>: landed[; newly ready: …]`, handled under After landing (D16, D17).
- Q13 (resolved, auto-pick): how the live pre-task daemon picks up the fix. Answer: a one-time operator step after the deploy (D18).
