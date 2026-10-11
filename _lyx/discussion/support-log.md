# Support log — driver-arbiter

## Interview

1. Read the board entry and the related entries (`orch-split`, `alignment-agent`, `run-message-routing`, `convergence-plan`, `weaver`, `config-layers-run-settings`), and the two latest commits, which moved landing into batten (`3a627c00d`) and split Burler rounds with per-perch auto-accept (`b2e4cf913`).
2. Read the driver stencil, its repair guide and both notify rules, the orch role stencil, the Bouncer escalation brief and parent notice, batten's InnerRun notice step, and the `loom resume`, `decision` and `circling` verb headers.
   Found: only the Bouncer writes `parent_notice`; `decision add` refuses a task session's strand; `circling` is bounded partly by no stencil naming it; the driver runs `haiku[medium]`.
3. Auto-pick: the driver spawns the arbiter through a loom verb (D1), against a batten-spawned arbiter; see Rejected alternatives.
4. Auto-pick: the arbiter records, the driver resumes (D3, D6).
5. Auto-pick: eligibility in Go with a closed list, excluding the batten-owned Publish halt (D2).
6. Message from `ly:orch` (operator decision, 2026-10-11): the spawn budget is a config key and a spent budget escalates to the parent as the final step; key name and default left to the writer.
   Recorded as D7 and acknowledged to `ly:orch`.
7. Auto-pick: keys `arbiter`, `arbiter_timeout_min`, `arbiter_max_spawns` with defaults `opus[high]`, 30, 3 (D7, D8).
8. Auto-pick: batten holds its notice while an arbiter is in flight (D10).
9. Lens review: six forks spawned (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness); all six returned. Duplicates are merged; each line gives the lenses, then the disposition.
   - board-fit (BLOCKING): eligibility keyed on `parent_notice` would admit a future parent-review escalation. Fixed: D2 now keys on a Bouncer escalation, naming `parent-review-wait`.
   - board-fit: zero values of the new keys in a file that lacks them. Fixed: D7 and D8 give each zero a safe meaning and name `config-layers-run-settings`.
   - board-fit: `orch-split` not named. Fixed: the Goal and D13 say "the driver's parent", and Notes names both stencil co-editors.
   - board-fit: shared machine with `alignment-agent`. Rejected: that note is unbuilt, so building for it is YAGNI.
   - board-fit: a class field for `convergence-plan`. Rejected: the verdict already carries the reason, and the class taxonomy is that note's to design.
   - board-fit: `lyx-halt` pause. Rejected: unbuilt.
   - board-fit: the board's dangling `orch-wake-policy` reference. Rejected: board housekeeping, not this record.
   - goal-scope, premises: `interrupted` under `handback` reopens the restart guard. Fixed: excluded in D2 and Scope.
   - goal-scope, premises: `blocked` eligibility reverses the driver's never-repaired rule, unbounded. Fixed: D6 names the reversal and bounds it by the goto precondition, the budget and the same-stop rule.
   - premises, scenarios: Go cannot see the fork's escalation. Fixed: D2 states it as a stencil-only bound, and the verb takes the envelope path.
   - goal-scope: arbiter powers at error stops. Fixed: stated in D3.
   - goal-scope: repair-cap interaction. Fixed: D6 allows the re-launch over a spent cap, bounded by the budget.
   - goal-scope, scenarios: a `resume` that cannot move the run. Fixed: D6 adds Go precondition checks, with an acceptance criterion.
   - goal-scope: the second Publish failure still wakes the parent. Rejected: the verb cannot tell first from second, and batten owns the halt; the boundary is stated in Scope.
   - scenarios: driver death mid-arbiter. Fixed: D1 keys attempts to the stop, D9 and D10 put the marker on the shuttle run with its deadline, D6 adds the unparked-wake rule, and an acceptance criterion covers it.
   - scenarios: one recurring stop spends the whole budget. Fixed: D7's same-stop rule.
   - scenarios: a refused arbiter verb. Fixed: D3, hand on quoting the refusal.
   - scenarios: a stale watched marker. Fixed: bound stated in D2.
   - scenarios: `unseeded` has no run directory. Fixed: excluded.
   - scenarios: concurrency criteria. Fixed: criteria for an orch goto during a live arbiter, and batten's revive skipping the arbiter strand (D11).
   - premises: D10 read the pair's config from batten. Fixed: the deadline is recorded in the marker.
   - premises: the D12 rationale over-claimed reaching Plan. Fixed.
   - premises, pattern-fit: misread of PATTERN-driver-choice-single-site. Fixed: the Constraints entry is restated.
   - pattern-fit: "question for the operator" wording. Fixed in D3 and D11.
   - pattern-fit: no-parent variant ownership. Fixed: D11 owns the choice.
   - pattern-fit: role-constant owner and the agentname compare. Fixed in D11 and D12.
   - pattern-fit: cobra audience. Fixed in D1.
   - pattern-fit: the driver stencil's goto sentence. Fixed in D6.
   - pattern-fit: the circling premise. Fixed: D3 names the escalation brief as the first stencil.
   - pattern-fit: a new PATTERN entry. Rejected: "the arbiter records, the driver re-launches" is local to two stencils and one verb, and is pinned by the arbiter stencil test over its verbs; no other module must obey it.
   - leanness: D6's requirement sat in Notes. Fixed: moved into D6.
   - leanness: Scope In restated the decisions. Fixed: it now points at them.
   - leanness: auto-mode assumptions restated decisions. Fixed: one pointer line.
   - leanness: duplicated PATTERN lists. Fixed: kept in Constraints only.
   - leanness: D13 folded into D3. Fixed.
   - leanness: Open risks repeated D1 and D10. Fixed: shortened.

