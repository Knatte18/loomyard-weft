---
round: 3
focus:
  - "Decision D7 in `_lyx/discussion/decision-record.md` lets an operator list any skill, a build skill included, in the `language` module's list, and its Rationale states a bound on that. Settle whether the stated bound, and the second `## Open risks` bullet that repeats it, match what the agents' session deny rules in `internal/shuttleengine/claudeengine/settings.go` actually refuse, including commands such as `goimports -w`, `golangci-lint run`, `go install ...@latest`, a package-scoped `go test` and `dotnet build` or `dotnet test`, and whether the Decision states what the hatch lets through and what bounds it."
  - "Decision D4 in `_lyx/discussion/decision-record.md` says which component loads the `language` module and which is told the list, and names `landingshed` as the merge conflict session's constructor. Settle whether its sentences agree on where the config load happens, so a plan writer cannot place the load inside `landingshed` when the wiring in `internal/loomcli/wiring.go` builds its deps."
---

The two entries are the sites changed after round 2; everything else in the pair stands as reviewed.
