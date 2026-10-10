---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5 (and D3)
    summary: The seed-commit signature also matches a lyx stencil sync commit, which no later pre-run reproduces, so D5's Bound is false
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5, Acceptance criteria
    summary: D5's all-or-nothing drop is refused whenever the board commit that triggered the push is ahead, which is nearly every Bolt push, so the Goal's drop-and-regenerate almost never fires
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D1, D1 Rationale, Open risks
    summary: D1 assumes the command's worktree holds lyx's own history, which is false in any hub whose code repo is not loomyard and in the specs pass, so those copies never refresh again after the first stamp
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D5, D4, Open risks
    summary: D5 says D4's pull-first and stamp retry cover an unpushed hub-wide config commit, but D4's fast-forward refuses on that same commit, so the retry repeats on every start verb and never clears it
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D1, Constraints (gitrepo-client-boundary line), Notes
    summary: D1 and Constraints require an in-process readGoGit ancestry read, but the in-process read only checks that the object exists, and gitrepo's reachability read is deliberately CLI-bound
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9 Rationale, D8
    summary: D9's bound claims no lyx run writes the store during gc, but D8 exempts the prime's sessions, so the one-day expiry is the only real bound and D9 does not say so
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D8, Open risks
    summary: A probe that keeps failing silently stops gc for good, and neither D8's bound nor Open risks account for that state
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Several D4, D5 and D8 behaviours have no acceptance scenario
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Scope Out, D4 Rationale
    summary: Part 4 and the 1829-commit incident are used without definition in the Plan producer's sole input
---

### [BLOCKING] The seed signature matches `lyx stencil sync` commits, which are not regenerable

**Location:** `_lyx/discussion/decision-record.md` D5, with D3's message form.

**Issue:** D5 drops any ahead commit whose message starts with the seed prefix and whose paths all lie under the stencils or specs subtree.
D5's Bound says this "discards only commits a running binary regenerates on its next pre-run".
`lyx stencil sync` (`internal/stencilcli/cli.go`, `syncCmd`) commits through `CommitSeededStencils` with the same messages, `lyx: seed stencils` and `lyx: seed specs`, into the same subtrees.
D3 keeps the `lyx: seed <subtree>` prefix and adds only the label, so a sync commit still matches the signature.
A sync commit is the operator's deliberate override: it is the way forward D1, D2 and D6 name, and it runs "even from a -dev build" and ignores ancestry.
Once dropped, it comes back only if the next pre-run refreshes, which D1 and D2 forbid exactly where sync was needed:
- a dev, unstamped or modified binary;
- a recorded revision on the pulled copy that is not an ancestor of the running binary;
- a worktree that cannot establish ancestry.
In each of those cases the drop silently loses the override, and the Bound's claim is false.

**Fix:** In D3, give a forced refresh a commit message distinct from the pre-run's seed, for example a `lyx: sync <subtree>` prefix.
In D5, state that the drop signature matches only the pre-run's seed prefix.
Then add an acceptance scenario showing that a sync commit fails the signature.

### [BLOCKING] D5's drop is refused in the case it exists for

**Location:** `_lyx/discussion/decision-record.md` D5; Acceptance criteria, the Bolt push and drop-primitive bullets.

**Issue:** D4 says a seed commit "rides the next board push", so it reaches Bolt's push path only with a board commit on top of it: the write that triggered `Bolt.Push`.
D5's primitive resets Bolt to its upstream and refuses the whole drop "when any ahead commit fails the signature".
In the case the Goal names, a seed commit that conflicts at push time, the ahead set therefore almost always holds a non-seed board commit.
D5 then refuses and falls back to "the push failure and its way forward as today", so the seed commit is left behind, which the Goal says must stop.
The acceptance criterion "drops them and lands the remaining commits" contradicts the primitive as decided.
A reset to upstream leaves no remaining commits, and the next criterion says a non-seed commit in the ahead set refuses the drop.

**Fix:** In D5, decide how a seed commit is removed from an ahead set that also holds board commits.
One option drops only the signature-matching commits and replays the rest onto upstream.
That needs its own bound for a replayed commit that then conflicts.
The other option keeps the whole-set reset and moves seed commits off the board-push route, so they never share an ahead set.
Then make the Bolt push acceptance criterion state what lands.

### [BLOCKING] D1's ancestry read cannot be answered outside loomyard's own repo or in the specs pass

**Location:** `_lyx/discussion/decision-record.md` D1 Decision and Rationale; Open risks.

**Issue:** D1 reads ancestry "against the code worktree the command runs in".
The Rationale says that worktree "holds the history to answer it".
That holds only in a hub whose code repo is loomyard itself.
The seed pass already supports a hub over another repository: `cmd/lyx/stencilseed.go` clears `sourceDir` for "a consumer repo", and `fabricengine.StencilSource` then returns the zero `Source` with no `Build`.
In such a hub neither revision exists in the worktree, so D1's "ancestry the worktree cannot establish" row applies to every file with a recorded revision.
After the first production binary stamps the copies, no later production binary ever refreshes them.
This is a regression from today, where a production binary refreshes an untouched copy.
The specs pass has the same gap: `seedStencilsAt` gives it `stencilstore.Source{}`, and D1 does not say where the specs subtree gets its ancestry answer.
Open risks names only the task-worktree case.

