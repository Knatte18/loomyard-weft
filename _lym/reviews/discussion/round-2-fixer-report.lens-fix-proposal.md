```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out (F13, F14, F15) through the fix-proposal lens.

Findings:

1. MEDIUM. Changed text: D5's new sentence "A motif run commits no work to its task branch, so its teardown deletes the branch locally and on origin, and the start that follows the clear meets no leftover branch". The same claim is in the new Constraints Fact about `remove.go`.
   - Summary: the guarantee holds only when Worktree-Teardown reaches Remove. The proposal states it with no condition.
   - Concrete case: the pair is not level when the teardown reads it. `worktreeTeardownProducer.Call` then ends the row Done with the pair standing, "to be removed later" (`internal/battenshed/teardown.go:131-133`, and the doc at lines 68-70).
   - Result: the motif run is done, `startClearsMotifRun` clears it, and the jacquard start still meets the standing pair and its task branch. Worktree-Create then halts with `createRefusal`'s leftover-branch way forward (`internal/battencli/wire.go:371-394`).
   - Evidence: `teardown.go:131-133`, `if !level { ... return "", nil }`.
   - Suggested fix: make the sentence say the branch is deleted once the teardown removes the pair, and that a pair left standing meets Worktree-Create's leftover-branch refusal, whose way forward is `lym batten cancel &lt;slug&gt; --remove`.

2. LOW. Changed text: D4's new non-batten jacquard way forward. It re-seeds the run as tapestry, then `lym loom goto --to Discussion-Write` and `lym loom start`.
   - Summary: the run that follows this way forward still counts as a jacquard run in the cost cells, although it ran Discussion.
   - Evidence: `seedAndCommitBootstrap` writes the run's usage record with `seedRecipe` on the first seed only, and "A re-entry keeps the record already there" (`internal/loomcli/sharedbootstrap.go:147-149`). That record holds recipe `jacquard`, so D9's cost cells file the run under jacquard. D9's bound says only tapestry runs with `stored_record` true are re-keyed, and this run's cost includes Discussion.
   - Suggested fix: D9's bound could name this case beside the rare one it already accepts.

F15's proposal: I found no case where it is wrong. For a shipped profile left as shipped, the derived block equals the template's block, because D7 line 143 defines the template's jacquard block as its tapestry block minus `loom.discussion`. The `defaults.jacquard` case also subsumes the old dangling-default drop.

Further tool calls: none.
