```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1–F5 change set for correctness, running each changed branch and each added test against its defect.

**F1, MEDIUM-adjacent, rated LOW:** `internal/fabricengine/remoteleftover_test.go`, `TestErrRemoteLeftoverError_NamesTheDeleteOfTheFullOriginRef`. The test builds the error with `originRef` set by hand. If `probeLiveLocalWeft` stopped setting `originRef: ns.Branch(weftBranch)` (`remoteleftover.go:194`), it would still pass. The existing integration rows only check that the message contains the short branch name (`add_remoteleftover_integration_test.go:51`), which an empty or short ref also satisfies. So nothing fails if the set site is wrong. The review asked for a one-repository row on the integration case, and none was added. `hubforge.NewOneRepoHub` exists for it.

**F1, LOW:** `ErrRemoteLeftover` can be built outside the package, and `leftoverDivergedAdoptedWeft` is the zero `kind`. An external construction therefore takes the weft branch of `Error()` with an empty `originRef`. The `battencli` tests at `owner_readopt_test.go:135` and `wire_test.go:761-762` do this, but I found no `Error()` assertion on them, and production only constructs weft kind at `remoteleftover.go:194`. This is not a defect today.

**F2, no finding:** `failedRemoteSteps` now emits `refs/tags/<tag>:<ns.Tag(tag)>`. Run for the one-repository case, it gives `refs/lym/tags/<tag>` as the destination. For the identity case it gives `refs/tags/<tag>`. When `originNamespaceAt` errors, the push is omitted. The old `TestNoticeRemoteTeardown` rows still pass by substring, because `/weft` has no `.git` and reads as identity. The new integration test would fail against the old text in both rows.

**F3, no finding:** The `fmt.Errorf` has one `%w` and two `%s`, matched to `err`, `OneRepositoryBoardBranch` and `OneRepositoryBoardBranch`. It is untested, but the finding was not a coverage gap.

**F4, no finding:** The tag rows run correctly through `namesRef` (`pushretry.go:83`). Under the namespaced value, `refs/lym/tags/archive/s/abc` maps to `refs/tags/archive/s/abc` and the row is retryable. Under identity it is unmapped and the row is not retryable. A broken tag mapping fails them.

**F5, no finding:** It is a doc-only change.

Calls made:
- Read `internal/gitrepo/pushretry.go:40-110`, because my context lacked `retryablePushRefusal` and `namesRef`, which the F4 rows exercise.
