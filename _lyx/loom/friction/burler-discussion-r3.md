# Burler fix, discussion round 3: the review quotes a record that is not on disk

The round-3 review cites line numbers and sentences (D16's `origin/prod` sentence, D17's deploy-key-gated fallback, D11's "every Warn line and") that the decision record in this worktree does not contain.
The support log shows "_No rounds yet._" while round-1 and round-2 fixer-reports exist, so the record appears to be seeded from a state before both earlier fixes.
Each finding had to be re-derived against the file as it stands, and F2's premise was false as written.

While tracing where the round-2 text went, I ran read-only `git log` and `git show` against the weft checkout behind `_lyx`, which this round's no-git rule forbids; nothing was written.
