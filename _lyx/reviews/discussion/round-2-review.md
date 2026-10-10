---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 (ForceRefresh sentence), D2
    summary: ForceRefresh stamps the running binary's revision whatever its channel or modified state, so a sync from a dev, task-branch or modified build recreates the frozen-copy state D2 exists to prevent, with no bound or Open risk
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3, D5 (signature and Bound)
    summary: The drop signature matches a pre-change binary's lyx stencil sync commit, which today commits under lyx seed stencils and lyx seed specs, so the claims that the signature never matches an operator's forced refresh and that every dropped commit is regenerated do not hold during the transition
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 last paragraph, Open risks third bullet
    summary: A hub-wide config commit whose push conflicts cannot be landed by a board push, since that push runs the same pull --rebase and D5 never drops the commit, yet D4 and Open risks name a board push as a way it clears
---

### [MEDIUM] A forced refresh from a non-production binary freezes the copy for production binaries

**Location:** `_lyx/discussion/decision-record.md` D1, the sentence beginning "`ForceRefresh`, behind `lyx stencil sync`", read against D2.

**Issue:** D1 says `ForceRefresh` ignores the ordering rule and writes the running binary's revision and commit time.
`lyx stencil sync` (`internal/stencilcli/cli.go`, `syncCmd`) runs from any binary, and D2 does not gate it by channel or modified state.
Run from a clean task-branch build, it records a revision W that main does not descend from.
A later production binary R then finds both commits in the shared hub store, `IsAncestor(W, R)` answers false, and the copy is left alone with an Info line.
That holds for every production deploy until main gains a commit descending from W, which never happens if the task lands squashed or is abandoned.
Where the worktree lacks W, the commit-time fallback applies, and a task build committed after the current deploy also blocks the refresh.
This is the stuck state D2's rationale names ("after which the production binary can never move it back"), now reached through the override verb instead of the pre-run.
A modified build is worse: D2 says it has no commit that names its content, yet D1 records its revision as the writer.
Nothing in D1's Bound, D2 or Open risks bounds this.

**Fix:** Decide what a forced refresh from a dev, unstamped or modified binary records.
For example, it writes the body without a recorded revision, so the next production binary refreshes the copy as it does any unstamped copy.
Or it records the revision and the case goes into Open risks, with `lyx stencil sync` from a production binary named as the way out.
Either way, state the choice in D1 and add an acceptance criterion for it.

### [LOW] The drop signature matches a pre-change sync commit

**Location:** `_lyx/discussion/decision-record.md` D3 ("so D5's signature never matches an operator's forced refresh") and D5's signature and Bound.

**Issue:** Today `syncCmd` commits with the messages `"lyx: seed stencils"` and `"lyx: seed specs"` (`internal/stencilcli/cli.go:165` and `:187`), and every path it touches is under the stencils or specs subtree.
The drop signature therefore matches a sync commit made by any pre-change binary.
Open risks already says pre-change binaries stay in use for a while, and a sync commit only rides the next board push, so it can sit unpushed in a clone's ahead set when a new binary's push conflicts.
D5 would then drop it.
The next pre-run does not regenerate it: a forced refresh over a hand-edited or ordering-blocked copy is exactly what the pre-run declines to write.
That contradicts both D5's Bound ("never a hand edit or an operator's forced refresh") and its regeneration claim.

**Fix:** Make the parenthesised label from D3, which no pre-change commit carries, part of the signature, and declare it with the prefix condition.
A pre-change seed commit then falls outside the signature and gets today's push failure.
Alternatively, keep the signature and add the case to Open risks.

### [LOW] A board push cannot land a conflicting hub-wide config commit

**Location:** `_lyx/discussion/decision-record.md` D4, the paragraph ending "or a board push lands the stuck commit", and the third Open risks bullet.

**Issue:** The Open risks bullet is about a hub-wide config commit "whose push conflicts".
`reconcileHubWide` pushes through `bolt.Push` (`internal/hubreconcile/walk.go:201`), the same path every board push takes.
A board push runs the same `pull --rebase` over the same ahead commit, so it meets the same conflict.
D5 leaves non-seed commits in place on a replay conflict, so the board push fails as before.
Only the operator resolving the rebase in the board dir clears it.
A board push lands the commit only when the original failure was transport, not a conflict.
D4 names a board push as an exit without separating those two cases, and Open risks names it under the conflict case, where it does not work.

**Fix:** In D4 and Open risks, say that a conflicting hub-wide commit clears only when the operator resolves `git pull --rebase` in the board dir.
Keep the board push as an exit only for a push that failed in transport.

## Verdicts on the focus items without a finding

- D5 against `lyx stencil sync` and regeneration: for a post-change binary, D3's `lyx: sync` prefix keeps sync commits out of the signature.
  For every seed commit a post-change pre-run makes, the drop leaves either the pulled upstream copy, which is the forward-correct state, or a copy the next allowed pre-run rewrites.
  The pre-change gap is F2.
- Seed commit beneath the triggering board commit: the primitive resets to the upstream that the failed `pull --rebase` just fetched, replays the board commit and retries.
  The retry is a no-op rebase unless upstream moved again, so the acceptance criterion ("drops the seed commits, replays the board commits onto upstream and lands them") follows from the decided primitive.
  Not a defect.
- Consumer hubs and the specs subtree: today `StencilSource` returns the zero `Source` for an empty `sourceDir` (`internal/fabricengine/warpforward.go:49`), and `seedStencilsAt` passes `stencilstore.Source{}` for specs (`cmd/lyx/stencilseed.go:131`).
  D1 decides that the specs Source carries the ordering answer while its source directory stays empty, and that a worktree lacking either commit falls back to commit time.
  The acceptance criteria pin both, so copies there refresh after a first stamp.
  How the Source is reshaped is a Plan-time interface fact the writer stencil excludes.
  Not a defect.
- Which `gitrepo` read: D1, the gitrepo-client-boundary Constraint and the Notes agree.
  Reachability goes through `Repo.IsAncestor` (`internal/gitrepo/ancestry.go`, `runChecked`), and presence through an object read on `readGoGit`.
  `Repo.SHAExists` (`internal/gitrepo/gitrepo.go:199`) is that read, and it errors when an object is missing.
  The Notes describe `HeadContains` correctly, as `StencilSource`'s existing HEAD read rather than D1's.
  Not a defect.
- Failed hub-wide push on each start verb: when upstream moved, the fast-forward refuses on divergence, the stamp stays absent and the Warn repeats, as D4 and the acceptance criterion say.
  When the push failed in transport over an unmoved upstream, the rerun commits nothing (`walk.go:198`) and writes the stamp, and the commit rides the next board push.
  That outcome is acceptable, and D4 does not claim otherwise.
  The disagreement over how a conflict clears is F3.
