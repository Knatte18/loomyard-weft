# Support log — parent-review-wait

## Interview

Autonomous run: no operator answered, so each turn below is the writer's own exploration and pick.

1. Read the board entry and the related entries `orch-wake-policy`, `alignment-agent`, `orch-split` and `driver-arbiter`; `shuttle-send` is no longer on the board, and the entry already moves mid-turn restart sends there.
2. Read `internal/parentreview/gate.go` and `doc.go`, the `Wait` loop and `handleGatedBoundary` in `internal/shuttleengine/wait.go`, `GateEntry`, the gate resolver in `internal/shedrecipe/entries_gate.go`, the loom recipe's `parent-review` element (`attempts: 1`, `pass_on_cap: true`, from commit 518dc7400), the review verbs in `internal/loomcli/review.go`, `RecordVerdict` and `PrepareRound` in the store, and the brief, delivery and orch role stencils.
3. Part 1 (warm cache): picked a gate-carried ping every 4 minutes after delivery, capped at 3 per round, over a fresh session on late reject (see Rejected alternatives).
   The first draft had no per-round cap; working the cost through showed 14 pings over a 60-minute wait cost more than the cold resume they avoid, so the cap was added.
4. Wait bound: picked a read-only expiry probe on `GateEntry`, called off the boundary, with finalize marking the request expired.
5. Live smoke: first closed with the `daemon-wakeups` reject (2026-10-10) as evidence; the lens review showed that evidence does not isolate the writer's hop, so this run's own parent review became the proof (D11).
6. Rewrite after reject: dropped pass-at-cap for `parent-review` and set `attempts: 2`; with one attempt and no pass-at-cap the first reject halts before the writer sees it.
7. The remaining PR #336 items (poll throttle, prime way-forward, empty `--review`, entry order, `delivered` after a verdict, round numbers, review scope) each took the smallest change that makes code and spec agree.
8. Lens review: six forks spawned (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness); all six returned.
   Duplicates merged; one line per finding:
   - premises, scenarios, pattern-fit, leanness (MEDIUM): the 3-ping cap and the ping clock lived in closure memory and reset on restart, and D1's fresh-closure clause was unreachable — fixed: ping count and last-ping time recorded in `delivery.json`, clause dropped, a failed ping counts, restart acceptance line added.
   - pattern-fit (MEDIUM): identical ping text meets shuttle's pending-send dedupe — fixed: each ping carries its number.
   - scenarios, goal-scope (MEDIUM): D2's probe ran only after an in-memory pending, so an attach to a stuck writer never reached it — fixed: probes run off the boundary whether or not the loop saw the entry pend; attach acceptance line added.
   - goal-scope (MEDIUM): `shuttle-send` is off the board — fixed: Out now cites its landing, commit e12fb6afb.
   - goal-scope, premises (MEDIUM/LOW): the live-smoke evidence from `daemon-wakeups` did not isolate the writer's hop (batten's waiting-review note is a second route) nor show a hand-started orch — fixed: new D11 makes this run's own round-1 delivery stamp and verdict the proof, with an acceptance line.
   - board-fit (MEDIUM): D10 overlaps `orch-wake-policy` on how a cap-halted run resumes — fixed: D10 uses the gate's existing terminal message, and Out leaves who resumes a halted run to `orch-wake-policy`.
   - board-fit (MEDIUM): overlap with `creel` stage 3 unnamed — fixed: Out names it.
   - board-fit (LOW): `config-layers-run-settings` owns per-task cap overrides — fixed: named in Out.
   - board-fit (LOW): under `alignment-agent` D4's rule would need to carry over to a cold agent — rejected: that agent is unbuilt, and its own design carries its brief.
   - scenarios (LOW): D2 called the probe "reads nothing" though it reads the store — fixed: read-only, unlocked, `Final` re-reads under the lock.
   - scenarios (LOW): `delivered --failed` after a verdict unspecified — fixed in D8 and its acceptance line.
   - scenarios, pattern-fit (LOW): "hand fix" at the cap names no path, and the writer never sees the cap's findings — fixed: D10 names the terminal message's way forward; the unseen findings are an Open risk.
   - scenarios (LOW): runs in flight across the binary change — fixed: D10 states the resume behaviour.
   - scenarios (LOW): `PrepareRound` racing a gate opening a request — rejected: preparation runs before the writer spawns, when no gate of that run is evaluating.
   - scenarios (LOW): acceptance for a turn start between probe and finalize — rejected: finalize's evaluation is the same either way, covered by the verdict-race line.
   - pattern-fit (LOW): the `wait.go` doc that counts finalize sites — fixed in D2.
   - pattern-fit (LOW): D6's refusal spec row names `reject` — fixed: D6 corrects the row, and Constraints names D6.
   - pattern-fit (LOW): D7 and D10 refusals have no clear reader — fixed: Constraints names them recipe-build refusals reached by a developer.
   - pattern-fit (LOW): stencil claims and markers tests — fixed: Notes point at them.
   - pattern-fit (LOW): import-set citation — fixed: cites `parentreview`'s `doc.go`.
   - goal-scope (LOW): review-scope confirmation silently assumed — fixed: an Open risk addressed to the parent reviewer.
   - goal-scope (LOW): D10 reverses 518dc7400 unnamed — fixed: named in D10.
   - goal-scope (LOW): operator-typed turns unaddressed — fixed: Out names the existing handling.
   - leanness (LOW): Scope In repeated the Decisions — fixed: Scope In is "D1 to D11" plus the writer and seats boundary.
   - leanness (LOW): old D11 decided nothing — fixed: moved to Scope Out.
   - leanness (LOW): Auto-mode assumptions repeated rationale — fixed: trimmed to one line each, meta bullet dropped.
   - leanness (LOW): Constraints repeated Decision rationale — fixed: duplicated tails cut from D1 and D2.
   - leanness (LOW): D2's Bound and Open risks both stated the mid-turn strand removal — fixed: kept in Open risks only.
   - leanness (LOW): "(see Decisions)" in Out pointed at nothing — fixed by the new D11.

