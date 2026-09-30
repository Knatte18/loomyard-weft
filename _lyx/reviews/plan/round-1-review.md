---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/03-add-remote-preflight.md
    summary: The unarchived-leftover test asserts no warp branch is left but never asserts no local weft branch is left, which the acceptance criterion names
  - id: F2
    severity: LOW
    location: _lyx/plan/03-add-remote-preflight.md
    summary: The SkipPush test checks only origin's weft branch, while the criterion says a SkipPush re-add touches no remote and step 11 pushes the warp branch regardless of SkipPush
  - id: F3
    severity: LOW
    location: _lyx/plan/03-add-remote-preflight.md
    summary: ImpactSummary claims no other change for callers but names none of the real Topology.Add callers or the two ls-remote reads every push-enabled Add now makes
  - id: F4
    severity: LOW
    location: _lyx/plan/05-add-replace-archived-weft.md
    summary: The warp fast-forwardable test sits in Card 5, so Card 3's own tests never exercise the warp probe's pass branch
  - id: F5
    severity: LOW
    location: _lyx/plan/05-add-replace-archived-weft.md
    summary: ImpactSummary omits the rollback change (a replaced remote weft branch is never restored) and the new package-level test hook
  - id: F6
    severity: LOW
    location: _lyx/plan/02-gate-archived-weft-delete.md
    summary: The empty-lease-SHA refusal on the archived-on-remote dirtiness kind is specified but no test covers it
  - id: F7
    severity: LOW
    location: _lyx/plan/03-add-remote-preflight.md
    summary: The shared ancestor helper re-implements merge-base --is-ancestor exit-code handling that gitrepo.Repo.IsAncestor already provides
  - id: F8
    severity: NIT
    location: _lyx/plan/02-gate-archived-weft-delete.md
    summary: destroyremote_integration_test.go's file header says no test in it deletes a remote ref, which Card 2's new cases make false, and the card does not say to update it
  - id: F9
    severity: NIT
    location: _lyx/plan/03-add-remote-preflight.md
    summary: Card 3 says the step-12 replacement comes from the next card, but Card 4 is next and Card 5 adds it
---

The plan carries every Decision and Constraint in the decision record.
The five cards map cleanly onto the record's scope: the leased `gitrepo` delete, the `destroy.go` extension, the pre-flight probes, the CLI envelope, and the step-12 replacement with docs.
No card deletes or force-pushes a remote warp branch, and none changes `Remove`.

I checked the git mechanics the plan relies on against a scratch repo.
`git push --force-with-lease=refs/heads/<b>:<sha> <remote> --delete <b>` deletes at a matching lease, is rejected as `stale info` at a moved one, and fails on an absent ref, so Card 1's semantics hold.
`git fetch --no-tags --refmap= origin refs/heads/<b> refs/tags/<t>` writes both into `FETCH_HEAD` and creates no local ref, so Card 3's fetch keeps the record's open-risk promise.
`git ls-remote --tags origin 'refs/tags/archive/x/*'` matches `archive/x/abc` and not `archive/x-y/…` or `archive/ax/…`.

### [MEDIUM] Unarchived-leftover test misses the weft branch

**Location:** `_lyx/plan/03-add-remote-preflight.md`, the first test bullet.
**Issue:** The acceptance criterion says a refused Add leaves "no warp or weft worktree, branch, portal or launcher".
The card asserts no warp or weft worktree, no warp branch, no portal and no launcher, so the local `<slug>-weft` branch is not asserted.
A plain Remove deletes that branch locally, so its absence after the refusal is the observable proof that the refusal ran before `createWeftWorktree`.
**Fix:** Add "no local `<slug>-weft` branch" to the test's post-conditions.

### [LOW] SkipPush test checks only one origin

**Location:** `_lyx/plan/03-add-remote-preflight.md`, the `SkipPush` test bullet.
**Issue:** The criterion says a SkipPush re-add "touches no remote".
In `internal/fabricengine/add.go`, step 11 runs `push -u origin <warpBranch>` unconditionally, and only `pushWeftBranch` honours `SkipPush`/`SkipGit`.
The test asserts only that origin's weft branch is unchanged, so the warp side of the criterion goes unchecked, and nothing in the plan records that the warp push still runs.
Under this setup the re-created warp branch sits at the same prime HEAD the first Add pushed, so the step-11 push leaves origin's warp ref where it was.
**Fix:** Assert origin's warp branch is unchanged too.
In the Intent, state that step 11 is not gated by `SkipPush`, which is outside this plan's scope, so the assertion holds only because the warp push is a no-op here.

### [LOW] Card 3 ImpactSummary does not cover the real caller set

**Location:** `_lyx/plan/03-add-remote-preflight.md`, `**ImpactSummary:**`.
**Issue:** "callers see a new typed refusal and no other change" names no callers.
The real set calls `Topology.Add` with a zero `AddOptions{}`: `fabriccli.runAdd` (`fabric.go:510`), `battencli`'s create-worktree row (`wire.go:324`, which passes an unrecognised error through `createRefusal` unchanged), and `hubforge.AddPair` (`hub.go:326`), which test suites across roughly ten packages use.
Every one now makes two `ls-remote` reads before its first mutation, and an unreachable origin now fails Add at pre-flight rather than at step 11.
No existing test outside `fabricengine` re-adds a removed slug, so the refusal is reachable there only through new behaviour.
**Fix:** Rewrite the ImpactSummary to name the three production and test entry points, state that the refusal reaches `battencli` unchanged, and note the added reads and the earlier failure point for an unreachable origin.

