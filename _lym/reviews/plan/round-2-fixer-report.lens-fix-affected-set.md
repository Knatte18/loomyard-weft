```yaml
lens: fix-affected-set
outcome: reported
```

The fix covers every member of the F1, F2 and F4 sets. The F3 set misses two sites.

**Findings left unchecked:** none. F5 and F6 have no set premise, so there was no set search to rerun for them.

**F1 (landingshed import allowlist): covered.**
- Across cards 8, 11, 13, 14 and 15, landingshed production code adds four imports the allowlist lacks:
  - `githubclient`: card 8, through `ParseOwnerRepo`.
  - `pullrequest`: cards 11, 13 and 15.
  - `parentdirective` and `editdirective`: card 14.
- Each one is now added by the first card that imports it.
- That matters because `AssertImportAllowlistNoStale` also fails on a listed import that no production file uses.
- Card 13's `gateslot` is already on the list.
- `hubgeom`, `boardengine`, `dobbyengine` and `runproc` are used only in loomcli.

**F2 (importers of `internal/pullrequest`): covered.**
- The cards whose code imports it are 7 (the package itself), 9 (loomcli), 11, 13, 14 and 15 (landingshed), 16 (pairteardown) and envkit.
- Card 16's widened guard allows all of them.
- Card 17 and card 12 import nothing from it.

**F3 (docs falsified once the row stops being a stub): two sites missed.**
- Most sites are covered: the recipe headers (card 11), the `loomengine` and `darnengine` config and template lines (card 12), the shedrecipe file, doc and test-env comments (card 11), `docs/overview.md:269` and `:578`, `README.md:70` and `internal/loomrecipe/doc.go:16`.
- LOW — card 11, Docs paragraph ("the landingshed doc's PR-Review paragraph describes the row as above"):
  - `internal/landingshed/doc.go:1` is the package summary sentence ("owns landing's general producers, Parent-Align, the PR-Review stub and Finalize").
  - It sits outside the PR-Review paragraph, and no card names it, so "the PR-Review stub" stays after card 11.
  - The evidence is the grep hit at `internal/landingshed/doc.go:1`.
  - This is the same gap the fixer closed for the loomrecipe doc sentence.
- LOW — card 11, the `TestPRReview_Call` rewrite:
  - The file comment of `internal/landingshed/prreview_test.go:3` reads "covers the PR-Review stub: it passes with its flag off and on, logs once only when on…".
  - Card 11 rewrites the file comments of `prreview.go` and `entries_prreview.go`, but no card rewrites this test file's comment.
  - The type doc at `internal/landingshed/prreview.go:13` ("standing in for the pull-request review that is not built yet") is also not named. It sits under the `PRReview` Edit target, but card 11's Intent rewrites only the file comments.

**F4 (sites that remove a `ClosedForContinue` record): covered.**
- Card 11's step 6 is the only remover.
- The clauses in card 15 (Finalize lands locally on `ClosedForContinue`, and a `Continue` without `ClosedForContinue` halts), card 9 (pr-continue is a no-op on either flag) and card 7 (record fields) still agree with the equal-head keep.

Tool calls:
1. grep of the package references in cards 8, 11, 13, 14 and 15: to list landingshed's new imports, which my context held only partly.
2. grep of the plan for `pullrequest` importers, and of cards 9, 12 and 17's targets: those cards were not in my context.
3. repo-wide grep for PR-Review stub and always-Done wording: the F3 set beyond the two sites the review named.
4. grep of card 12 and the landingshed-doc mentions in cards 11 to 15: to check whether the remaining stub sites are covered.
5. grep of the plan for `ClosedForContinue` removal and reroute wording: the F4 set.
6. grep of the `ClosedForContinue` clauses in cards 15, 9 and 7: those card texts were not in my context.
