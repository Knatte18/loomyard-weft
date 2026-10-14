# Support log — perch-budget-arbiter

## Interview

Autonomous session: no operator answered, so each turn below is an exploration finding and the writer's own pick.

1. Read the board entry and the related entries (`orch-split`, `overview-seats`, `model-downgrade`, `report-timing`, `codex-engine`, `hardener`, `discussion-multi-llm-bench`).
   None duplicates this task; `report-timing` owns counting arbiter spawns, `orch-split` the driver split.
2. Checked what landed since the entry's arbiter note (2026-10-12): `072e2ed86` (judge-cap-call) moved the soft-cap call to the Bouncer's rule and added `hard-cap`; `1c812c1f8` added the Go driver and tackler; `30385e1ae` made the verb refuse a Merge-In verify halt naming a target.
   So the board table's soft-cap, no-BLOCKING and Merge-In rows are already done; the remaining arbiter work is narrowing awaiting eligibility, the driver stencil, and the double notice.
3. Darn config: `review_max_bounces` and `review_hard_max_bounces` are `CategoryGuard` in a `HubWide` module; `PrepareRunSettings` runs `ValidateLocal`, so run_settings refuses them.
   Hub-wide modules already read the worktree local layer for category-none keys, so making them none is enough for per-run use.
   Picked a shared package for the key family (D1), `skip_review` as the off switch (D3) and a migration (D4).
4. Double notice: `notices.log` shows batten state-change notices for `logging-cleanup` (09:26:22, since 09:22:51) and `tender` (15:32:29, since 15:28:56), both "report none yet", while the arbiter ran.
   Batten's `arbiterHolds` should hold; picked a hold that starts at the stop episode for an arbiter-eligible stop with a bounded deadline, and left the precise root cause to Plan (D9).
5. Eligibility single source: picked the verb as the one filter for the LLM driver (D8) and one shared Go function for the verb and Go driver (D5).
6. Lens review, all six forks returned:
   - premises MEDIUM, `budget` also arises from a re-escalated round and pre-upgrade runs: fixed, D5 lists every source and sends all to the parent.
   - premises LOW (D4 reconcile path, D2 upsert path, D5 seam, tender timing): confirmations, no change.
   - goal-scope MEDIUM, D8 departs from the entry's stencil narrowing silently: fixed, D8 says it replaces it and covers PR-Gate.
   - goal-scope MEDIUM / scenarios MEDIUM ×3, D9's hold scope, end conditions and backstop: fixed, D9 now a pre-marker hold for awaiting and blocked stops only, ending on a stop report, park marker, hand-on or applied resume, then the marker hold governs; criteria pin the incident shape, tackler-first and restart.
   - goal-scope LOW, per-run "no review" stays impossible: fixed, D2's bound says so.
   - goal-scope LOW / pattern-fit LOW, `PATTERN.md` in Scope: fixed, only the background file.
   - goal-scope LOW, D1 widens the loom diff: rejected, licensed by the entry and covered by the loom-unchanged criterion.
   - goal-scope LOW / scenarios MEDIUM / leanness 10 / pattern-fit LOW, D4 run_settings refusal and migration criteria: fixed, refusal wording, spec row and criteria added.
   - pattern-fit MEDIUM, D2 downgrades a guard without a bound: fixed, bound added.
   - pattern-fit MEDIUM / scenarios LOW, D4 runs at reconcile, not load: fixed, D4 names `hubreconcile.Ensure` and the load refusal's way forward.
   - pattern-fit MEDIUM, batten must use D5's function: fixed in D5 and D9.
   - pattern-fit LOW, every key stays classified: fixed in D1 and Constraints.
   - board-fit MEDIUM, `overview-seats` overlap: fixed, Open risks.
   - board-fit LOW, `report-timing` source records: fixed in Scope Out.
   - board-fit LOW, `orch-split`: no conflict, no change.
   - board-fit LOW, "perch" in a package name: fixed, renamed `internal/reviewbudget`.
   - scenarios LOW, per-run caps can park a run: fixed, Open risks.
   - scenarios LOW, orch race via other signals: fixed, D9 keeps the orch stencil guidance.
   - scenarios LOW, lightweight status reads `skip_review` false on an unreadable file: fixed, D3 states it.
   - scenarios MEDIUM, unreadable or unknown escalation cause: fixed in D5 and criteria.
   - leanness MEDIUM, D9 incident narrative and duplicate open risk: fixed, the narrative is cut to the requirement and the duplicate risk removed.
   - leanness MEDIUM, rationales argue against alternatives: fixed in D1, D6, D8.
   - leanness MEDIUM, D10 is a non-change: fixed, moved to Scope Out.
   - leanness LOW, D6 is a non-change: rejected, it answers the entry's open point on case 3.
   - leanness LOW, Auto-mode assumptions repeat decisions, Goal repeats triggers: fixed.
   - leanness LOW, `file:line` in Notes: fixed.
   - leanness LOW, generic Constraints: partly fixed, `PATTERN-tolerant-records` cut; the rest bind a decision's refusals or notice.

## Rejected alternatives

- A parity test between darn and loom config instead of a shared package: detects drift after it happens rather than preventing it.
- Moving Darn-Review's keys into `loom.yaml` as a `darn_` perch: splits darn's config across two files, one per-worktree and one hub-wide.
- Renaming darn's budget keys to `darn_max_bounces`: churn and a second migration for no reader benefit.
- Per-run off switch for darn: hollows a review gate from a board field; the entry keeps the guard on the switch.
- Keeping the arbiter for `budget` escalations: those now mean unreadable round inputs, which its verbs cannot repair.
- An `escalation_cause` field on the loop envelope so the LLM stencil can restate the list: generic shed plumbing for a second copy of the rule.
- Dropping case 3's arbiter spawn: the fork or tackler already filters it, and the arbiter's purpose is saving orch turns.
- A recipe edge for the Merge-In verify goto: batten's retry and the Go driver's rule already do it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the shared budget shape live? — A new leaf package `internal/reviewbudget`, since `darnengine` must not import `loomengine`.
- Q2 (auto-pick): Name of darn's off switch? — `skip_review`, matching darn's run-wide `review_*` keys.
- Q3 (auto-pick): Migrate old `review_max_bounces: 0`? — Yes, to `skip_review: true`, so a hub's meaning survives the binary change.
- Q4 (auto-pick): Does darn get an auto-accept key? — No; not asked.
- Q5 (auto-pick): Who settles a `budget` escalation? — The parent.
- Q6 (auto-pick): Is case 3 worth an opus spawn? — Yes, behind the fork or tackler.
- Q7 (auto-pick): Merge-In verify goto as a recipe edge? — No.
- Q8 (auto-pick): How does the LLM driver know eligibility at an awaiting `parent_notice` stop? — It calls the verb; a refusal is the planned hand-off.
- Q9 (auto-pick): Fix for the double notice? — A batten pre-marker hold for awaiting and blocked eligible stops from the stop's own time, bounded by the arbiter timeout plus the report wait, then the existing marker hold; the driver relays only after the arbiter.
- Q10 (auto-pick): Where does a `budget` escalation go, whatever its source? — The parent.
