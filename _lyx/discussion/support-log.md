# Support log — orch-wake-policy

## Interview

The session ran autonomously: every question below was settled by the writer as an auto-pick (see `## Question ledger`).

1. Read the board entry and the related entries `orch-split`, `orch-continue`, `alignment-agent`, `config-layers-run-settings`, `burler-split-budget`, `parent-review-wait`, `runs-overview`, `merge-review`, `convergence-plan` and `lyx-cleanup`.
2. Read the orch role stencil and the newest orch note in the prime, which lists the landing routine (pull, `update-plugins.sh`, reconcile, intake, prune, teardown check, fabric commit, start cleared tasks) and the `driver_not_parked` retry loop.
3. Explored batten (`battenshed` doc, `batten-recipe.yaml`, the awaiting arm), landing (`landingshed` doc, Publish failure record and way-forward), `hubreconcile`, the transient classification (`gitexec`, `githubclient`, `shedtransient`) and fabric's `pushretry.go`.
   Found that batten already retries an approval resume while the driver is not parked, that `hubreconcile.Ensure` already reconciles after a binary change, and that the shedverbs loop re-steps a transient once.
4. Settled the design: a Land row between Run-Shed and Worktree-Teardown, a hub-wide `deploy` key, notices only on land/teardown failure and newly ready tasks, one automatic Publish verify retry keyed by the failure record, "cannot lock ref" in fabric's push retry, and orch stencil and `CLAUDE.md` moves.
5. Lens review by forks: all six returned. Findings, duplicates merged:
   - scenarios BLOCKING, ready set lost after prune on restart or concurrent landing: fixed, D1 steps 4–6 persist a pre-prune snapshot; two criteria added.
   - scenarios BLOCKING, restart mid-retry: fixed, D5 marker records `started`/`goto-done`/`resumed` and a restart finishes it; criterion added.
   - scenarios MEDIUM, goto refused under the run lock: fixed, D5 retries the goto too.
   - scenarios + premises MEDIUM, stale failure record triggers a retry: fixed, D5 matches the record's HEAD and write time to the halt.
   - scenarios + premises LOW, prime lock held over the deploy: fixed, D1 uses a separate land lock and takes the prime lock only around the pull.
   - scenarios + premises LOW, parent-review approval path in D7: fixed, D7 covers it.
   - scenarios MEDIUM, Notify seam for the new rows and dropped notices: fixed, D4 states the seam and what a drop leaves.
   - scenarios MEDIUM, missing must-cover scenarios: fixed, criteria added for dirty prime, restarts, lock-refused goto, stale record, in-flight status.
   - scenarios LOW, dirty prime halts at deploy: fixed, D1 step 1 checks before the pull; goal-scope LOW and leanness MEDIUM on the same point: fixed, dirty-prime behaviour moved into D1 with a ten-minute wait.
   - scenarios LOW, batten killed between Done and Land: fixed, Open risks.
   - scenarios LOW, "(failed)" retry widens too: fixed, D6 bounds both forms.
   - premises MEDIUM, wrong push sites: fixed, D6 names the gitrepo push primitives, the archive tag push and the detached sync child.
   - premises LOW, exit wait is minutes not seconds: fixed, Auto-mode assumptions.
   - premises LOW, reconcile stamp flip-flop: fixed, Open risks.
   - premises + pattern-fit LOW, resume is not a shedverbs body: fixed, D5 resumes through the Spawn seam.
   - premises + goal-scope + leanness LOW, intake latency: fixed, D8 takes intake at any wake; Open risks.
   - pattern-fit MEDIUM, refusal-spec row and PATTERN-refusal-way-forward: fixed, D9 and Constraints.
   - pattern-fit MEDIUM, battenshed import allowlist: fixed, D1 states every effect is a seam and the allowlist does not grow.
   - pattern-fit LOW, push-both-sides for D6 and the prime pull: fixed, D1 step 2 and D6.
   - goal-scope MEDIUM, ready notice departs from the agreed wake list: fixed, D4 says it is temporary and replaced by `orch-split`.
   - goal-scope MEDIUM, nobody deploys while `deploy` is empty: fixed, D8 and the first Auto-mode assumption.
   - goal-scope LOW, amending superseded entries before landing: fixed, D8 states why it is acceptable.
   - goal-scope LOW, Goal overclaims: fixed, Goal reworded.
   - board-fit MEDIUM, concurrent edits with `config-layers-run-settings`: fixed, Open risks.
   - board-fit MEDIUM, PR-Gate may leave the orch (`alignment-agent`, `merge-review`): fixed, Open risks.
   - board-fit MEDIUM, `burler-split-budget` rename makes the target stale: fixed, D5 does not retry a refused goto; Open risks.
   - board-fit LOW, prune vs `lyx-cleanup`: rejected, both call the same idempotent board prune, and `lyx-cleanup` is an unbuilt note that can call it too.
   - board-fit LOW, reuse the board's readiness: rejected, the board's layer treats a done dependency differently from the stored `depends_on` the snapshot diff needs; D1 defines the predicate once.
   - board-fit LOW, convergence log of the auto-retry: fixed, D5 logs it and adds a batten history entry.
   - board-fit LOW, `runs-overview` notice log: rejected, that log is unbuilt.
   - board-fit LOW, `parent-review-wait` also edits the orch stencil: rejected, no shared section.
   - leanness LOW, Constraints restate Decision citations: fixed, duplicates dropped.
   - leanness LOW, Open risks 1–2 are not new: fixed, binary-swap bullet cut; prune-everything folded into D1's concurrency reasoning.
   - leanness LOW, Auto-mode bullet 3 repeats D1: fixed, reworded to the timing assumption only.
   - leanness LOW, Notes bullet 2 repeats D5: fixed, trimmed to file names.
   - leanness LOW, D3's third sentence: rejected, it stops the plan writer from re-adding a reconcile step.

