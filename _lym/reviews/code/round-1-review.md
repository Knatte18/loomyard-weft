---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/originreconcile.go:640
    summary: A run-scope reconcile with PairMayBeAbsent treats a registered but prunable task worktree as present, so the slug is never classified and an owned-elsewhere, ended or stopped entry is not judged before the re-adopt
    fix: In originTargets, compute registered over non-prunable entries only, by adding "!entry.Prunable &&" inside the slices.ContainsFunc predicate at line 640. A prunable pair then takes the "!registered" case (absent = true, slug joins pairs, wanted rejects every entry) when scope.PairMayBeAbsent is set, and still returns today's error when it is not. Add a row to the PairMayBeAbsent subtest in internal/fabricengine/originreconcile_integration_test.go that deletes the task worktree directory without pruning, then asserts PairAbsent on the run's entry and that an owned-elsewhere owner reads owned_elsewhere.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/battencli/readoptstop.go:22
    summary: battencli re-declares the archive tag's 12-digit tip length and hand-forms the tag name, which fabricengine owns, so a change to the tag format leaves the way forward naming a tag that archiving never made
    fix: In internal/fabricengine/archive.go export the tag-name function as ArchiveTagName (rename archiveTagName and its callers in the package, and update its doc comment opening). In internal/battencli/readoptstop.go delete archiveTagTipLength and replace the tag construction in leftoverArchiveWayForward (line 173) with fabricengine.ArchiveTagName(c.slug, tip); keep the short-tip guard by having the caller check len(tip) against the same exported length, or drop the guard if ArchiveTagName returns an error for a short tip.
    origin: handler
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/add_adoptonly_integration_test.go:90
    summary: The adopt-only skips of the acting worktree's detached-or-unborn HEAD refusal and of the fork point's empty-marker and anchor-move stop have no test row, though the record's D1 lists them; only the source-dirt skip is pinned
    fix: Add two rows to the adopt-only table in add_adoptonly_integration_test.go. One runs Add with AdoptOnly from a prime whose HEAD is detached and asserts the pair is re-adopted. The other runs it with the prime's own records marker empty, or its anchor move record present, and asserts the same. Each must fail if the matching "!opts.AdoptOnly" guard in add.go is removed.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/fabricengine/cleanupremotewarp.go:149
    summary: Error branches of the diff have no row - the remote-warp sweep's non-held teardown-lock error, the gone-read error of refuseContinuePass, and the unreadable-prime branch of the loomcli RemoveFollowUp closure
    fix: Add one row per branch. In the existing remote-warp sweep test, make lockPairTeardown fail with an error other than ErrPairBusy and assert the candidate's Error is set and its Reason empty. In internal/battencli/continue_pass_test.go add a refuseContinuePass row whose Gone seam returns an error and assert the refusal text starts "whether its task worktree is gone does not read". In a loomcli landing-deps test call RemoveFollowUp with an unreadable prime and assert it returns "".
    origin: lens:code-tests
---

### [MEDIUM] Prunable registration skips the absent-pair judgment

**Location:** internal/fabricengine/originreconcile.go:640, with the skip at :663.
**Issue:** `registered` is true for a task worktree whose directory is gone but whose registration `git worktree prune` has not yet dropped, because `slices.ContainsFunc` does not read `Prunable`. The `default` case is taken, and the loop at :663 then skips the prunable entry, so the slug never joins `pairs`.
`classifyEntries` builds the run's entry only from `recorded`, which derives from `pairs`, so the entry is never classified and `PairAbsent` stays false.
`resumeRefusal` finds no entry, and `readoptGonePair` goes on to prune and `Add` without having judged whether origin reads the run as owned by another machine, ended or stopped.
D1 names this stale-registration case itself (acceptance bullet 2) and requires the first reconcile to judge the entry before the pair is touched; `PATTERN-owner-machine` bars resuming where the entry is owned elsewhere.
The parent's branch is also still judged, which D1 says it is not while the pair is absent.

### [LOW] battencli duplicates the archive tag format

**Location:** internal/battencli/readoptstop.go:22, :173.
**Issue:** `archiveTagTipLength = 12` repeats fabricengine's `archiveTagSHALen`, and `leftoverArchiveWayForward` hand-forms `archive/<slug>/<first 12 hex>` beside fabricengine's `archiveTagName`.
`PATTERN-named-constants` asks for one declaration of a name that can go stale.
The same pattern holds for `fabricRemoveCommand`, declared once in battencli and once in landingshed, and for `landRow` and `worktreeTeardownRow` in pairteardown; the latter pair is held for an import cycle and pinned by `TestBattenFollowUp`, so only the tag format is filed.

### [LOW] Adopt-only skips of the HEAD and fork-point checks are unpinned

**Location:** internal/fabricengine/add_adoptonly_integration_test.go:90.
**Issue:** D1 lists the source-dirt check, the acting worktree's HEAD read with its detached-or-unborn refusal, and the fork point's empty-marker refusal and anchor-move stop as checks adopt-only skips. Only the source-dirt skip has a subtest, so removing either other guard breaks no test.

### [LOW] Error branches without a test row

**Location:** internal/fabricengine/cleanupremotewarp.go:149, internal/battencli/arm.go:640, internal/loomcli/landingdeps.go:130.
**Issue:** Three added error branches have no row: the remote-warp sweep's non-held lock error, `refuseContinuePass` when `Gone` errors, and the `RemoveFollowUp` closure when the prime is unreadable. The weft sweep's equivalent lock-error branch is pinned at `pairbranch_archive_integration_test.go:200`, which shows the expected shape.

## Rejected

- Lens code-affected-set F1 (D3 follow-up missing at `battencli/wire.go:319` and `fabricengine/failedpush.go:233`): the `incompletePairRemedy` remedy already tells the operator to delete the leftover task branch, which is the part of the cancel D3's rationale needs, and a re-run of `lym batten run` starts a fresh create from Worktree-Create. The failed-push way forward lives in fabricengine, which PATTERN-fabric-isolation bars from learning of batten's records; the batten consumers that quote it (`stoppedRefusal` paths, `reportVerdict`) do carry the follow-up, as card 13 specifies.
- Lens code-affected-set F2 (`loomcli/landingdeps.go:338` returns `ErrPairPushRunning` unhandled): a landing's push must succeed for the landing to proceed, so a loud halt naming the removal and the re-run is the right answer; D7 names only the `commit-records` push for the skip.
- Lens code-affected-set F3 (undeclared edit to `pattern/PATTERN-fabric-git.md`): the entry's binding text in `PATTERN.md` is unchanged and still true; the added bullet is the documentation of D4's new behaviour that the rubric's repository-rules bullet expects to land in the same diff.
- Lens code-tests L1 (no test that a plain push refused with "missing necessary objects" is not retried): `PushWithRetry` passes no extra reasons by construction, and the existing retry tables already pin the classifier; a row would test unchanged default behaviour.
- Lens code-tests, D4 pause gate on the branch push has no row: the gate is inherited from the remote-refs lock, which `PATTERN-remote-refs-lock` pins with its own test.
