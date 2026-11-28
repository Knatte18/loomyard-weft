```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches.

**Each function or branch, the case I ran, and the result:**

1. **`TargetSnapshot.overlayID`, the new `len(files) == 0` branch** (`internal/shedadapters/targetsnapshot.go:114-116`):
   - `Paths` nil or empty: `listTargetFiles` returns `[]`, so the branch returns an error and no hash. Correct.
   - Every path missing: each one is skipped at `os.Lstat` (`targetsnapshot.go:232`), the list is empty, and it returns an error. Correct.
   - One path is an existing but empty directory: `WalkDir` adds no regular file, so it returns an error, not the hash of empty input. Correct under the entry's "empty hash input should fail loudly".
   - One path is a dangling symlink: `Lstat` finds it, but it is not a regular file, so nothing is added and it returns an error. Correct.
   - One path is missing beside an existing file: the missing path is skipped and the file is hashed, as before. Correct.
   - When `listTargetFiles` itself fails, it returns that error before reaching the new branch. Correct.
2. **`TargetSnapshot.Take` when `ID` fails** (`targetsnapshot.go:59-65`): it returns `"", err`, and `copyOverlay` never runs, so no cache directory is created for an empty id. Correct. Under the old code, an empty target made `copyOverlay` rename an empty staging directory into `CacheDir/e3b0…`. The new branch removes that state.
3. **`recordRoundStart` when `Take` fails** (`internal/shedadapters/burlerreview.go:229-232`): it logs at Warn and leaves `TargetID` as `""`. The record is still written with its paths and digests. Correct, and the state stays consistent.
4. **`recordRoundChanges` when the previous round's `TargetID` is `""`** (`internal/shedadapters/roundcontext.go:298`): it writes no changes file, and the round falls back to the non-incremental hold. Correct.
5. **`recordRoundChanges` when the previous id exists but the fixer removed every target file** (`overlayChangedSince`, `targetsnapshot.go:172-211`): `after` is `[]`, so every file in the cached copy is listed as changed. Correct. The new error does not reach this path, because `ChangedSince` never calls `ID`.
6. **`recordPostFixTargetID` when `ID` fails** (`internal/shedadapters/bouncerstalemate.go:33-37`): it logs and records no post-fix id. On a replay, `PostFixTargetID == ""` triggers the read again (`bouncerstalemate.go:28`). If the target has reappeared by then, the post-fix id is set, but a `TargetID` of `""` still fails the `roundUnchanged` guard. Correct: the round counts as changed.
7. **`roundUnchanged` when both ids are empty** (`bouncerstalemate.go:55`): the `record.TargetID == ""` guard returns false. Correct.
8. **`fixLeftTargetUnchanged` when both ids are empty** (`internal/shedadapters/bouncerverify.go:185`): it returns false. Correct.
9. **`reviewTargetSnapshot` root, loom wiring** (`internal/shedrecipe/entries_burler.go:59` against `internal/loomcli/wiring.go:608,913` and `internal/hubgeom/hubgeom.go:169`): `Env.AnchorPath` is `location.AnchorPath()`, which is the burler geometry's `WorktreeRoot`. The snapshot root and the root the reviewers and fixer use now agree, and that holds on a subpath anchor too. Correct.
10. **`reviewTargetSnapshot` root, Bouncer row** (`internal/shedrecipe/entries_bouncer.go:225,312`): `AnchorPath` is required to be absolute, and the Bouncer builds its snapshot through the same constructor, so it hashes under the same root as the review row. Correct.
11. **`reviewTargetSnapshot` with an empty `Env.AnchorPath` on a BurlerReview row**: the row never checks `AnchorPath`, so the root would be `""` and the paths would resolve against the process cwd. The old field, `WorktreeRoot`, was not checked either, and `CacheDir` already used `AnchorPath`, so the diff adds no new failure. Every production Env that runs a burler row sets `AnchorPath` (`internal/loomcli/wiring.go:913`). Correct as far as this diff goes.
12. **A run already in flight across the upgrade:**
    - A round whose start the old binary recorded as `e3b0…` gets a real post-fix hash from the new binary. The two differ, so the round counts as changed. Correct.
    - The new `recordRoundChanges` against `CacheDir/e3b0…` (the old binary's empty copy) lists every current file as added. Correct.
    - A round where the old binary recorded both ids as `e3b0…` keeps its recorded verdict on replay. That is recorded state the diff does not reach, and the board entry's workaround covers it. No defect in the diff's code.

**Findings:** none. Every changed function and branch gave the correct result in each case above, and no failure path swallows an error, misreports one or leaves state inconsistent.

**Tool calls:** none; my inherited context held every definition and call site the cases needed.
