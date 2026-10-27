---
round: 1
focus:
  - "`internal/fabricengine/recordsgit.go` `commitWeftAt`: it sits under `Bolt.Commit` but is also the weft commit path of other callers; settle whether seeding excludes there changes behavior for any caller beyond Bolt, and whether the seed error is wrapped or returned bare compared with its sibling errors."
  - "`seedWeftArtifactExcludes` against a repo whose gitdir is a worktree's common gitdir or has no `info/` directory yet (the unseeded fixture in `bolt_integration_test.go`): settle that the seed works there and is idempotent across repeated `Commit` calls."
  - "`internal/fabricengine/clone.go` comment rewrite: settle whether the new text still states the ordering reason for seeding before `WireJunctionsWith` truthfully, and whether the fatal-failure rationale it now gives matches the code."
  - "`internal/fabriccli/hubmutation_integration_test.go`: settle that `RunCLIIn(h.BoardDir(), …)` resolves the test hub's own records and warp, that the `t.Cleanup` reset runs on every path including a parallel subtest, and that the `logger.TraceDir()` assertion checks the scenario's goal rather than restating the setup."
  - "`bolt_integration_test.go`: settle that dropping the push-lock removal is covered by the seeded exclude of `.gitrepo-push.lock`, and that the `LsFiles` assertion does not duplicate an existing test's coverage."
  - "Docs: settle that the new `doc.go` sentence and the `bolt.go` and `recordsgit.go` comments agree with each other and with what `commitWeftAt` does, and that no other doc still says Bolt's commit seeds nothing."
  - "Scope against the decision record: settle that the diff touches nothing beyond D1 and D2 (no logger, `boardengine` or bench-file change)."
---

Seeded from the decision record's D1 and D2 and the committed diff; no prior round exists.
