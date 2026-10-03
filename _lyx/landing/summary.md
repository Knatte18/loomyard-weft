# Review segments converge on fresh reviews; circling escalates

A review segment now stops only on a judge verdict over a fresh review, and escalates to the operator when it circles.
Before, the Bouncer judge acted as a second reviewer: it read the artifacts and the fixer report, and "every finding fixed" counted as converged although no fresh reviewer had seen the fixes.
The rework-generations run showed this: one round found eight findings, the fixer fixed all of them, and the segment ended with no second review.

## What changes

- **Finding class.**
  Every burler review finding carries a required `class`: `design`, `scope`, `decision` or `consistency`.
  `ParseReview` rejects a missing, empty or unknown class like a bad `severity`.
  The three loom rubrics (Discussion, Plan, Webster) each define `design` for their own catchment.
  `design` is the single gating class, a `burlerengine` constant.
  The fixer still fixes every finding of every class and severity.
- **Judge verdict.**
  The verdict is `CONVERGED`, `CONTINUE` or `CIRCLING`, replacing `APPROVED`/`BLOCKING`.
  `CONVERGED` runs Approve, Commit and returns Done; `CONTINUE` returns Stuck to the round producer; `CIRCLING` returns Awaiting.
- **Facts file.**
  Before each judge spawn, Go writes `round-<N>-facts.md` into the run directory: per-round finding, severity and class counts, the gating count, the trend across rounds, and recurring or reopened ledger keys.
  A round whose review no longer parses gets a parse-error row instead of counts and never fails the judge call.
- **Judge prompt.**
  The judge reads the facts file, the latest review and the previous ledger.
  The artifact paths and the fixer report are off its list, and the reviewer's own top-level `verdict:` is not a convergence signal.
  The prompt states the default rule: `CONTINUE` while the latest fresh round has a gating finding at MEDIUM or worse or any BLOCKING finding; `CONVERGED` when it has neither; `CIRCLING` when such findings persist with no progress across at least two fresh rounds.
  A parse-error row for the latest round means `CONTINUE`.
  The judge may relabel a finding's class or severity only where the review's own entry shows it is mislabeled, names each such finding in its rationale, and never relabels BLOCKING downward.
- **Circling resolution.**
  `lyx loom circling accept [<slug>]` and `lyx loom circling continue [<slug>]` record the operator's decision for the round.
  The verbs refuse unless the run is awaiting at a Bouncer row whose latest verdict is `CIRCLING`, and refuse a second decision for the same round.
  `continue` returns Stuck and counts against `max_bounces`; when the circling round is the last budgeted one, the Awaiting Reason says so and names `lyx loom goto`.
  `accept` settles the segment as `CONVERGED` does.
- **Model key.**
  Loom config gains `judge: sonnet[medium]`.
  Every Bouncer row's seed and judge passes take it; BurlerRound rows keep `review`; a row's own overrides still win.
- **Docs and invariants.**
  `CONSTRAINTS.md`'s Review Round Invariant gains the two lines: convergence only from a judge verdict over a fresh review, and class decides who decides, never whether a finding is fixed.
  The `shedadapters` and `burlerengine` package docs are rewritten, and the completed roadmap entry and `manifest/designs/review-finding-classification.md` are removed.

## Design decisions to check

- **Accept has two recorded states.**
  The verb writes pending; the Bouncer writes settled before the Approve seam, as a `CONVERGED` verdict is on disk before Approve.
  Only a pending accept settles, and a settled accept counts as "settled earlier" at Call entry, so a resume after a failed Commit seam clears and re-seeds instead of passing the same accept twice.
  Getting this wrong lets a reworked artifact pass with no review.
- **Legacy verdicts.**
  A legacy `APPROVED`/`BLOCKING` already on disk at Call entry reads as `CONVERGED`/`CONTINUE` for the clear and replay, so a run dir from the old binary is not re-judged as unsettled.
  A legacy word harvested from a judge spawned or attached in this Call never settles; the round is re-judged under the new prompt.
- **Verbs check no caller identity.**
  The refusal preconditions are the whole bound, like `lyx loom review` and `lyx loom approve`; no agent stencil names the verbs.
- **Seed pass.**
  It never returns Done, and the `Skip` seam is unchanged.
- **Out of scope.**
  `treadleengine` and its judge, the fixer's behavior, reviewer model choice, and Go enforcement of the judge's ledger carry-forward.

## To check by hand

- The integration suite reported `status: OK`; no live loom run with a real sonnet judge or a real circling escalation was done.
  A sonnet[medium] judge may misread a trend; the written default rule, the named-departure requirement and `max_bounces` bound it.
- The cobra-tree test for the circling verbs is `internal/loomcli/circling_integration_test.go`, not `circling_test.go`, because the Test Tier Purity Invariant bars `hubforge.NewHub` from untagged files.
- Stencil changes need a production deploy (`update-plugins.sh`) before the board copies stop drifting from the source; this was not run.
- Reviewers may file real problems under a non-gating class; the fixer still fixes them, and the judge can relabel only from the review's own entry.
