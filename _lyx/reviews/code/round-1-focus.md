---
round: 1
focus:
  - "docs/overview.md, gate bullet: the merge of parent `main` kept the task's added sentence on the `--card` vet beside the parent's other edits to the file. Settle whether the sentence still matches what `internal/gatecli/doc.go`, `scope.go` and `tiers.go` do (scoper `go` only, other tier tags that add a file to a card package, a failed vet stopping no later tier, vet or guard run)."
  - "Settle whether the merge left the amended side consistent with the board entry: a card gate under `--card` vets the card's packages under the other tag sets, and nothing the parent brought in changes `internal/gatecli` or its tests in a way that breaks that."
---

The amendment is narrow: the only file both sides changed is `docs/overview.md`, and the parent's edits there are link retargets away from the gate bullet.
