# Lens review

Every lens fork returned a report.

- criteria-coverage, MEDIUM, Merge-In skip after a writer-gate pass untested: fixed as proposed; Card 7's darn-form steps add a `verifytree.Verify` call at site `Merge-In` with the same scoped command on the same tree, asserting `StatusSkipped`.
- criteria-coverage and branch-coverage (merged), MEDIUM, no `Publish.Call` row with the seam answering false or across two calls: fixed as proposed; Card 8's `TestPublishVerify_Proceeds` gains a false row and a two-call true-then-false row.
- criteria-coverage and branch-coverage (merged), MEDIUM, no post-merge classification step: fixed as proposed; Card 7's `TestDarnTreeScope_ClassifiesAtEachCall` gains a parent merge that stays tree-scoped and one whose resolution adds a production path, and Card 8 asserts the landing source at those steps.
- criteria-coverage, MEDIUM, the change-description criterion claimed with no carrier: fixed as proposed (second option); the overview's Shared Decision now says the criterion is met at Describe, outside every card, from the `doc.go` text Card 2 writes.
- criteria-coverage, LOW, no rename step through the wiring seam: fixed as proposed; Card 7 adds a rename of `internal/x.go` to `tools/x.go` answering false.
- criteria-coverage, LOW, reconcile test bypasses configsync (merged with test-economy, LOW, `TestReconcileKeepsTreeScope` repeats yamlengine's open-map carry): fixed differently; test-economy's proposed drop alone would leave darn's `MigrateConfig` plus reconcile path unpinned (`internal/configsync/configsync.go:604-610` runs `Migrate` before `Reconcile`), so the new top-level test is replaced by a darn row in `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`, and `./internal/configsync` joins the plan verify.
- criteria-coverage, LOW, no editor row clearing both keys in one save: fixed as proposed; Card 3 adds that row.
- criteria-coverage, LOW, no tree-scope-off assertion on the landing source: fixed as proposed; Card 8 asserts `VerifyCommand()` and `PublishVerifySkip()` at the emptied-`tree_scope_verify` step too.
- branch-coverage and row-reachability (merged), MEDIUM, `GuardNotIntegration` way forward unreached: fixed as proposed; Card 6's scenario adds a marked test in a `//go:build tmux` file.
- branch-coverage, MEDIUM, stop at the first non-zero exit untested: fixed as proposed; Card 6 adds a row with the stand-in exiting 3.
- branch-coverage, MEDIUM, darn-form guard-scan rejection unreached: fixed as proposed; Card 7's misplaced-marker step adds a darn-form call.
- branch-coverage and row-reachability (merged), LOW, parse-kind `Line` never asserted: fixed as proposed; Card 4's parse row asserts a non-zero line.
- branch-coverage, LOW, `--guards` with no marked test unspecified: fixed as proposed; Card 6 specifies exit 0 with no spawn and adds a row.
- branch-coverage, LOW, darn-form headroom, plan-verify record and fallback rows: fixed as proposed for headroom and plan-verify, which Card 7 adds as darn-form calls; the fallback was already reached by Card 7's non-Go-file step.
- branch-coverage, LOW, seam uncalled with `publish_verify` empty untested: fixed as proposed; Card 8 adds a row whose seam fails the test when called.
- branch-coverage, LOW, `darnTreeScope` and `darnVerifySource` error branches: fixed differently; the proposed rows for every error were not all added, because the merge-base and diff reads propagate raw git errors the refusal spec exempts as transient (rubric "Reachability of a prescribed refusal"); the config refusal, which is no I/O error, gets a step in `TestDarnTreeScope_ClassifiesAtEachCall` naming `./tools`.
- branch-coverage, LOW, multiple guard packages and `-run` override untested: fixed as proposed; Card 6's module holds `g` and `h`, and a row passes `-- -run X`.
- test-economy, LOW, `TestEditHubWideLoadCheck` should be rows of `TestDispatchHubWideBoard`: fixed as proposed; the test is dropped and its rows join `TestDispatchHubWideBoard` through a `module` column.
- test-economy, LOW, two `--set` refusal rows repeat covered behaviour: fixed as proposed (alternative); the no-tree refusal row is dropped and the duplicate-key row is kept, stated as pinning D1's `--set` way forward for a duplicate.
- test-economy, LOW, Publish skip spread over two tests: fixed as proposed; the skip row in `TestPublishVerify_Proceeds` writes the record first and asserts its removal, and `TestPublishVerify_FailureRecord` leaves Card 8.
- forced-edits, LOW, darnengine's `verify` descriptions stale: fixed as proposed; Card 2 restates `Config.Verify`, the template comment, `config.go`'s header and `doc.go`.
- forced-edits, LOW, impactset's package sentence and the overview line name only the round gate: fixed as proposed; Card 5 restates both and lists `docs/overview.md`.
- forced-edits, MEDIUM, overview gate synopsis lacks `--guards`: fixed as proposed; Card 6 lists `docs/overview.md` and restates the synopsis in the same commit.
- forced-edits, LOW, `wireDarn` and `darnVerifySource` doc comments: fixed as proposed; Cards 7 and 8 name the restatements.
- row-reachability, MEDIUM, the run-start editor way forward never reaches the operator because `loomengine.RecipeOverride` loads `darn.yaml` first (`internal/loomcli/sharedbootstrap.go:110`, `internal/loomengine/config.go:1163`): fixed differently; the proposal named the callers only, so Card 2 adds `darnengine.WithEditorWayForward`, and Card 7 routes `RecipeOverride`'s darn branch and `wireDarn` through it, with a `TestRecipeOverride` row.
- row-reachability, MEDIUM, the merge-base row cannot tell the merge base from the latest recorded pass: fixed as proposed; Card 7's row tells a merge base after `planPass.Commit` with `a/a.go` changes between them and asserts no pass of the command derived from `planPass.Commit`.
- row-reachability, LOW, `--guards ./pkg` row may hit the package-directory refusal first: fixed as proposed; Card 6 orders the tag refusal first and the row lists `pkg` among its dirs.
- row-reachability, LOW, `TestFindGuardTests` needs per-row kind and line fields: fixed as proposed; Card 4 names `wantKind` and `wantLine`, line 4 for the floating-body row.