## Rejected alternatives

- Land steps after Worktree-Teardown: the deploy would not wait for the driver's exit grace, but a failure would leave no pair to inspect and Worktree-Teardown would stop being the terminal.
- Land steps inside Run-Shed's done arm: mixes the child wait with prime writes and a new failure class into a row whose Stucks are all budget-exempt self-routes.
- Running `lyx config reconcile --apply` in the Land row: the batten process is the old binary and would reconcile against old templates; `hubreconcile.Ensure` already owns this (PATTERN-binary-change-reconcile).
- Batten importing inbox issues mechanically: triage (labels, fold or new note) needs judgement.
- A notice whenever the inbox holds issues: every run's Friction-Reflect files issues, so it would wake the orch at nearly every landing, defeating the goal.
- Batten starting newly ready tasks itself: that is `orch-split`'s mechanical run driver; doing it here duplicates that entry.
- Routing Publish's `on_stuck` to Webster-Burler in the loom recipe: Publish's Stuck covers dirty trees, push rejections and remote rules too, and shed routes every Stuck of a row to one target.
- Matching the verify-failure way-forward text in the child's error: drivers and batten read structured fields, never error text.
- Classifying "cannot lock ref" as a git transport failure in `gitexec`: it is a rejected push, not an unreached remote, and `IsTransportFailure`'s contract excludes rejected pushes; fabric's push retry already handles the analogous "(failed)" refusal.
- A fixed `update-plugins.sh` call in Go: lyx drives other repositories, which have no such script.
- Holding the prime lock over pull and deploy: a deploy near ten minutes would make a waiting Worktree-Create or Worktree-Teardown go Stuck.
- Halting at once on a dirty prime: a prime agent mid-commit would turn a landing into an orch wake.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where do the land steps run? Recommended and picked: a new batten row between Run-Shed and Worktree-Teardown, under the prime lock. Resolved.
- Q2 (auto-pick): How is the deploy command configured? Picked: hub-wide `landing.yaml` `deploy`, empty by default. Resolved.
- Q3 (auto-pick): Does the Land row reconcile config? Picked: no, `hubreconcile.Ensure` at the next start verb does. Resolved.
- Q4 (auto-pick): Who takes the inbox and amends superseded entries? Picked: the orch at PR-Gate after approving. Resolved.
- Q5 (auto-pick): Who starts tasks a landing made ready? Picked: the orch, on a ready-task notice, until `orch-split` lands. Resolved.
- Q6 (auto-pick): How does batten know a Publish halt is a verify failure and where to send it? Picked: the Publish failure record, with a recipe-written target field. Resolved.
- Q7 (auto-pick): How is "cannot lock ref" retried? Picked: fabric's bounded push retry, extended to every fabric push. Resolved.
- Q8 (auto-pick): Does the approval path need code? Picked: no, the awaiting arm already retries; the stencil retires the orch's loop and a test pins it. Resolved.
- Q9 (auto-pick): Who sets the hub's `deploy` value? Picked: the orch, from the prime, after this task lands. Resolved; listed under Auto-mode assumptions.
- Q10 (auto-pick, after lens review): How are landings serialized without starving pair creation? Picked: a hub land lock over the row, the prime lock only around the pull. Resolved.
- Q11 (auto-pick, after lens review): What does a dirty prime do? Picked: wait up to ten minutes, then Stuck with a notice. Resolved.
