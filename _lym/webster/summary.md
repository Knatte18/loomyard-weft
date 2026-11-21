# One-repository hub layout in origin's refs/lym/ namespace

All four batches ended `done`.

## 01-gitrepo-refspec-upstream

The first attempt was rejected as `failed`, and a second recovery run brought it to `done` at head `e7b5e37a`.
The digest lists deviations outside the batch's own targets: `go.mod`, `internal/fabricengine/pushclassified.go`,
`internal/fabricengine/recordslayout_integration_test.go`, `internal/gitrepo/gogit.go`, `internal/gitrepo/gogit_test.go`,
`internal/gitrepo/push.go`, `internal/gitrepo/pushretry.go`, `internal/gitrepo/refnamespace.go` and `internal/gitrepo/worktree.go`.
It lists no departures.

## 03-namespaced-remote-reads

Cards 3 and 4 landed at head `78611358`.
Fabric's listings and fetch sources go through the records namespace.
`CloneHub` handles the one-URL check, refusals, join and seed, the clone gained a board-push verb, and `lym fabric clone` gained `--single-repo`.
Deviations: `internal/fabricengine/clone.go`, `internal/fabricengine/clone_onerepository_test.go` and `internal/fabricengine/destroy.go`.
No departures.

## 05-one-repository-builder

Cards 5, 6 and 7 landed at head `16cafa34`.
`hubforge` gained the one-repository builder, the clone's tests run over it, and the prime's records branch is read from `lym.primeRecordsBranch`.
The pair lifecycle is pinned in the namespace.
The fork also fixed `pushCloneSeed` in `internal/fabricengine/destroy.go`, outside the card targets, so a failed seed can restore a leftover records branch.
Deviations: `internal/fabricengine/clone_singlerepo_integration_test.go`, `internal/fabricengine/destroy.go` and `internal/fabricengine/onerepository_lifecycle_integration_test.go`.
No departures.

## 08-docs-and-pattern

Card 8 landed at head `c4127fc8`.
It adds `PATTERN-single-repo-hub` and its background file, and updates `PATTERN-code-conventions`, `docs/getting-started.md` and `README.md`.
No deviations, departures or renames.

## Scope guard

The scope guard reported only the deviations named above.
Batches 3 and 5 reported tiers 0 and 1 green, and tiers 2 and 3 did not run.
