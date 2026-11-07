---
round: 1
focus:
  - "internal/hubforge/template.go `packRepositories`: the record names `git repack -ad`, the diff uses `repack -d --cruft`. Settle whether the departure carries a `Pattern-Departure:` trailer or is covered by the amended record, and whether any git version this repo supports lacks `--cruft`."
  - "internal/hubforge/template.go `gitDirsUnder`: settle whether it finds every repository of a template (bares, clones, worktree gitdirs, a `.git` file pointing elsewhere) and whether a repository left unpacked would still hand a copy loose objects or loose refs."
  - "internal/hubforge/template.go `requireUnpoisoned`/`requireIntact`/`copyTemplate`: settle that CopyHub and CopyRegistered check only the poisoned flag, and that SharedHub still walks the fingerprint before and after the test and poisons on a change."
  - "internal/hubforge/template.go `commitRelocatedBinding`: settle that `commit --amend -- <file>` stages the relocated binding, leaves the board clean, and keeps HEAD, branch and tracking-ref reflog entries that `internal/fabricengine/reflog.go` reads."
  - "Packing happens before the fingerprint: settle that a SharedHub taken right after the build sees no change, and that relocation never needs to rewrite a pack or `packed-refs`."
  - "internal/hubforge/template_test.go and hub_test.go: check each acceptance criterion (board HEAD equals upstream and the records bare tip, newest tracking-ref reflog names that SHA, clean board for shape and committing registration, no loose objects after build, SharedHub write poisons and a later CopyHub fails) is covered, and apply the test-economy bullets to the new tests."
  - "internal/hubforge/doc.go and the template.go header comments: check them against the Go comment conventions of the repo's constraints document and against the behavior the diff leaves; check `_lym/landing/summary.md` claims against the diff."
---

Seed focus for round 1 of the code review of `46eae006..HEAD`, covering the hubforge D3 change.
