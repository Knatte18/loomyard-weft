---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-gitrepo-retrying-read.md:61
    summary: Card 2 folds every ErrObjectNotFound from HeadContains' readGoGit call into (false, nil), so a walk that hits a missing ancestor no longer errors and card 1's bound test cannot tell a bounded walk from an unbounded one
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-hub-store-gc.md:298
    summary: Card 5's Remove subtest unsets the gc keys first, but the pair it then creates goes through Topology.Add, which writes them back, so the after-Remove assertion passes even if Remove never writes them
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/01-headcontains-bounded-walk.md:57
    summary: Card 1 does not say the bound step builds its own repository; in TestHistoryReads' shared repository, its old commits move HEAD and the deleted object breaks every later step that walks the whole history
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/05-hub-store-gc.md:293
    summary: D6 requires the PATTERN-hub-store-gc entry itself to state its carve-out, but card 5 puts the carve-out only in the background file, so spawned agents reading the inlined PATTERN overview never see it
---

### [MEDIUM] HeadContains' not-found mapping swallows a walk failure and makes card 1's test vacuous

**Location:** `_lyx/plan/02-gitrepo-retrying-read.md:61` (lines 122 and 124 of the concatenated plan: the composite-method rule and the "maps `plumbing.ErrObjectNotFound` after `readGoGit` returns" rule), with `_lyx/plan/01-headcontains-bounded-walk.md:59`.

**Issue:** Card 2 lets `HeadContains` do "its whole go-git work in one `read` closure (tree resolution, diff and walk together)", and then maps `plumbing.ErrObjectNotFound` from the `readGoGit` call into `false` with no error.
Today only the target lookup gets that mapping (`blobread.go:76-84`), and a walk error is returned wrapped with `%w` (`blobread.go:108-110`).
go-git's committer-time walker returns `GetCommit`'s error as is when a parent is missing (`commit_walker_ctime.go`, `Next`, go-git v5.19.1), so that error's chain holds `plumbing.ErrObjectNotFound`.
If the walk shares the closure, a missing ancestor goes through `readGoGit` (fingerprint unchanged, so no retry) and then into the "absent" mapping, and `HeadContains` returns `(false, nil)`.
This has two effects:
- Card 1's bound test asserts `(false, nil)` "which an unbounded walk cannot, since reading the deleted commit would fail". After card 2, an unbounded walk also returns `(false, nil)`, so the test keeps passing if the bound is removed.
- A store with a missing ancestor now reports `false` silently instead of failing, which contradicts card 2's own "every method's contract stay as they are".

**Fix:** In card 2's Intent, apply the absent-object mapping only to the target lookup.
Either run the target lookup in its own `readGoGit` call and map its not-found before the walk's `readGoGit` call, or have the closure turn a target-lookup not-found into a sentinel and leave the walk's error untouched.
Also state that a not-found during the walk stays an error, so card 1's `(false, nil)` assertion keeps telling the two walks apart.

### [MEDIUM] Card 5's Remove subtest is vacuous as specified

**Location:** `_lyx/plan/05-hub-store-gc.md:298` ("each subtest first unsets them in both stores") and the second subtest ("After a pair is removed, both stores' config holds both keys again").

**Issue:** `hubforge.AddPairWith` calls `h.Topology.Add` (`internal/hubforge/hub.go:334-343`), and after card 5, `Topology.Add` calls `disableStoreAutoGC`.
If the keys are unset before the pair is created, which is the order card 5 describes, Add writes them back.
The after-Remove check then passes even if `Remove` or `housekeepStores` never writes them.
The acceptance criterion this subtest stands for asks about "a hub whose stores lack those keys" at removal time.

**Fix:** For the Remove subtest, have card 5 unset both keys in both stores after `AddPairWith` returns and before `Remove`.
Have the subtest check that they are absent at that point, then assert they are present after `Remove`.
The Add subtest keeps unsetting them before the pair is created.

### [LOW] Card 1's bound step does not say it uses its own repository

**Location:** `_lyx/plan/01-headcontains-bounded-walk.md:57`.

**Issue:** "`TestHistoryReads` ... gains a step for the bound. Its fixture commits at least three old commits ... then deletes the oldest old commit's loose object file."
`TestHistoryReads` runs its steps serially against one shared repository (`blobread_integration_test.go:27-44`).
Other steps depend on HEAD being commit C, and some walk the whole history (`PathRevisions`).
Read as "the test's fixture", the step moves HEAD and deletes an object those steps would later read, and the outcome depends on step order.

**Fix:** State that the bound step builds its own repository with `newRepo` and leaves the shared fixture untouched.

### [LOW] The PATTERN-hub-store-gc entry drops the carve-out D6 puts in the entry

**Location:** `_lyx/plan/05-hub-store-gc.md:293-294`.

**Issue:** D6 says the D2 PATTERN entry "states its carve-out", meaning housekeeping is not a destructive primitive under `PATTERN-fabric-destruction-chokepoint`.
Card 5 puts that only in `pattern/PATTERN-hub-store-gc.md`.
Spawned agents get only the PATTERN overview inlined in their prompts.
A reviewer applying `PATTERN-fabric-destruction-chokepoint` from the overview alone would then see a fabricengine file outside `destroy.go` deleting objects, with no exemption stated.
`PATTERN.md` is 16185 of 16384 bytes, but card 2 already frees room, and a short clause such as "not a destructive primitive" costs a few dozen bytes.

**Fix:** Have card 5's entry line carry a short carve-out clause that points at `PATTERN-fabric-destruction-chokepoint`, and keep the full carve-out in the background file.

## Focus directives

- **Card 5, gc failure.** Not a defect.
  I checked with git 2.53.0.
  `gc.bigPackThreshold=bogus` makes `git gc --auto` exit 128 with "bad numeric config value" at both `-c gc.auto=6700` and `-c gc.auto=0`, so the failure subtest gets a non-zero exit whatever the threshold.
  A live process holding `gc.pid`, with the store above the auto limit (three packs, `gc.autoPackLimit=2`), gives exit 0, no output and no repack; a second run after releasing the lock repacked to one pack.
  So the Intent is right that a held `gc.pid` is no failure, and the subtest does not depend on it.
- **Card 2, regression table.** Not a defect.
  Each row has its own fresh `Repo`.
  `forcePackIndexFreeze`'s fabricated miss runs with snapshot `""` and current fingerprint `""` in a repository with no packs, so `readGoGit` returns without reindexing and `lastPackFingerprint` stays `""`.
  The row's own commit, repack and prune-packed then change the fingerprint, so each row's single read reaches a stale index and goes through the retry by itself.
- **Card 1, walk order.** Deterministic.
  go-git v5.19.1's `commitIteratorByCTime.Next` loads every parent of a commit before returning it.
  The bounded walk therefore reads HEAD, HEAD's parent and that parent's parent, and never the commit three generations down.
  The card's "at least three generations below HEAD" placement is correct.
  F1 is the separate problem that card 2 makes this assertion unable to fail.
- **Card 4, process reaping.** Deterministic to the same degree as `verifyrun`'s accepted `TestRun_ExpiryKillsBackgroundedDescendant`, which polls `Signal(0)` to a bounded deadline.
  I checked with this git that an `ext::sh -c ...` remote records its pid and hangs until killed, and that the `ext::` command's stderr reaches git's stderr, which the low-speed environment row relies on.

## Focus departures

None.
