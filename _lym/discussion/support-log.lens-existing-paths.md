```yaml
lens: existing-paths
outcome: reported
```

Task: review the Weaver decision record through the existing-paths lens, tracing the code that already reads or writes the state its decisions introduce.

**Findings**

1. **BLOCKING — D1 (reroute route), D9 and D11: after a reroute, plan review can't see Weaver's done cards.**
   - Several existing paths read only `_lym/webster/` and its state:
     - `loomshed.ValidatePlan` (`internal/loomshed/gates.go`) loads `websterengine.LoadState(websterengine.Dir(anchor))` to find done cards and `EditedDoneCards`. It is run by the plan gate and by `internal/loomcli/validate.go`.
     - `loomshed.WebsterRecordNote` (`internal/loomshed/publishfailure.go`, wired at `internal/loomcli/wiring.go`) tells the Plan-Review rubric which cards are frozen.
     - The renumber card verbs (`internal/loomcli/renumber.go`) bound renumbering at the last begun card and take webster's state-mutation lease.
   - Under Weaver, `_lym/webster/` is absent:
     - every card reads as not done, so the done-card freeze and `done-card-edited` stop holding;
     - the rubric note says `none`;
     - renumber can move cards Weaver already built.
   - So a reroute to Plan-Burler-Review after Weaver has run batches lets the plan be rewritten under work that already landed, and D3's "drop the cards already done" then misreads the rewritten plan.
   - The record doesn't say whether these readers become implementer-neutral, for example by reading the active implementer's done-card record through a shared seam. That belongs in D11's shared machinery.

2. **MEDIUM — D9 and D11: Weaver's commit leaves out the plan directory.**
   - Webster's `CommitWebster` (`internal/loomcli/wiring.go`) commits `websterengine.DirRel()` together with `planparser.PlanDirRel()`.
   - The comment there says why: webster rewrites card files during the run, for handle binding, canonicalization and rename rewrites.
   - D11 has Weaver run the same post-batch checks, which rewrite pending cards. D9 commits only `_lym/weaver/`.
   - The rewritten plan files then stay uncommitted. D6's "leaves the tree unclean" check would blame the next attempt, and the worktree's removal can refuse.
   - `planCommitPathspecWith` also stages `_lym/webster/` alongside the plan commit, but nothing does the same for `_lym/weaver/`.

3. **MEDIUM — Scope, D9: a PR rework round never archives Weaver's run directory.**
   - `internal/loomshed/prrework.go` moves `webster/` into `prior-generation/`, through its `ArchiveWebster` seam, "so the next webster run starts fresh".
   - Nothing archives `_lym/weaver/`. The next generation's Weaver would therefore:
     - resume from the old state with every batch done;
     - attach to stale strands or skip work (D12);
     - carry the old journal.
   - The record says nothing about rework generations.

4. **MEDIUM — D10 and D9 (no `summary.md`): Describe loses the flaky-test triage and the verify-fix check items.**
   - `landingshed.DescribeInputs.RunRecordPath` and `PriorRunRecordPaths` (`internal/landingshed/describe.go`) read webster's `summary.md` for manual-check items.
   - `webster-spec.md` says the `## Integration suite triage` and `## Verify gate fixes` sections are carried into the change description for the PR reviewer.
   - D10 has the same flaky rerun and `fix:` commits, but with Q11's choice of no `summary.md`, they never reach the PR description.
   - The record should either keep an equivalent record for Describe or state that it is dropped.

5. **LOW — D2 and D12: `lym loom decision add` notices reach Weaver's batch strands differently from Webster's.**
   - `decisionNoticeInline` and `heldBackReason` (`internal/loomcli/decision.go`) treat only `MerriamStrandRole` as the implementer.
   - A live Weaver batch or fixer strand would get a decision notice without the inline text.
   - In a halted run, a leftover Weaver strand would be woken, which the Merriam guard exists to prevent.
   - The record doesn't say whether a batch strand should receive decisions mid-attempt, or whether only the next attempt's prompt carries them.

6. **LOW — D10: reporting and trace code is keyed to Webster.**
   - `internal/reportengine/timing.go` has `PhaseWebster`, and `failuretrace.Locations` takes `websterengine.VerifyFailure`.
   - Weaver's verify failures and timing go unreported unless D11's sharing covers `VerifyFailure` and the reports learn the phase.
   - The record is silent on this.

**Paths traced**

| Decision | Path | Result |
| --- | --- | --- |
| D1 reroute | `ValidatePlan` | breaks |
| D1 reroute | `WebsterRecordNote` | breaks |
| D1 reroute | renumber verbs | breaks |
| D1 reroute | `planCommitPathspecWith` | `_lym/weaver/` not staged |
| D3 | `ValidatePlan` freeze on a rerouted plan | breaks, follows from finding 1 |
| D9 | `CommitWebster` pathspec | breaks: plan directory omitted |
| D9 | PR rework archive | breaks |
| D9 | Describe `RunRecordPath` | Weaver passes no `summary.md`, which is allowed, but the triage is lost |
| D10 | Describe verify-fix items | breaks |
| D10 | `failuretrace` and `reportengine` phase | unreported |
| D10 | drift entry (`driftengine` doc on a re-entered Webster) | survives, since D10 runs the row's gates |
| D12 | decision notices to a halted run's leftover strand | breaks: no guard for Weaver strands |
| D5 | config registry, a new module | survives |
| D1 | `implementerEntry` table | survives |

**Tool calls**

1. Searched for packages outside webster that reference `websterengine` or `_lym/webster`: my context did not list who reads webster's state.
2. Searched loom, landing, drift and fabric code for `summary.md`, `Dir` and `Merriam` uses: to find the specific readers.
3. Read `wiring.go`, `gates.go`, `publishfailure.go`, `prrework.go`, `renumber.go` and `decision.go`: to see how each reader uses webster's state.
4. Read `describe.go`, `prrework.go`'s archive seam, `reportengine`, `failuretrace` and the callers of `ValidatePlan`: to confirm the Describe and rework paths and who calls the plan validation.
