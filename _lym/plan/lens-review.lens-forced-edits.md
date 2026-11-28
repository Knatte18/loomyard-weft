```yaml
lens: forced-edits
outcome: reported
```

Task: review the plan through the forced-edits lens, covering what each change forces outside the members its cards name.

1. **BLOCKING — Cards 7, 8 and 10: the new `landingshed.Deps` seams are never registered in the env seam tables.**
   - These cards add `Deps` fields:
     - Card 7: `Queue`, `RecordsHead`, `NewestLanding`, `MergesClean`, and `Backstop`, a struct of funcs.
     - Card 8: `MemberDeps`, `WriteLedgerLanding`, `RecordsToHold`, `BuildChain`, `FirstParent`, and `Detach`, a `DetachDeps` struct.
     - Card 10: `DetachState`.
   - `internal/testkit/envkit/envkit.go:93-102` holds a nil-legal map keyed `Landing.<field>`, and `envkit.go:207` `LandingDeps` fills the landing seams.
   - `internal/battencli/wire_seams_test.go:97-102` holds the allowed-nil table for batten (`"Landing.TaskHead": "loom-only seam, batten lands no branch"`, …).
   - `collectSeamStructs` (`envkit_test.go:248`) walks every struct-typed exported field. `seamViolations` reports a NilSeams test "whose markers miss a glyph of envStructs". So `internal/landingqueue#BackstopSeams` and `internal/landingshed#DetachDeps` must join the `//lym:seam` marker lists in `envkit_test.go:24-40` and `wire_seams_test.go:236-240`.
   - The cards list those tests only under `Uses`, and no card lists `envkit.go`'s map or `LandingDeps`, or battencli's table var. The env seam tests then fail on the first of cards 7, 8 and 10.
   - Fix: each card puts the table var, the `envkit.go` members and the marked test functions under `Edit`, with a row or marker per new field and struct.

2. **MEDIUM — Card 4: the `pushClassified` hold-off is undefined for its callers.**
   - Card 4 makes a held-off code side return nil, but nil reads as "pushed" to its callers. They are `internal/fabricengine/pushanchored.go:331`, `coalesce.go:77`, `bolt.go:147`, `landing.go:51` and `export_test.go:1092`. No card checks or lists what they do on that nil.
   - Card 4 also changes `readLocalBranch`, which `TipsOnOrigin` (`pusherreads.go:205,217`), the hand-over check, and `pushanchored.go:132` read too. The Intent names only `RunCommitsAhead` and `targetAhead`.
   - No row pins the hold-off branch of `pushClassified` itself.

3. **LOW — Card 5: the `finalize.go` file header goes stale.**
   - `internal/landingshed/finalize.go:1-4` reads "With no parent lock held it merges the parent's tip into the task worktree and gates the merged tree. Under the parent lock it checks the parent has not moved since."
   - Card 5's change makes this false, and no card names it.
   - The file comment encloses no member, so the rubric's way is the file self glyph `internal/landingshed/finalize.go#`, which `redundant-file-target` refuses beside the member glyphs. Either the Intent states the header rewrite explicitly, or card 12 lists the file.

4. **LOW — Card 9: `Finalize.runFixer` likely needs the re-sign arrow.**
   - The card says `runFixer` "gains that entry point", entered "with a failure already recorded" and spending the attempt from the recorded log. That implies a new parameter, but the bullet carries no `->` head.
   - Its callers inside `finalize.go`, `gateAndFix` (line 1175) and `judgeFixerChange` (line 1335), are then uncovered unless they are listed.

5. **LOW — Cards 7 and 8: `Finalize.recordLanding` takes a new input with no head.**
   - Both cards edit it so a held or batched run measures from `FirstParent(commit)` instead of `parentHeadMerged`. Its signature is `(commit string, headErr error)` today. If it needs a held/batched flag or the base commit, the arrow is missing. If not, the Intent should say where that choice comes from.

6. **LOW — Card 11: `acquireBranchLock` now does landing-queue work inside fabric.**
   - The card makes `internal/fabriccli` import `landingqueue`, `runpusher` and `pushledger`.
   - `PATTERN-fabric-boundary`'s Fabric isolation rule says a seam only one module could use belongs in that module.
   - Nothing makes the doc wrong that no card lists: `internal/fabriccli/doc.go` sits under `Uses`, but `acquireBranchLock`'s behaviour changes. Check whether the doc describes the lock takers.

Glyph check: every existing glyph the cards list resolved as `found`. That includes `loomCLI.newShed`, `TestMergePrimitives`, `TestRunCommitsAhead`, `resolveWithParentRule`, `reconcileReport`, the wire and envkit tests and `TestLockedCheck_RefusesUnderTheRunLock`.

Tool calls:
- `lym quarry resolve` on 21 listed glyphs, to confirm the spellings, which my context had not resolved.
- A grep for `parentMerger` fakes, `pushClassified` callers, `//lym:seam` markers, Shed builders, `TaskHead` and `runFixer` callers, and lost-race references, to find forced edits beyond my pre-read.
- A grep of the stale lost-race references in `finalize_test.go` and `finalize_owner_test.go`, plus a locate of the seam markers, to tell real hits from a same-named local.
- A read of the seam marker blocks and the drift-test body, to see which tests the markers attach to.
- A read of `envkit_test.go`'s seam walk and battencli's seam table, to confirm nested structs are collected and the table is keyed per field.
- A read of the envkit and battencli `Landing.*` entries, to confirm `envkit.go` and battencli's table list landing seams field by field.