### [LOW] Warp probe's pass branch untested in its own card

**Location:** `_lyx/plan/05-add-replace-archived-weft.md`, the "warp fast-forwardable" test bullet.
**Issue:** `probeWarpLeftover`'s pass branch (the remote tip is an ancestor of the warp HEAD) is Card 3's code.
Card 3's own tests exercise only the refusal branch, so a Card 3 defect in the pass branch surfaces only in Card 5.
The test sits in Card 5 because a plain Remove leaves an archived weft leftover, which only Card 5 replaces.
A `Remove(l, slug, false, true)` deletes the remote weft branch but never a remote warp branch (per `remove.go`'s doc comment), so Card 3 can test the warp pass branch deterministically on its own.
**Fix:** Move the test to Card 3's integration file, set up with `remote=true` on the Remove, and drop it from Card 5.

### [LOW] Card 5 ImpactSummary omits the rollback change

**Location:** `_lyx/plan/05-add-replace-archived-weft.md`, `**ImpactSummary:**`.
**Issue:** The summary names the new deletion and the docs.
It leaves out what changes for a failing Add.
Once the replacement has run, a step-12 push failure leaves origin with no `<slug>-weft` branch at all, and the old content survives only in the archive tag.
The summary also omits the new package-level `addBeforeWeftReplaceHook` var, which is nil in production and which makes the lease-race test non-parallel.
**Fix:** Extend the summary with the post-replacement rollback outcome and the hook.

### [LOW] Empty-lease refusal has no test

**Location:** `_lyx/plan/02-gate-archived-weft-delete.md`, Intent and Tests.
**Issue:** The card has `checkRemoteBranchRequest` refuse the archived-on-remote kind on an empty lease SHA.
The record licenses this, because its dirtiness kind is answered by the reachability proof and the lease SHA together.
None of the listed cases exercises the refusal, although the integration test already covers the sibling empty-archive-tag refusal.
**Fix:** Add an integration case: a valid archive tag with an empty lease SHA is refused on `CheckDirtiness`, and origin keeps the branch.

### [LOW] Ancestor helper duplicates gitrepo.IsAncestor

**Location:** `_lyx/plan/03-add-remote-preflight.md`, the ancestor-helper bullet.
**Issue:** The helper is specified as `merge-base --is-ancestor` with exit 1 meaning no and any other failure returned.
That is exactly `gitrepo.Repo.IsAncestor` (`internal/gitrepo/ancestry.go`), which `fabricengine` already calls in `mergeguards.go` and `merge.go`.
The only new logic is the missing-object pre-check.
**Fix:** Build the helper as the `cat-file -e <sha>^{commit}` pre-check followed by `gitrepo.New(dir).IsAncestor`, and add `internal/gitrepo#Repo.IsAncestor` to the card's `Uses`.

### [NIT] Stale file header in destroyremote_integration_test.go

**Location:** `_lyx/plan/02-gate-archived-weft-delete.md`, the `destroyremote_integration_test.go` bullet.
**Issue:** That file's header says each case runs through `CheckRemoteBranchRequestForTest`, never through `deleteRemoteBranch`, and that "no test here asserts a ref was actually deleted on a remote".
Card 2's new cases drive `deleteRemoteBranch` and assert an actual remote deletion.
**Fix:** Have the card update the header to describe the new executor-level cases.

### [NIT] Card 3 names the wrong follow-up card

**Location:** `_lyx/plan/03-add-remote-preflight.md`, the `add.go` paragraph.
**Issue:** "Hold the `weftLeftover` answer for the step-12 replacement the next card adds" is wrong, because Card 4 is the CLI envelope and Card 5 adds the replacement.
**Fix:** Name Card 5.

## Focus verdicts

- **Card 3 targets (`probeWarpLeftover`, the ancestor helper):** not a defect.
  Both live in `remoteleftover.go`, which Card 3 creates.
  No other card references either, so declaring them as `plan:` handles would trip `handle-unreferenced`.
  `probeWarpLeftover` has its own refusal test in the same card.
- **Card 3 ImpactSummary vs `hubforge`:** a finding (F3).
- **Unarchived-leftover assertions:** a finding (F1).
- **`destroy.go` cases:** all six record cases are carried.
  Unset ownership, unset dirtiness and wrong branch are hermetic.
  Checked-out weft not refused, unarchived tip (as an empty tag) and stale lease are integration tests.
  The extra cases are licensed: primary weft refused follows from the record's ownership definition, and a `branchRequest` refusing the remote-only kind keeps the shared enum closed.
  The empty-lease refusal is licensed but untested (F6).
- **Adopt path and warp side:** not a defect.
  With `localExists` set, Card 3 answers from the local-tip ancestor test alone and never consults an archive tag, so a diverged remote is refused even when a tag covers it.
  Card 5 replaces only on a non-empty answer, and no card deletes or force-pushes a remote warp branch.
- **Warp fast-forwardable test placement:** a finding (F4).
- **Card 5 ImpactSummary:** a finding (F5).
