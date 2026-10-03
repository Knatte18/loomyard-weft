# Card 20 — retire the detached-checkout primitives

**Edit:**
- `internal/gitrepo/gitrepo.go`
- `internal/gitrepo/gitrepo_test.go`
- `internal/gitrepo/parity_test.go`
- `internal/gitrepo/doc.go`
- `internal/fabricengine/warpforward.go`
- `internal/fabricengine/warpforward_integration_test.go`
- `internal/fabricengine/fabric.go`
- `internal/fabricengine/doc.go`
- `cmd/lyx/gitrepoboundary_test.go`

**Uses:**
- `internal/websterengine/triage.go`

**Intent:**
D4 retires every detached checkout of the live worktree.
After card 17 deletes the integration bisect and the triage baseline, `gitrepo.Repo.CheckoutDetached`/`RestoreBranch` and their `Fabric` forwarders have no production caller.
Both package docs justify the pair only by that caller: `gitrepo`'s as its one admitted in-place checkout exception, and `fabricengine`'s as raw primitives left unguarded against a live merge record.
This card deletes the four methods so no raw, merge-record-unguarded detach stays exported for a future caller to pick up.
`CurrentBranch` stays, since `fabricengine/weftgit.go` and `loomcli/landingbranches.go` call it.

Deleted:
`CheckoutDetached` and `RestoreBranch` from `gitrepo.go` and `warpforward.go`;
`gitrepo_test.go`'s `TestBisectPrimitives_DetachRestoreCycle` and `TestCheckoutDetached_RejectsNonHexSHA`;
`warpforward_integration_test.go`'s `TestFabricWarp_DetachVerifyRestoreRoundTrip` and `TestFabricWarp_RestoreBranchInvalidRefErrors`, with the file header naming the remaining methods;
the two names from `gitrepoboundary_test.go`'s `gitrepoPinnedRunBoundMethods`, which pins the CLI-bound method set by equality and so must shrink with it.

Docs:
`gitrepo/doc.go` drops every mention of the pair (the runChecked method list, the "in-place bisect exception" bullet, the scope-boundaries trio and exception paragraph, and the go-git measurement note's "Pull and CheckoutDetached stay CLI-bound" clause), leaving `CurrentBranch` a plain read;
`fabricengine/doc.go` drops the "webster's integration bisect drives them" sentence from the attached-HEAD precondition and the "one knowing exception" paragraph about the pair;
`fabric.go`'s carve-out comment and `parity_test.go`'s `TestCurrentBranch_Parity` comment stop naming `RestoreBranch`.

Tests: none added; the deleted methods' tests go with them, and `gitrepoboundary_test.go`'s equality check pins the narrowed set.

**ImpactSummary:** removes four exported methods with no production caller once card 17 lands, so no runtime behavior changes; the reach is gitrepo, fabricengine and one cmd/lyx guard list.
