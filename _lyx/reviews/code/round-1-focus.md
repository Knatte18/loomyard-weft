---
round: 1
focus:
  - "internal/adoptcli/adopt.go: does the suite seam run once per call, after the dobbies, and only when an owner was recorded for an entry or the prime? Settle that a refused adopt or a call with no recorded owner never reaches it."
  - "internal/adoptcli/deps.go: does the hub seam spawn `lyx gate test --gate finalize --wait` detached via `proc.Detach` in the prime worktree from `battencli.ResolveBattenPrime`, truncate the log at each start, log the spawn through `internal/logger`, and start nothing under `go test`?"
  - "Failure path: does a failed spawn set `suite_warning` naming `lyx gate test --gate finalize` and leave every slug's outcome and the exit status unchanged? Are the `suite` pid and log fields in the envelope set correctly?"
  - "internal/lyxdirs/dirs.go and deps.go: is the log path built only through the new `lyxdirs` token, under the board-level `.lyx` directory, and never tracked?"
  - "Docs in the same diff: do the `lyx adopt` long help in cli.go, the `internal/adoptcli` doc.go and docs/overview.md describe the new step accurately and match the code?"
  - "Tests in adopt_test.go and deps_test.go: is each acceptance criterion covered once through the existing `adopt` table with a faked seam, without duplicate coverage or per-helper tests?"
---

Seed focus from the decision record's scope and acceptance criteria; no prior round exists.
