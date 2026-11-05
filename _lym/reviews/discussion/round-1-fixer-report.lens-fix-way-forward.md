```yaml
lens: fix-way-forward
outcome: reported
```

The task was to check that every way forward the fix introduced can be reached from the state it names and leads to progress.

1. **MEDIUM**: D6, "A failed seal check is a finding naming which check failed and its way forward", with the `--restore` and `--discard` bullets.
   - **Summary:** at the writer's gate entry the finding names a verb that nobody in that session can run.
   - **Evidence:** D11 admits `lym drift amend` only to the parent or operator, and only at a Stuck row, and its criterion says it "is refused … at a running row". During Darn-Write the writer cannot run `--restore` or `--discard`, so the entry re-prompts it until its attempts run out. Only that later Stuck reaches the parent. In tapestry, D8 now sends a failed `drift` entry to Master's fixer fork or `fix-verify`. Those are code fixers and cannot run the parent's verb either. The finding's way forward only works once the run is stopped, and the record never says the entry should end at a Stuck right away, or how.

2. **MEDIUM**: D7, "In tapestry it is the fix loop the plan settles under the operator's decision 3, and the reason names it."
   - **Summary:** for tapestry, the code-fixing way forward from a Stuck Drift-Verify or Drift-Recheck has no stated route.
   - **Evidence:** the support log's rejected alternative still says "Webster re-entered after its last batch has no card to run". D8's tapestry path (fixer fork or `fix-verify`) only works while Merriam's run-level gate runs inside Webster, not at a later Stuck row. `lym loom goto` moves a run only back, and it is refused onto a fix row whose review verdict has settled (`internal/loomcli/cli.go` goto Long). So nothing in the record reaches code from a Stuck Drift-Recheck after Code-Burler-Fix. The reason "names it", but "it" does not exist yet. The plan hand-off should at least state that it must provide a route reachable from a Stuck Drift-Verify and Drift-Recheck.

3. **LOW**: D15's new reinterpretation bullet: "the way forward is to set `scope` to `full` or `off` with `lym config drift` and drop `skip`."
   - **Summary:** this sends the operator to `lym config drift` to drop `skip`, which is now a retired key.
   - **Evidence:** D15 says "`lym config reconcile` prunes the removed keys", and `internal/configengine/doc.go:75-81` says retired keys are dropped on load and pruned by reconcile. `lym config drift` edits template units, and `skip` is no longer one. The part of the way forward that drops `skip` should name `lym config reconcile`.

Tool calls:
- One grep of `internal/websterengine/runlevel.go` and `internal/gateslot/config.go`. I needed to know whether a run-level gate failure goes to Master's fixer regardless of which entry failed, and whether gateslot refuses an unknown gate key. My context held neither file.
