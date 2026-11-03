```yaml
lens: fix-consistency
outcome: reported
```

No findings against the fix, but two stale descriptions of Land's behavior are left behind. The first is a `LOW` finding. The second is a `NIT`, which falls below the rubric's severities, so I'm noting it only.

- **LOW, `internal/battenshed/doc.go:183-196`.** The package doc's ordered list of the Land row's steps does not mention the new first step. That step reads `ChildLanded` and, when it is false, skips the prime check, pull, deploy and notice and only prunes the board. `internal/battenshed/land.go` and `internal/shedrecipe/entries_batten.go` were updated; this description of the same row was not. The line "it checks the prime, where a child whose parent branch is not the prime's branch skips the pull and deploy" now reads as if the prime check always comes first.
- **NIT, `docs/overview.md:714`.** It says Land "pulls the prime when the child's parent branch is the prime's branch". It never says a child that finished no Finalize is skipped. It is the same gap, in the user-facing doc.
- **NIT, `internal/battenshed/land.go:98-99`.** The `NewLand` comment says "Under the land lock it pulls the prime, runs the deploy command and prunes the board". It does not carry the new skip. The `Call` comment and the file header do.

The twin-site checks were consistent:
- Every `LandDeps{` literal (`internal/battenshed/land_test.go`, `internal/shedrecipe/fixture_test.go`, `internal/battencli/run_test.go`, `internal/battencli/land.go`, `internal/testkit/envkit/envkit.go`) now sets `ChildLanded`.
- The `entries_batten.go` doc comment and its seam list agree.
- The `entries_batten_test.go` zero-seam row and the `loomcli` `intentionallyNil` entry were added.

Tool calls made:
1. `grep` for `LandDeps{` literals and docs mentioning Land. My context lacked a repo-wide search for twin fillers and descriptions.
2. `sed` of `internal/battenshed/doc.go:178-200` and `land.go:96-100`. My context lacked `doc.go` and the post-edit `NewLand` comment.