## Rejected alternatives

- **Batten spawns the arbiter** in place of its orch notice: one Go owner of what reaches the orch, serialised with the Publish retry by construction.
  Rejected: the board entry and the operator's budget decision both name the driver as spawner and escalator, and runs no batten watches would get no arbiter.
- **The arbiter resumes the run itself** with `lyx loom resume`: rejected, since the driver would then be a second party racing the arbiter's resume, and the driver is the loop's owner.
- **The arbiter only recommends, the driver executes every verb**: rejected, since a Haiku driver would then run `circling accept` on another agent's say-so with no added check, whereas the one move the driver makes on a verdict, its re-launch's goto, is checked in Go against the attempt's recorded state (D6), and the arbiter's records would carry the driver's strand name.
- **Arbitrate the batten-owned Publish halt too**: rejected, since batten's goto and resume would race the arbiter on the same halt, and the halt's way forward is known.
- **Eligibility keyed on `parent_notice`**: rejected after lens review, since a later parent-review escalation could then reach the arbiter.
- **`interrupted` under `handback` eligible**: rejected after lens review, since a live agent may still run and a re-launch would restart it.
- **A PATTERN entry for the arbiter's powers**: rejected, since the rule binds two stencils and one verb and no other module.
- **A fork of the driver as the arbiter**: rejected, since a fork runs on the driver's model and the board entry asks for a cold, stronger agent.

## Review rounds

- Round 1 (BLOCKING): at an error stop D1's attempt key and D7's same-stop rule matched the same case, since a recurring producer error appends no history. Fixed: one stop key named in D1, an applied mark written when the re-launch consumes a `resume` attempt, and D7 refusing an applied attempt's key.
  D6's un-pause could override a human's `goto` or `pause`. Fixed: the re-launch applies a `resume` only over the state the arbiter's records account for, otherwise hands on; the orch-goto criterion now hands on instead of re-launching.
  A missing watched marker admitted the arbiter at the batten-owned Publish halt. Fixed: the target-naming Publish halt is refused in every run, since an absent marker cannot be read as unwatched.
  Also fixed: D2's stencil-only bound states its consequence, the Scope cross-reference, a negative `arbiter_timeout_min` is refused, and the batten hold's release paths are in the criteria.
- Round 2 (BLOCKING): the arbiter's own goto cleared a human pause set during the arbiter before D6's state check could see it. Fixed: the arbiter no longer runs `goto`; a `resume` verdict names a goto target, which the re-launch performs under the status lock only after checking the stop key and the pause conditions recorded at spawn.
  D6's un-pause clause rested on a false premise, since `lyx loom resume` writes no status and a step re-calls `current_producer` from `paused` and `awaiting`. Fixed: the clause is gone and the ledger question is resolved by the code.
  Also fixed: D7 hands on an applied attempt with an unchanged stop key without claiming recurrence, and D2's list of Go inputs names the verify-failure record.

## Question ledger

- Who spawns the arbiter? Resolved, auto-pick: the driver, through a loom verb (D1).
- What the arbiter may do itself versus recommend (board Open)? Resolved, auto-pick: three recording verbs, never resume (D3).
- Its stencil and model key (board Open)? Resolved, auto-pick: role `arbiter`, key `arbiter` at `opus[high]` (D8, D11).
- A bound on spawns per run (board Open)? Resolved, operator decision via `ly:orch`: a config budget, then escalation to the parent (D7); key name and default an auto-pick.
- Does a goto-paused run step on a bare loop re-launch? Resolved by the code: a step short-circuits only a `done` run and re-calls `current_producer` from `paused` and `awaiting` alike, and `lyx loom resume` writes no status (D6).
- Zero values of the new keys in pre-existing pairs? Resolved, auto-pick: each zero has a safe meaning (D7, D8).
