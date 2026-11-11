---
round: 1
focus:
  - "internal/adoptcli/check.go and adopt.go: does the adopt check's tab now close or shrink when the check ends? Settle whether a failed check keeps it open and whether the suite log stays readable after a close."
  - "internal/adoptcli/check_test.go and adopt_test.go: do the changed tests still pin the tab-lifecycle behavior, and does any new test duplicate an existing one?"
  - "internal/reedcli/cli.go, attach.go, up.go: do `lym reed attach` and `lym reed up` refuse a directory that is not a fabric worktree of the hub, such as `_bolt` or a weft checkout? Settle whether the refusal names the prime or a worktree to attach to, and whether a legitimate worktree or the prime is still accepted."
  - "internal/reedcli/cli_test.go: does TestRunCLI_AttachAndUp_RefuseFabricCheckouts cover both verbs and the accepted path without duplicating an existing test? It has no coverage map, so judge it by the test-economy bullets."
  - "internal/adoptcli/doc.go and internal/reedcli/doc.go: do the doc changes match the new behavior, and do the repository's rules require any other doc to land in the same diff?"
  - "Scope against the Goal: does the diff make any change beyond the two faults, the lingering adopt-check tab and the reed attach/up refusal?"
---

The decision record holds the Goal alone, with two imported notes; the other sections are empty.
