# Card 6 — git-helpers-other-packages

**Edit:**
- `gitrepo/ancestry_integration_test.go`
- `gitrepo/blobread_integration_test.go`
- `gitrepo/commitempty_integration_test.go`
- `gitrepo/deleteremotebranch_integration_test.go`
- `gitrepo/fetch_integration_test.go`
- `gitrepo/gitrepo_test.go`
- `gitrepo/gogit_test.go`
- `gitrepo/merge_integration_test.go`
- `gitrepo/parity_test.go`
- `gitrepo/pull_test.go`
- `gitrepo/push_test.go`
- `gitrepo/remote_integration_test.go`
- `gitrepo/reset_test.go`
- `gitrepo/worktree_test.go`
- `fabriccli/cloneconfigcommit_integration_test.go`
- `fabriccli/envelopecontract_integration_test.go`
- `fabriccli/merge_cli_integration_test.go`
- `fabriccli/mergein_webster_integration_test.go`
- `fabriccli/pushbypass_integration_test.go`
- `fabriccli/remoteenvelope_integration_test.go`
- `//cmd/lyx/main_integration_test.go`
- `//cmd/lyx/stencilseed_integration_test.go`
- `hubforge/hub_test.go`
- `ideengine/spawn_clean_integration_test.go`
- `ideengine/spawn_driven_hub_integration_test.go`
- `loomcli/commitrecords_integration_test.go`
- `loomcli/parentreview_integration_test.go`
- `loomcli/smoke_test.go`
- `loomcli/wiring_commitstatus_integration_test.go`
- `loomcli/wiring_rework_integration_test.go`
- `landingshed/finalize_integration_test.go`
- `landingshed/publish_integration_test.go`

**Uses:**
- `gitkit/query.go`

**Intent:**
Convert the private git query and commit helpers outside fabricengine and the webster pair to `gitkit` (D4, #325 §1).

- `gitrepo`: `commitAll` and `runGit` (`gitrepo_test.go`), `writeAndCommit` (`gogit_test.go`) and `commitFile` (`parity_test.go`).
  `runGit` stays only at call sites that assert on its exit code or stderr as behaviour; setup calls move to `gitkit.Git`.
  The parity oracle in `oracle_test.go` is not touched here: it must stay independent of every helper (card 17 moves it).
- `fabriccli`: `gitLsFiles`, `branchExistsLocallyCLI`, `commitOnBranchCLI`, `commitOnCurrentBranchCLI`, `headSHA`, `branchSHA` and `remoteEnvelopeBranchExists`.
- `cmd/lyx`: the raw `gitexec.RunGit` repo seeding in the two integration files.
- `hubforge`: `hub_test.go`'s `runGit` (the production `mustGit` in `hub.go` is not a test helper and stays).
- `ideengine`: `commitFile` and `gitOut`.
- `loomcli`: `gitOut`, `mustGitOut` and `headSHA` in `wiring_commitstatus_integration_test.go`, and `weftCommitCount` and `mustGitSmoke` in `smoke_test.go`.
  `wiring_commitstatus_integration_test.go` is D9-protected: restructure only.
- `landingshed`: `commitOnCurrentBranchLanding`, `gitParentCountLanding` and `currentSHALanding`.

Every touched file is already `integration`- or `smoke`-tagged, so the D4 spawn rule holds; a package that newly matches the Hermetic Env scan gets a `testmain_test.go` calling `gitkit.HermeticGitEnv()`.
Assertions are unchanged; apply D10 to every touched test.

**ImpactSummary:** Tagged tests in seven packages lose their private git helpers to gitkit; no assertion or production code changes.

**Verify:** go test -tags smoke ./internal/loomcli/...
