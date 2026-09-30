---
round: 1
focus:
  - "Card 2's no-checkout git sequence (worktree add --no-checkout, read-tree HEAD, ls-files via PathTracked, rm -r --cached, checkout-index -a, read-tree HEAD): does it leave a weft worktree equivalent to today's full checkout outside the run-records root (file modes, symlinks, attribute filters, index stat state), and does CommitWeftPaths' StageAndCommit, handed RunsRootRel for a root that is tracked but absent on disk, actually stage those deletions in the one origin commit?"
  - "Card 2 against the decision record's Constraints: are the new raw git calls in forkrecords.go consistent with the Fabric Git Invariant, and does an index-only `git rm --cached` stay outside the Fabric Destruction Chokepoint as the Shared Decision claims (cmd/lyx/destructiveguard_test.go's banned-token list is the mechanical side)? Is the SkipGit behaviour the card states, with deletions left unstaged, consistent with the 'no run records on disk' constraint and with Decision 5's single commit?"
  - "Card 3's targets against its Intent: the Intent edits the deps.go file header and the create/teardown Call doc comments, but the card targets only the PrimeLock, worktreeCreateProducer.Call and worktreeTeardownProducer.Call symbols plus doc.go. Does the target list cover every file the card says it changes?"
  - "Card 3 fidelity to Decision 7: the bound is attempt-counted (bound / poll-interval sleeps) while Decision 7 says 'until a bound elapses', and the Stuck reason claims 'after waiting 10m0s'. Does the attempt-counted wait still honour the decision and keep the reason truthful? Do the listed tests cover the acceptance criterion that the release runs on every exit after acquisition, for both producers?"
  - "Card 4's granularity and grounding: is a test-only card, pinning behaviour card 2 introduces and already covers with its own integration test, an independently reviewable unit, or should it be bundled into card 2? Are the fixture seams it relies on (fabricengine.CommitAnchoredPaths on prime's pair, shedrun.RecipeBatten, Env.InnerRun.ResolveStatus/ReadStatus reading the child's location) real, and do they exercise what the decision record's two battencli acceptance criteria describe?"
---

Card 2 carries the whole fix, so most of round 1's attention goes to the correctness of its git mechanism and to the constraints the decision record attaches to it.
Card 3 is next, on where its targets and its bound semantics diverge from its Intent and from Decision 7.
Card 4 is a granularity and grounding question.

Checked and left out:
The other PrimeLock fixtures (battenrecipe, shedrecipe, shedbuild, battencli run_test) always acquire, so a nil `Sleep` never makes them wait in real time, and card 3's ImpactSummary holds.
`teardown_test.go`'s contention test shares `create_test.go`'s `fakePrimeLock`, which card 3 gives a no-op `Sleep`.
The verify section runs every targeted package, integration tag included.
Missing `Uses` entries are excluded by the rubric's Do-not-flag list.
