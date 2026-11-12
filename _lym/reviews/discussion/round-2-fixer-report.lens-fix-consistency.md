```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 change set agrees with the sibling decisions, criteria, constraints and cross-references it touches (fix-consistency lens).

1. **LOW.** The D4 push criterion and the D4 decision text describe a held-back push differently.
   - Changed text: the new D4 acceptance criterion, "a testmap call in a task whose entry the run-scope origin reconcile reads as ended, with the parent lacking its work, commits its maps and pushes no code branch."
   - Evidence: D4 says "when the verdict holds the push back, the verb pushes nothing and its commit waits for the pair's next push."
   - The criterion only pins "no code branch", which is weaker than D4's "pushes nothing". A records-only push in a task pair would pass the criterion even though D4 and the `PATTERN-pair-remote` constraint rule it out.

2. **LOW.** D5's new commit in the weft pull is a map commit that the owner-machine rule does not cover.
   - Changed text: D5, "Fabric's weft pull first commits every uncommitted path under `_lym/testcover/` through D4's scoped weft commit…"
   - Evidence: the PATTERN changes line says "committing coverage maps … only on the entry's owner machine". D6 places its refusal in `lym gate testmap` only.
   - The pull's commit runs on any machine's sync, and neither D5, D6 nor the PATTERN changes line says how it squares with owner-only commits. Its premise is that a non-owner never has map dirt, and nothing states it.

3. **LOW.** D7's two sentences about the package job now describe different mechanisms.
   - Changed text: D7, "A package job measuring tests into a map the pass left unwritten starts from the plan's map at the pass's HEAD … never the file on disk."
   - Evidence: the unchanged D7 line 80 still names "the guard of a package job that runs tests into the planned map (`internal/gatecli/testmappackage.go`)" as a reader that reads the planned HEAD. Today that guard reads the file on disk (`testmappackage.go:81-87`).
   - If the job never starts from the file, that guard's comparison of the on-disk commit no longer has a role. The sibling sentence was not reconciled with the new one.

4. **LOW.** D1 lists the audit as a reader that resolves the maps through the anchor, but the audit runs inside the writer.
   - Changed text: D1, "The readers, the gate's coverage scope, `lym gate redundant` and the audit, resolve the maps through the worktree's anchor; the writer, `lym gate testmap`, writes into the weft worktree's `_lym` path…"
   - Evidence: the audit is the closing round of `lym gate testmap` (`testmap.go` `mapPackages` → `r.audit`). D7 also has the audit write the planned map at HEAD.
   - So the audit is part of the writer, and putting it among the anchor readers contradicts D4's rule that the verb never goes through the junction.

5. **LOW.** The new `PATTERN-owner-machine` constraint bullet has no matching PATTERN change.
   - Changed text: the `PATTERN-owner-machine` constraint bullet, "D4's push in a task pair runs the run-scope origin reconcile first, as a loom run's push does; see PATTERN changes."
   - Evidence: the PATTERN changes line adds only "committing coverage maps" to the opening rule. The pattern's reconcile clause still reads "a loom push reconciles run-scope origin first" (`PATTERN-owner-machine.md:7`, `:14`).
   - The bullet points to PATTERN changes for a rule extension that the PATTERN changes line does not carry.

I made no further tool calls.
