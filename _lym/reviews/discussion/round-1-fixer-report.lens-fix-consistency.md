```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, whether the round-1 change set still agrees with the parts of the decision record and support log it touches.

1. LOW — D11, around the new `--discard` bullet. The sibling sentences were not updated:
   - "An amendment is recorded in `scope.json` with who, when and its entries or the restore" leaves out a discard.
   - "Containment reads the sealed expectation and every amendment" no longer holds once `--discard` sets the sealed expectation aside.
   - D7's first classification bullet has the same problem: "expected: covered by an entry of the sealed expectation or an amendment (D11)".
   
   Evidence: D11's new bullet says `--discard` makes containment read "an empty expectation and every amendment", and D13 now lists "entries, restore or discard". The recording sentence and the two containment sentences still describe the two original modes only.

2. LOW — D12's change admits `defer` at a Stuck Drift-Expect. Its twin in Auto-mode assumptions was left as it was: "D12: `defer` is admitted after the writer when the baseline directory is missing." Drift-Expect runs before the writer, so that line now understates where D12 admits `defer`.

3. LOW — D8's rewrite moves the "writer fixes its own regressions" premise to Darn-Write only and sends tapestry findings to Master's fixer paths. D8's Rationale was left unchanged:
   - "The writer meets 'key `dP` of case X moved, outside the expectation' while it can still fix it, up to the entry's attempts."
   - "...so the recipe-level entry reaches tapestry's writer."
   
   In tapestry, Master edits no code and hands the findings to a fixer fork or `fix-verify`, which the rewritten Decision states. The Rationale still presents the writer as the one who fixes.

4. LOW — Support log, new "Round 1" entry: "Twenty-two findings, all fixed". F4 fixes darn concretely but leaves tapestry's code-fixing route to the plan ("In tapestry it is the fix loop the plan settles under the operator's decision 3"). F3 likewise leaves the tapestry wiring to the plan. If the fixer-report records either as deferred, or as fixed differently, this sentence contradicts that report. It also pins a count, which the round's fixer-report already holds.

Tool calls: none. My context already held the review, both files as changed, and the rubric.
