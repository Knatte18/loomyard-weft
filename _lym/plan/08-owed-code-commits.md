# Card 8 — owed-code-commits

**Edit:**
- `internal/fabricengine#RunCommitsAhead`
- `internal/fabricengine_test#TestRunCommitsAhead`

**Uses:**
- `internal/pushledger#HeldRecordsCommit`

**Intent:** D1: a pair target is owed whenever either branch is ahead of its origin tracking ref, whatever made the commits.
`RunCommitsAhead` reads a pair's code branch with `readLocalBranch` and reports it ahead when its tip holds a commit its tracking ref lacks, or it has no upstream and holds a commit,
in place of today's read of only the tip the records tip's `Warp-SHA` trailer names.
A plain `git commit` past the last transition's named tip and a landing's merge on a beam branch therefore owe a push.
The prime still reads its records branch alone, since its code branch is pushed only by the landing target (card 11), to a tip a landing recorded.
A code branch whose tracking ref is pruned reads level, as today.
For a pair whose run's records are held, through `pushledger.HeldRecordsCommit` of its slug, it counts the records branch only up to `<hold>^`, the most any push may send (card 9),
so a pair ahead only by held commits reads level for both of its readers, the pusher's `Ahead` and the starter's `ifAhead` read, and neither the dobby's tick nor its pusher marks it owed again.
The doc comment drops the sentences about the named tip.

`TestRunCommitsAhead`, tier1, gains a row: a pair whose records tip names the code tip, then a plain `git commit` on the code branch, reads ahead, where the merge base reads level;
the row "code commits past the named tip are not counted" is replaced by it, and the prime row with an unpushed code commit still reads level;
a pair whose records branch is ahead only by a commit a landing entry holds, written through `pushledger.Update`, reads level, and ahead once a commit below the hold is added.

**ImpactSummary:** Widens one local read the pusher and its starts call, and stops it counting held records commits; the prime's answer is unchanged.