**Fix:** In D1, decide the ordering source for a hub whose code repo is not loomyard and for the specs subtree.
Options include an ordering carried in the binary itself, or a stated fallback when neither revision is known locally.
Then bound that fallback, and add a matching acceptance scenario.

### [MEDIUM] The stamp-absent retry cannot clear a hub-wide config commit that failed to push

**Location:** `_lyx/discussion/decision-record.md` D5's sentence "A hub-wide config commit is never dropped: D4's pull-first and stamp retry cover it"; D4's fast-forward failure rule; Open risks, third bullet.

**Issue:** A hub-wide config commit whose push fails stays ahead in Bolt, and `hubreconcile` leaves the stamp absent.
The next start verb reruns the walk, which runs D4's fetch, drop and fast-forward.
D5's primitive refuses to drop the config commit, so the fast-forward fails on a non-regenerable ahead commit over a moved upstream, and D4 skips the write with a Warn.
The stamp stays absent, so every later start verb repeats the fetch and the Warn.
Pre-run seeding with a write due is also blocked for as long as that commit sits there.
Open risks says the case "needs `git pull --rebase` in the board dir by hand", which contradicts D5's claim that the retry covers it.

**Fix:** In D5, drop the claim that the retry covers it.
In D4, state that a non-regenerable ahead commit stops the walk and every later start verb until the operator acts, and that the Warn names `git pull --rebase` in the board dir.
Alternatively, decide that the walk's own retry pushes the stuck commit through Bolt's rebase-retry push before it fast-forwards.

### [MEDIUM] The ancestry read mechanism contradicts the existing primitives

**Location:** `_lyx/discussion/decision-record.md` D1, "read through `fabricengine`'s existing in-process ancestry read"; Constraints, "the ancestry read through `readGoGit`"; Notes, "D1 needs an is-ancestor read of two revisions, which `gitrepo` exposes".

**Issue:** The existing in-process read, `gitrepo.Repo.HeadContains` behind `StencilSource`, goes through `readGoGit` and answers whether a commit object exists, not ancestry.
`gitrepo.Repo.IsAncestor` is the reachability read D1 needs.
`internal/gitrepo/ancestry.go` makes it CLI-bound through `git merge-base --is-ancestor` on purpose, because an object-existence answer cannot tell reachable from surviving.
So D1 and the Constraints line pin an in-process `readGoGit` route that either does not exist or answers the wrong question.
Meanwhile the Notes point at the CLI-bound read.

**Fix:** In D1, name the read as the reachability read `gitrepo` already has.
Change the Constraints line to route it through `gitexec`, as `IsAncestor` does, and keep `readGoGit` only for the object-presence check.

### [LOW] D9's bound rests on a claim D8 does not make

**Location:** `_lyx/discussion/decision-record.md` D9 Rationale, "With D8 no lyx run writes the store during the gc"; D8, "the prime's own sessions, the orch among them, never count".

**Issue:** D8 deliberately ignores the prime's sessions, and the orch writes the code store from the prime when it lands.
D9's claim is therefore false, and what bounds the newly allowed deletion is the one-day expiry alone.
That deletion is unreachable loose objects between one day and git's two-week default.
The Decision text does not name that window, and Open risks restates it only through git's default.

**Fix:** In D9's Decision, state what gc may now delete (unreachable loose objects older than a day, where it used to wait two weeks) and that the expiry alone bounds it.
Then drop the D8 clause from the Rationale.

### [LOW] A persistently failing probe stops gc silently

**Location:** `_lyx/discussion/decision-record.md` D8, "a probe that fails counts as in flight"; Open risks, fourth bullet.

**Issue:** D8 logs the skip at Info "with the sessions found", but says nothing about a skip caused by a failed probe.
A probe that fails on every removal looks the same as a busy hub, so lyx never gc's the stores and nothing above Info says why.
One example is a listing that errors when the hub's tmux server is not running, which is the quietest hub there is.
Open risks covers only "a hub that is never quiet".

**Fix:** In D8, state that a listing with no server counts as no sessions, not as a failure, and that a skip caused by a probe failure logs a Warn with the error.
Extend the Open risks bullet to cover a probe that keeps failing.

### [LOW] Acceptance criteria miss several decided behaviours

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.

**Issue:** No scenario covers:
- `fabric reconcile`'s hub-config heal running D4's fetch, drop and fast-forward;
- D5's rule that the drop never removes a hub-wide config commit;
- D8's failed probe counting as in flight;
- D2's zero-value `Mode` never refreshing.

D2's zero-value `Mode` rule protects every caller that does not set a mode, not only an unstamped binary, so the unstamped-binary scenario does not cover it.

**Fix:** Add one scenario per behaviour, or fold each into the existing scenario for its Decision.

### [LOW] Terms the Plan producer cannot resolve

**Location:** `_lyx/discussion/decision-record.md` Scope Out, "Part 4 is solved inside housekeeping"; D4 Rationale, "1829 missing commits".

**Issue:** The decision record is the Plan producer's sole input.
"Part 4" is never defined in it.
"1829 missing commits" refers to an incident the record does not describe.
Both are deliberation narrative that points at the board entry or issue.

**Fix:** Rewrite the Scope Out item as the fact itself, for example "no gc or prune at landing; loose-object growth is handled in housekeeping at removal (D9)".
In D4's Rationale, replace the incident count with the general case, "a clone behind its upstream".
The incident detail can move to `support-log.md` if it is worth keeping.
