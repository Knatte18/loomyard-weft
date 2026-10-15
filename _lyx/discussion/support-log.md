# Support log — twb-d0-ha

## Interview

Autonomous mode: no operator answered; every pick is in the question ledger.

1. Read the board entry and `pause-background-work`, which records a hibernate request failing confirmation on background work and the orch's turn, and `orch-split`.
   Read `tenderengine`, `tendercli`, `pauserecord`, `orchengine`, `hubgeom`'s pause geometry, batten's Land row and `shortcutcli`.
2. Settled the trigger forms, the firing point (Land row seam), the waiter as a detached daemon verb, confirmation's predicate and bound, the resume after wake, and the shortcuts.
3. Lens review: all six forks returned.
   - premises, pattern-fit, scenarios: `tenderengine.Resume` with an empty slug lifts slug entries too — fixed: D8 adds a scope-exact hub resume, D2 records the waiter's entry.
   - premises, pattern-fit: D5 cited `selfexec` as the spawner — fixed: `proc.Detach` as `orchcli` does.
   - premises: adopt `selfexec.Check` in the waiter — rejected: an older binary running the waiter's logic is harmless, kept as an open risk.
   - premises, pattern-fit, scenarios: waiter's caller identity — fixed: spawn drops the strand-name variable, caller `hibernate` (D5, D7).
   - premises: land lock has no read-only probe — fixed: try-acquire released at once (D7).
   - premises: D7's bound loose on who sends keys — fixed: keys go only through the pause seam.
   - pattern-fit: daemon-wakeups declared list — fixed in Constraints.
   - pattern-fit: D1 Deps vs direct `tenderengine` calls — fixed: engine works over seams, imports no `tenderengine`.
   - pattern-fit: Land seam geometry — fixed: path told through `LandDeps` (D6).
   - board-fit, leanness: orch-turn premise false and `pause-background-work`'s orch point unnamed — fixed: D7 gives the orch no exemption and says why.
   - board-fit: twin bench entries, `orch-split`, `shed-tackler` — no change needed.
   - board-fit: route spawns through `spawn-audit-log`'s launch path — rejected: unbuilt; logger logging meets today's entry.
   - goal-scope: `--after` refusal blocked runs waiting on the orch — fixed: D4 accepts any batten run.
   - goal-scope: background-work dependency — rejected treating background as idle (`pause-background-work`'s call); the dependency is named in Scope, D7 and Open risks.
   - goal-scope: resume after wake unlicensed — rejected: kept as an auto-pick with rationale, listed in Auto-mode assumptions.
   - goal-scope: plain verb point implicit — fixed in Scope.
   - goal-scope: `--when-idle` semantics — kept, flagged in Auto-mode assumptions.
   - goal-scope: Goal said orch only — fixed.
   - scenarios: concurrent requests — fixed: record lock in D2/D3, criterion added.
   - scenarios: cancel racing later states — fixed: re-read before each move, cancel rules per state, criteria added.
   - scenarios: hub entry lifted mid-confirmation — fixed: fails the request (D7).
   - scenarios: pause halting another Land — fixed: waiter waits for the land lock before pausing; residual case in Open risks.
   - scenarios: `watch` bound and caller check — fixed (D3).
   - scenarios: `--when-idle` with no run — fixed: triggers at once, criterion added.
   - scenarios: wall or awake time — fixed: confirmation wall, sleep wait awake.
   - scenarios: criteria gaps (wake notice, dead waiter, unknown key) — fixed.
   - leanness: D6 rationale named a rejected alternative — fixed.
   - leanness: Constraints repeated Decision citations — fixed: trimmed.
   - leanness: background limitation stated thrice, assumptions restating decisions, Goal re-listing Scope, suspend line, exact symbols in notes — fixed.

## Rejected alternatives

- The orch runs `systemctl hibernate` itself: the board's direction rules it out, since the session hangs mid-call.
- A plain `lyx hibernate` verb that hibernates in its own process: the caller is an LLM session, the same hang.
- The waiter polls batten's status file for the slug's Land: races the pair's teardown; the Land row marking the request does not.
- A machine-wide request under the user's state directory covering every hub: no hub registry exists and the operator runs one hub.
- Leaving the hub paused after the wake: every run would hold until someone typed `lyx resume`.
- Treating background-only sessions as idle in confirmation: that is `pause-background-work`'s design choice, not this task's.
- `systemctl suspend` as a fallback when hibernate fails: not asked for.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): what "every instance" means. Pick: every session in the hub; one hub per machine.
- Q2 (auto-pick): trigger forms. Pick: now, `--after <slug>`, `--when-idle`, all from the board's points.
- Q3 (auto-pick): who fires. Pick: the Land row marks the request after deploy and prune; the detached waiter does the rest.
- Q4 (auto-pick): what confirmation requires. Pick: no busy session (turn or background), no network op, land lock free, within ten minutes.
- Q5 (auto-pick): resume after wake. Pick: the waiter lifts its own entry once `SleepWatch` sees the sleep end.
- Q6 (auto-pick): shortcuts. Pick: add `p` and `r`.
- Q7 (auto-pick): a hub entry already live when the request triggers. Pick: use it, take none, lift none.
- Note: the `lyx` on `PATH` is a later build than this worktree's HEAD; the design reads HEAD only.