## Rejected alternatives

- **Fresh writer session on a late reject**, seeded with the decision record, the support log and the findings: loses the writer's exploration, so the rewrite re-explores the code at a cost close to the cold resume it replaces; the respawn path also has to be taught not to re-run the lens review.
- **The writer waits on its own verdict with capped `lyx shuttle await` calls, as the Burler fixer does**: each re-run costs the same cache read as a ping, and it moves the verdict hand-off from the gate's boundary send into the writer's turn, re-architecting a gate that works.
- **Uncapped pings until the wait bound**: worst case 14 pings, more than one cold resume.
- **Mapping the discussion role to a `1h` prompt-cache TTL instead of pinging**: no ping is needed, but every cache write of the writer's session costs about 2 times instead of 1.25 on every run, more than three pings on the runs that need them (D1, Rationale).
- **A 4-minute interval constant in `internal/parentreview`**: spells a Claude cache lifetime outside `claudeengine` and pings a writer whose role is mapped to `1h` for nothing.
- **Narrowing D2's probe so it never fires while the mechanical entry fails**: a writer that never ends its turn is held to the run deadline and then finalized over the same tree, so the narrowing only delays the same halt.
- **Isolating D11's hop with the orch's transcript**: no lyx verb reads it today, so the gap is raised to the parent reviewer instead.
- **Pings while delivery is unrecorded**: a notice the reviewer never got yields no verdict, so those pings buy nothing.
- **Evaluating the whole gate closure off the boundary**: its carry path records prompts it could not send mid-turn, spending the delivery budget; a read-only probe avoids that.
- **Keeping pass-at-cap and notifying the parent when the rewrite passes unreviewed**: still lets an unreviewed rewrite reach Plan, which the board entry rules out.
- **`attempts: 1` without pass-at-cap**: the first reject halts the run before the writer sees its findings.
- **`delivered` after a verdict refusing**: the writer would treat a no-op as a failure to fix.
- **A tmux-tier or llm-tier live smoke test**: the hop has run live already, and an llm-tier test of a two-session SendMessage exchange is expensive to keep.
- **Widening the parent brief to millhouse's `orch-review` scope**: duplicates the Discussion-Review perch and raises orch spend.

## Review rounds

_No rounds yet._

## Question ledger

- Ping or fresh session for a late reject? Auto-pick: ping, capped (D1). Resolved.
- Ping interval and cap? Auto-pick: 4 minutes, 3 per round, constants in `internal/parentreview`, with the count and last-ping time in `delivery.json`. Resolved.
- Ping before delivery is recorded? Auto-pick: no. Resolved.
- How does expiry fire mid-turn without a send? Auto-pick: read-only probe plus finalize (D2). Resolved.
- Throttle period? Auto-pick: 5 seconds (D3). Resolved.
- Live smoke: test or evidence? Auto-pick: this run's own round-1 delivery stamp and verdict, no test (D11). Resolved.
- Where does the no-reply rule go? Auto-pick: the brief and the orch role stencil (D4). Resolved.
- `delivered` after a verdict: refuse or no-op? Auto-pick: no-op success (D8). Resolved.
- Empty round directories: skip `PrepareRound` without a reviewer, or reuse a requestless round? Auto-pick: reuse, which also covers a spawn that died before its gate (D9). Resolved.
- Rewrite after reject: review it or escalate at the cap? Auto-pick: both; every rewrite is reviewed and the cap halts (D10). Resolved.
- Reject cap value? Auto-pick: `attempts: 2`. Resolved.
- Review scope (the board entry asks to confirm with the operator): auto-pick: keep scope-only (Scope, Out); unconfirmed by the operator, raised to the parent reviewer under Open risks.
