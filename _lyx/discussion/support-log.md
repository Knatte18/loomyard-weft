# Support log — reed-multi-window

## Interview

Autonomous run: no operator answers; every pick below is the writer's, recorded in the ledger.

1. Read the board entry and the related entries (`run-window-bugs`, which depends on this task; `pause-esc-rewind`; `orch-split`; `seats-replaces-singlellm`; `creel`; `discussion-multi-llm-bench`).
   The entry folds three ideas: own-window anchoring (now wanted, producer as the unit), strands in depth (2026-10-05), and the extra agent strand; two ideas are dropped.
2. Explored reed: `render.AnchorOwnWindow` is declared and refused; every op addresses one strands' window resolved through Selvage; `OpenWindow` already opens detached hidden windows; the status bar already gives non-strand windows buttons.
3. Explored shuttle and its spawners: an empty anchor defaults to `below-parent`; the loom driver sets it, the orch does not; seats declare a sharing size.
4. Settled scope, the group-size rule, split setting, focus and announcement, fallback, resume, the shuttle default and the open verb (D1–D13).
5. Lens review (six forks, all returned):
   - goal-scope MEDIUM / scenarios BLOCKING / premises LOW — died, timed-out and kept-pane runs never close their window: fixed in D5 (remain-on-exit on, dead window until the owner removes the strand; seat tables remove their advisors before returning) with an acceptance line.
   - scenarios BLOCKING / premises MEDIUM — group keyed by role prefix lets a later round join an earlier window: fixed in D11 (group names the table instance, prefix plus round).
   - scenarios MEDIUM — remain-on-exit for own windows undecided: fixed in D5.
   - scenarios MEDIUM / goal-scope LOW — Resume of an opened session unspecified: fixed in D12.
   - scenarios MEDIUM / pattern-fit MEDIUM — the open verb is unbounded and skips the pause hold: fixed in D12 (refuses under a pause; count bound stated).
   - scenarios MEDIUM — mismatched `WindowSize` in one group: fixed in D2 with an acceptance line.
   - pattern-fit MEDIUM — `reed.yaml` is per-worktree, not per-hub: fixed in D3.
   - pattern-fit MEDIUM — the own-window end has no bound: fixed in D6 (`@lyx_own` only, under the op lock, after bindings persist).
   - premises MEDIUM — no binary change restarts the watchdog: fixed, moved from Open risks to Constraints as a fact Plan designs for.
   - board-fit MEDIUM — self-removal could fire into a teardown: fixed in D12 (teardown ends the pane shell first).
   - board-fit MEDIUM — the detached-window size has no owner: fixed, Open risks hands it to `run-window-bugs`.
   - pattern-fit / scenarios / premises LOW — hook install point and psmux fallback: fixed in D4.
   - scenarios LOW — `OpenWindow` name collision: fixed in D6 with an acceptance line.
   - scenarios LOW — a hand-killed own window: fixed in D6 (reads absent, as today).
   - scenarios LOW — attach to a window 0 strand after deploy: fixed in D10 with an acceptance line.
   - scenarios LOW — open-verb numbering and pause scenarios: fixed in acceptance criteria; the no-session case is rejected, since a strand add boots the session as every add does today.
   - scenarios LOW — two groups added at once: rejected, as the per-worktree op lock serialises adds and D2's rule is order-independent.
   - premises LOW — "nothing types into a pane" is false for name repair: fixed, constraint removed and D12 states the rename.
   - premises LOW — the self-removal chain is new: fixed in D12.
   - board-fit LOW — the window 0 exception list should be a rule: fixed in D10.
   - board-fit LOW — one-seat tables under `seats-replaces-singlellm`: fixed in D11.
   - board-fit LOW / pattern-fit LOW — missing `PATTERN-spawn-observability`, `PATTERN-shell-mechanics-seam`, `PATTERN-cliwire-sole-wiring`, literal role: fixed in Constraints.
   - board-fit LOW — tabs without keys rest on tmux's default prefix keys: fixed in Scope Out.
   - goal-scope LOW — keeping zoom departs from the literal board text: kept, as Q7 records the reading.
   - goal-scope LOW — psmux behaviour unstated: fixed for the hook in D4; the rest follows reed's existing optional-verb handling.
   - leanness LOW — D13 restates a no-change: fixed, moved to Scope Out.
   - leanness LOW — duplicated and no-op constraints, restating auto-mode bullets, doc list: fixed by trimming them.
   - leanness LOW — D4's option names are mechanism: rejected, as the names make the display-only bound checkable.

## Rejected alternatives

- Overflow by arrival (a group window takes strands until full, later ones go apart): the arrangement would depend on arrival order, and a five-seat table would show two together and three apart, against the operator's line.
- The spawner decides shared or per-strand itself: every spawner would read reed's cap; declaring the group size keeps the policy in reed.
- Keeping the shuttle default at `below-parent` and setting `own-window` at each producer site: many sites, and a new producer would silently land in window 0.
- tmux's activity flag as the announcement: a Claude pane redraws constantly, so the flag would be set almost always.
- An automatic switch when the current window is idle: the operator rules out focus moves on spawn, and idle is hard to judge for a reading operator.
- Removing zoom: window 0 still holds several panes worth zooming.
- Moving a live strand between window 0 and an own window: not asked for, and it needs `join-pane`/`break-pane` handling across every op.
- Putting `lyx shuttle open` in reed: reed is below shuttle and knows no provider.
- New key bindings for stepping between windows: not asked for; the status bar click and tmux's own window keys remain.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Which agents get own windows? Every shuttle run by default; the loom driver and the orch stay in window 0 (D10).
- Q2 (auto-pick): Who decides shared versus per-strand? reed, from the group size the spawner declares and `window_share_max` (D2).
- Q3 (auto-pick): Share cap default? 2 (D2).
- Q4 (auto-pick): Where is the split direction set? `reed.yaml` `window_split`, default `width` (D3).
- Q5 (auto-pick): How does a new window announce itself? An unseen-marked status bar button cleared on first visit (D4).
- Q6 (auto-pick): Does the operator follow the window when it closes? Only a client viewing it moves, to window 0 (D5).
- Q7 (auto-pick): Does zoom go? No (Scope Out): window 0 still holds several panes worth zooming, so the own window replaces zoom only for spawned agents.
- Q8 (auto-pick): Is the extra agent strand in scope? Yes, as `lyx shuttle open` (D12), since the entry folds it in and names its layout as answered by own windows.
- Q9 (auto-pick): Anchor changes into or out of `own-window`? Refused with a way forward (D9).
- Q10 (resolved by the premises lens): Does a binary change restart the hub watchdog daemon? No; Constraints has Plan design for it.
- Q11 (auto-pick): Does `lyx shuttle open` wait or refuse under a pause? It refuses, since a CLI verb the orch runs must not block indefinitely (D12).
- Q12 (auto-pick): What happens to a dead agent's own window? It stays showing the dead pane until the strand's owner removes it (D5).
