# Card 8 — owed-code-commits

**Edit:**
- `internal/fabricengine#RunCommitsAhead`
- `internal/fabricengine_test#TestRunCommitsAhead`

**Uses:**
- `internal/fabricengine/doc.go`

**Intent:** D1: a pair target is owed whenever either branch is ahead of its origin tracking ref, whatever made the commits.
`RunCommitsAhead` reads a pair's code branch with `readLocalBranch` and reports it ahead when its tip holds a commit its tracking ref lacks, or it has no upstream and holds a commit,
in place of today's read of only the tip the records tip's `Warp-SHA` trailer names.
A plain `git commit` past the last transition's named tip and a landing's merge on a beam branch therefore owe a push.
The prime still reads its records branch alone, since its code branch is pushed only by the landing target (card 11), to a tip a landing recorded.
A code branch whose tracking ref is pruned reads level, as today.
The doc comment drops the sentences about the named tip.

`TestRunCommitsAhead`, tier1, gains a row: a pair whose records tip names the code tip, then a plain `git commit` on the code branch, reads ahead, where the merge base reads level;
the row "code commits past the named tip are not counted" is replaced by it, and the prime row with an unpushed code commit still reads level.

**ImpactSummary:** Widens one local read the pusher and its starts call; the prime's answer is unchanged.
