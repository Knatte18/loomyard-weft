```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that every refusal, step, row or test this fix introduced can be reached and makes progress.

**MEDIUM: the `decision add` way forward for an uncommitted or changed record strands a run that is past Record-Write**
- Changed text: D4, "The first two name `lym loom goto --to Record-Write` then `lym loom step`, so Record-Write commits the pair", and the matching D4 acceptance row.
- Problem: the step commits the record, but the run's current row is then the one after Record-Write, which is Drift-Baseline. No goto back to the row the run was on is named.
- Evidence: on a run that is at Darn-Write or later, the next `lym loom resume` passes Drift-Baseline after code was committed. D4's own in-flight bullet forbids this route: "a route that never passes Drift-Baseline after the writer committed code". It ends that route with `goto --to Darn-Write` before `resume`, and this new route has no such step.

**MEDIUM: Drift-Baseline's Stuck branch never reaches Darn-Write's refusal**
- Changed text: D4, the Drift-Baseline bullet: "may record a scope from the board entry alone, which then stands, or return Stuck with drift's ways forward, and in either case Darn-Write's missing-record refusal below is what sends the run back to Record-Write". The added Open risks line says the same.
- Problem: on the Stuck branch the run halts at Drift-Baseline and never reaches Darn-Write. Drift's ways forward end in `lym loom resume`, which spawns the baseline agent again on the same missing record, so the loop makes no progress.
- Evidence: `internal/driftengine/doc.go:68-71`. A Done agent with no scope record is a Stuck, and capture refusals name `lym drift defer` or `lym drift capture` with `lym loom resume`. None of these names `goto --to Record-Write`.

**MEDIUM: the stencil's no-parent stop report still names the note route that D6 now says loops**
- Changed text: D6, "a `Decision for &lt;slug&gt;:` note does not settle a stop … The darn writer stencil drops its route of writing a note's answer into the record", and the matching D6 acceptance row.
- Problem: the fix drops only the step where the writer copies a note into the record. The stop report still tells the operator to write a note. Under the new D6 that halts again, and the halt's reason, which is that description, then contradicts the halt's own way forward (`decision add` then `resume`).
- Evidence: `contracts/stencils/darn/darn-template-write.md:48` writes the stop report with the way forward "a `Decision for {{.slug}}:` board note that answers it and then `lym loom resume`". Lines 49-50 hold the re-prompt-and-read-note route. D6 says the description is the halt's reason, and nothing in D6 retargets that stop-report line.

**MEDIUM: the "ignored" temporary file is committed on resume**
- Changed text: D4, "writing each to a sibling temporary file and renaming it into place, so … a temporary file a crash left behind is ignored", and the D4 acceptance row "ignores it and writes the file whole".
- Problem: the resume step's commit stages the torn temporary file with the pair.
- Evidence: Record-Write commits through the discussion commit. `discussionCommitPathspec` (`internal/loomcli/wiring.go:324-328`) stages the whole discussion directory with `git add -- &lt;dir&gt;`, so a sibling temporary file left in that directory lands in the commit. The decision does not say to remove the temporary file or to put it outside that directory.

**LOW: the missing-historical-entry way forward leads straight into a second refusal**
- Changed text: D4, "the third names restoring the dropped entry from the record's committed history and re-running the verb".
- Problem: following it works only in two hops, and the first re-run is refused with a different way forward.
- Evidence: `checkDarnRecord` (`internal/loomcli/decision.go:383-394`) checks "differs from its committed copy" before the history check. Restoring the entry in the working copy makes the working copy differ from HEAD, so the re-run is refused as a differing record and sent through `goto --to Record-Write` then `step`. That route also carries the first finding's missing goto back.

Tool calls:
1. Grepped `internal/loomcli/wiring.go` and `internal/loomshed` for the discussion commit's staging, to check what a resume commits; my context did not show what the commit stages.
2. Read `internal/loomcli/wiring.go:320-340` (`discussionCommitPathspec`) to confirm the commit stages the whole directory.
