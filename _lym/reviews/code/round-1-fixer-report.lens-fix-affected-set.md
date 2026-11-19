no findings

- F1: `prReviewSendBackEdge` has two references, its definition and the command seam at `prfindings.go:343`; no test calls it directly.
- F2: the only sites describing which paths pass through `wireLanding` are `sharedbootstrap.go:438` and `doc.go:42`, and they agree.
- F3: the finding names only `prfindings_integration_test.go`, which no longer calls `AddPair`.

Tool calls: 1, a grep per finding over `internal/loomcli` and the repo.
