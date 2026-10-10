# Support log: publish-recovery

## Interview

The session ran autonomously, so every turn below is the agent's own exploration and pick; no operator answered.

1. Board read.
   The entry merges five issues (#462, #471, #450, #452, #453) and depends on `gate-test-scope` and `git-robustness`, both done.
   Related entries read: `webster-replan-state` (goto-to-Plan webster state, unstarted), `driver-context` (Go step loop, running), `orch-owns-the-loop`, `run-message-routing` (driver as message router, open brainstorm), `lyx-halt`, `convergence-plan`, `fabric-reset-to-sha`, `finalize-discrepancy-doc`, `config-overlay`.
2. What `gate-test-scope` already landed for #462.
   Publish writes a failure record on a failed plan verify or `publish_verify`; the Webster-Review rubric renders it under `## Publish failure` and widens the range to the merge-in commit; the Webster-Burler round gate runs the failing tests by name in the `tmux` tier.
   So the receiving side exists and the gap is the route: no Stuck reason, spec row or orch rule names it.
3. Route choice for #462.
   `goto` admits any row before the current one, so both `Webster-Bouncer` and `Webster-Burler` are reachable from a blocked Publish, and a goto refreshes the segment's bounce budget.
   `Webster-Burler` was picked: its reviewer half reads the same rubric and raises the failing tests, so the Bouncer's judge turn is spent only after the fix.
4. #471 cause.
   Publish's pre-merge clean check runs ahead of the resolver, and the resolver aborts any in-progress merge before a clean attempt, so the staged leftover is flagged as uncommitted work by a check whose advice would commit an unfinished merge.
   Finalize's merge-in step has the same order.
   The fabric merge record carries `Source`, the caller's branch, so an own-leftover test is possible without a new record.
5. #471 resume path.
   `lyx loom resume` and `start` refuse over any parked or foreign merge with kind `merge_in_progress`; the hub's workaround was `lyx fabric merge --abort` then resume.
   The guard is narrowed only for the two rows that redo the merge-in themselves.
6. #450 cause.
   `ReadConfigChanges` diffs the merge-base of the two records tips against the task tip, so a parent commit that reached the task through a records sync is reported.
   A three-way rule was chosen over a plain parent-tip diff because a parent change the task never synced would otherwise be reported in the other direction.
7. #452 state.
   The incident predates `shuttle-send` (landed 2026-10-08 21:37), which records a lyx-stopped run as `stopped`; the attach probe already treats a terminal outcome as respawn-eligible at any age, and the burler stops halves through that stop.
   The remaining work is a pinning scenario.
8. #453 mechanism.
   Reed exports the strand's full name to each spawned session, `lyx config` already refuses a hub-wide write from a slug-bearing name, and the orch's name is slug-free.
   The plan stencil itself tells the plan agent to run `decision add --by <who>`, so the stencil changes with the verb.
9. Docs and spec surfaces found: `contracts/specs/refusal-spec.md` (landing rows 272 to 276, decision rows 233 to 243, resume row 257), `contracts/stencils/orch/orch-template-role.md` (escalations list), `contracts/stencils/loom/loom-template-plan.md` (step 1), `docs/overview.md` verb lines.

## Rejected alternatives

- Automatic routing from a blocked Publish to the fixer (recipe `on_stuck` on Publish, or a Stuck that carries a target row).
  One `on_stuck` per row would send merge, push and dirty-tree faults, which stay a human's, through a review round before halting; an outcome-dependent edge is a shed engine change out of this task's reach.
- The driver runs the goto itself at a Publish blocked on a verify failure.
  The driver stencil hands every `blocked` back by design, and `driver-context` is rewriting the driver loop; a one-case exception would need the driver to parse reason text.
- `lyx loom reject` accepted at a blocked Publish.
  Needs an open pull request, which a Publish blocked before its push lacks, and routes through PR-Rework's whole generation.
- Goto target `Webster-Bouncer`.
  Spends the judge's turn restating findings the record already names; the Burler's reviewer raises them directly.
- Post-Discussion parent decisions as a Webster-Review focus.
  A decision that changes what the code must do changes the plan; the Plan route exists and `webster-replan-state` is fixing its webster state.
- Continue (resume) the staged merge-in instead of aborting and redoing.
  Contradicts the resolver's contract, which never trusts a session's claim and re-verifies by content from a clean attempt; a resume path would need its own verification of a half-resolved tree.
- Abort any parked merge regardless of source.
  The resolver does this today inside a step, but at the row's entry it could discard an operator's parked merge of another branch; the source check costs one record read.
- Keep the resume and start refusal unchanged and let only the driver's re-step reach the row.
  The hub resumes a blocked run through `lyx loom resume`, so the refusal would still force the hand abort.
- A plain parent-tip diff for the config notice.
  Reports a parent change the task never synced as the task's change.
- A transient class for the young-run-dir attach refusal.
  The driver's transient re-step is immediate and would meet the same refusal; the age rule exists for a run that may still be starting.
- Deny hooks on the plan session for `lyx loom decision add`.
  A deny reads command text per role; the verb's own check covers every spawned role and needs no per-stencil wiring.
- Refuse only `--by parent` from a task session.
  A task session could relabel its call as the operator's.
- Tell the parent from a run agent by a slug-free strand name, the shape of `lyx config`'s hub-wide write check.
  `hubgeom.decideParent` records a slug-bearing parent `<shortname>:<slug>:orch` for a pair created from a task worktree, so the slug rule would refuse the very caller the way forward names; comparing with the recorded parent name drops that premise.
- An exemption for the driver role in D6.
  `run-message-routing` has not decided whether the driver records decisions; granting it now designs for an unbuilt route.

## Review rounds

_No rounds yet._

## Question ledger

- Route target for a blocked Publish: auto-pick `Webster-Burler` over `Webster-Bouncer`; rationale in D1. Resolved.
- Automatic versus hub-run goto: auto-pick hub-run; rationale in Scope and Rejected alternatives. Resolved.
- Abort versus continue a staged merge-in: auto-pick abort and redo, bounded to a parent-sourced fabric record; rationale in D2. Resolved.
- Narrow the resume and start guard: auto-pick yes, for Publish and Finalize over a parent-sourced parked merge only; rationale in D3. Resolved.
- A non-zero verify exit whose failure record write failed: auto-pick no goto, the reason stays as today; rationale in D1. Resolved.
- A conflict session that outlived its dead driver: auto-pick stop it through shuttle's stop verb before D2's abort; rationale in D2 and D3. Resolved.
- A conflict-session stop that fails: auto-pick leave the merge untouched and stop Stuck naming `lyx reed remove <guid>`, as the existing seat and superseded-strand rows do; rationale in D2's Bound. Resolved.
- A named `decision add` caller whose parent resolution errors: auto-pick refuse, naming the error and the operator's unnamed shell, so the guard never fails open; rationale in D6's Bound. Resolved.
- Config notice rule: auto-pick the three-way comparison; rationale in D4. Resolved.
- Whether #452 needs code: auto-pick no, pin the scenario; rationale in D5. Resolved.
- Labels refused from a task session: auto-pick both `parent` and `operator`; rationale in D6. Resolved.
- Driver exemption for `decision add`: auto-pick none; recorded as a bound in D6. Resolved.
- Whether the orch strand name ever carries a slug: the recorded parent of a pair created from a task worktree already does, though no orch runs there until `orch-any-worktree`; D6 compares with the recorded parent name instead of assuming a slug-free one, and the decision record's Open risks carries the dependency. Resolved.
- Environment: `lyx board get` and `lyx board list` worked; no tool failure was met. Nothing outside the two output files was touched.
