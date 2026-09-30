# Support log — verify-triage

## Interview

Autonomous session: no operator answered; every decision below is a self-pick recorded in the question ledger.

1. **Task read.**
   Board task 21 asks for rerun → baseline comparison → halt only on regression, plus failing test names and output tails in `integration.yaml`.
   Trigger: run `ide-hub-workspace` halted twice on a flaky fabricengine test the task did not touch.
2. **Current flow established.**
   Master forks one integration fork; the fork runs the plan's `## verify:` once and writes `status: OK | FAILED`.
   On FAILED, Master ends `stuck`, and `runIntegrationStage` bisects in-process and escalates;
   a Master `done` over FAILED is a loud error.
   The fork's output is never seen by Go.
3. **Where triage runs.**
   Three candidates: Go after Master's session, a new verb Master calls, or the integration fork running a verb.
   Picked Go after Master's session (D1): no Bash-tool timeout, existing detached-checkout and restore machinery, existing unleased-phase split.
4. **How the outcome reaches the driver.**
   With Go deciding after Master exits, either Go promotes a Master `stuck` to `done`, or Master always ends `done` and Go demotes on regression.
   Picked demotion (D2): a `done` run already has a validated `summary.md`; a `stuck` one may not.
   Added D3 so an old-stencil Master that still ends `stuck` does not skip triage.
5. **Failure names.**
   The task wants names and tails in the report.
   The fork is an LLM; Go parsing is deterministic.
   Picked: fork redirects output to a scratch log, Go parses (D4, D5).
6. **Rerun granularity.**
   The task says "rerun the failing tests".
   A targeted `go test -run` rerun needs the right flags and build tags from whichever `&&` link failed, and a dropped `-tags` makes `-run` match nothing and pass — a false flaky that masks a regression.
   Picked a whole-command rerun (D6); plan verifies are targeted suites, and bisect already pays whole runs.
7. **Baseline.**
   "Parent commit" read as the tree the plan started from: first parent of the first card SHA (D7).
   The parent branch's head could have moved and would answer a different question.
8. **Ambiguity policy.**
   Every case triage cannot prove — test absent at baseline, chain stopped early at baseline, opaque failure, no bisector — falls to halting (D7, D8).
   Triage can only remove halts it can prove unwarranted.
9. **Bisect interaction.**
   With a pre-existing red present, the exit-code predicate fails at every SHA and blames card 1.
   Picked an identity-based predicate on regression (D9).
10. **Recording.**
    The Shed adapter forwards only the summary pointer on `Done`, so `RunResult` warnings alone would not reach the operator in a loom run.
    Picked `summary.md` section + warning + friction note + log line (D11).

## Rejected alternatives

- **Master calls a new `lyx webster triage-integration` verb.**
  Master's foreground Bash calls are capped at ten minutes and whole-verify reruns can exceed it;
  it also adds a verb, help-tree tests, and a Master-side decision to a mechanical check.
- **The integration fork runs triage.**
  Same Bash cap, and the fork's prompt would have to drop its "never run lyx webster" guard, which exists because a fork once ran Master's poll loop and deadlocked.
- **Go promotes a Master `stuck` to `done` after a non-regression triage.**
  Requires Master to write a done-quality `summary.md` on a `stuck` outcome and re-running the run-exit audit on promotion; outcome.yaml would contradict the result.
- **Targeted `go test -run` reruns at head and baseline.**
  Flag and build-tag attribution across `&&` links is unreliable, and a lost tag produces a silent pass — the one error triage must never make.
- **Fork lists failing tests itself in the report.**
  LLM extraction from long output can omit or invent names; Go parsing is deterministic.
- **Full log stored in `_lyx/webster/reports/`.**
  `_lyx` is committed; a full test log is ephemeral diagnosis material and belongs in `.lyx` scratch.
- **Baseline = parent branch's current head.**
  The parent may have moved since the task branched, so a failure there says nothing about whether this task caused it.
- **Classify an opaque failure pre-existing when the baseline is also red.**
  The baseline can be red for a different reason; with no identity to compare, excusing it could mask a regression.
- **A separate `internal/gotestout` package for the parser.**
  One consumer; YAGNI.
- **More than one rerun, or a persistent flaky-test registry.**
  Not asked for; the task specifies one rerun.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Where does triage run?** Resolved, auto-pick: Go, in `runIntegrationStage` after Master's session (D1).
- **Q2 — How does a non-regression verdict become a continuing run?** Resolved, auto-pick: Master always ends `done` after the integration report lands; Go demotes to `stuck` only on regression (D2).
- **Q3 — What if Master still ends `stuck` over FAILED?** Resolved, auto-pick: triage runs anyway; a non-regression never promotes `stuck` (D3).
- **Q4 — Who extracts failing test names?** Resolved, auto-pick: Go parses a log the fork writes to `.lyx` scratch (D4).
- **Q5 — Identity granularity?** Resolved, auto-pick: package + top-level test; package-only for build/setup/panic failures; one opaque identity otherwise (D5).
- **Q6 — Rerun the failing tests or the whole command?** Resolved, auto-pick: whole command, once (D6).
- **Q7 — Which commit is "the parent"?** Resolved, auto-pick: first parent of the first accumulated card SHA (D7).
- **Q8 — How to classify a test absent or not run at baseline?** Resolved, auto-pick: regression (D7).
- **Q9 — Opaque failure red at baseline?** Resolved, auto-pick: still a regression (D7).
- **Q10 — No fabric repo?** Resolved, auto-pick: rerun only; red rerun halts with warnings (D8).
- **Q11 — Bisect predicate?** Resolved, auto-pick: regressing identities, exit code only for opaque (D9).
- **Q12 — Report shape and who writes the added fields?** Resolved, auto-pick: Go appends `failures` and `triage` under the lease (D10).
- **Q13 — Do pre-existing failures also go to friction?** Resolved, auto-pick: yes, same note as flaky (D11).
- **Q14 — Tail length?** Resolved, auto-pick: 40 lines per failure (D5).
- **Environment:** no tool failures observed during exploration.
