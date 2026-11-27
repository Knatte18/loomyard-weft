---
round: 1
focus:
  - "D3 with D1, D4 and D7: trace a refused target after it leaves the ledger while its branch is still ahead of origin. Settle what the dobby's ahead check and the pusher then do with it, what clears the refusal, and what the all-clear and the `sync:` field report meanwhile."
  - "D5 with D6: walk a local landing end to end with origin unreachable, including the lagging-parent fast-forward, the owner check and every other step that reads origin or GitHub. Settle whether the Goal's and the acceptance criterion's claim that the landing returns done holds, and what the fast-forward does when origin cannot be read."
  - "D5 ordering and recovery: settle what happens on a crash or resume between the merge and the owed record, what reads the board `done` or the origin parent while the landing push is owed or refused, and who ends a refused landing push once Finalize has returned done."
  - "D7 bound: settle whether reachability from a local branch or the code branch's reflog separates lym's commits from hand commits, including a hand commit that was pulled or merged locally and reflog expiry, whether the git behaviour relied on is pinned, which push paths `every other push path` binds, and what way forward a refused reset has, the takeover case included."
  - "D6 and D7 classification: D6 re-steps without a count cap. Settle which permanent failures (permission denied, protected branch or hook decline, missing repository, GitHub 403 or 404, rate limit) could be read as `git-transport`, `github-api` or owed, and whether the record decides each."
  - "D1 coverage: settle what starts or wakes the pusher for each owed source (plain `git commit`, a beam-branch merge, a branch with no origin tracking ref, a diverged branch), and whether every existing push path is either moved to the pusher or named as keeping its own push."
  - "D1, D2, D3, D4, D13 attack surface: for each new target kind, the push-now mark, the `lym fabric sync-status` verb and the pair left standing, settle what it can skip or let through and what bounds it. Include cancel, hand-over and removal dropping a ledger entry whose commits are still only local."
  - "D9: the removal deletes files. Settle whether git can evaluate ignore rules from the first parent's tree as stated, whether that behaviour is pinned (nested ignore files, negations, exclude files outside the tree, symlinks and nested repositories), and whether `again on every resume` can remove files the bound does not intend."
  - "Open risks: for each line, settle whether it is a question only the parent can answer or a choice the writer can make, and check the entry on the board against Scope Out and D3/D4."
  - "Premises and board fit: check the root-cause statements in Constraints and the claims in D10 and D12 against the code and issues, check the board for entries or decision notes that overlap or contradict (`fabric-core-push-origin`, `pr-modes-no-agent` among them), and settle whether any fact the design rests on is left unsettled."
  - "Acceptance criteria: settle whether each observed fault in Goal has a test or a stated reason it has none, whether D11 names its findings and tests precisely enough to check, and whether every doc statement the change must make true, PATTERN backgrounds included, is named."
---

Round 1 weighs the failure and recovery paths of the design (refused, owed, resumed, taken over) over its happy path,
since most Decisions interact through the ledger and the removal of synchronous pushes.
