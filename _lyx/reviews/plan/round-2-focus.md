---
round: 2
focus:
  - "Card 15 (`_lyx/plan/15-webster-verify-gate.md`), closure step 3: the fixer-commit check walks the first-parent chain from the pre-fix head to HEAD, accepting non-merge commits and clean parent merges, judged by `parentMergeRejection` in `internal/websterengine/gitwrap.go`. Both `parentMergeRejection` and `resolveParentTips` take a `*gitrepo.Repo`, and `gitrepo.New` is the only constructor. `cmd/lyx/rawgitmutation_test.go`'s untagged `TestNoRawGitMutation_WebsterProductionSource` fails any websterengine production file containing `gitrepo.New(` except the allowlisted `gitwrap.go`. Settle whether card 15 now targets `gitwrap.go` (or otherwise names where the walk lives) so that the new `verifygate.go` never constructs a repo itself."
  - "Cards 15 and 17 (`_lyx/plan/17-retire-integration-stage.md`): outside `gitwrap.go`, the gitwrap helpers `integrationFixHeadRefusal`, `commitsBetween` and `commitParentCount` are used only by `internal/websterengine/integrationfix.go`, which card 17 deletes. `cmd/lyx/rawgitmutation_test.go`'s header comment names `FabricBisector`, which card 17 also deletes. Settle whether some card drops every helper left without a caller and updates that comment."
---

Both entries concern `internal/websterengine/gitwrap.go` and the raw-git guard over websterengine.
The plan's other cards were checked in round 1 and need no new focus.
