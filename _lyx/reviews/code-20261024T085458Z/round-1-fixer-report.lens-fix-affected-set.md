```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I reran the search for each finding whose premise is a set (F1, F4 and F5). The fix covers every member I found. I did not recheck F2, F3 and F6, whose premises are not sets.

- **F1 and F2:** `internal/loomcli/decision.go` has one `json.NewDecoder` in `internal/loomcli/decision.go` and no `More()` call is left in `internal/loomcli/decision.go` or `internal/discussionparser`. The `--from` file has no other decoding site that needs `DisallowUnknownFields()` or the trailing-data check.
- **F4:** The `AppendDecision` references outside `doc.go` are in `identity.go`, `patternchanges.go` and `append.go`, plus their tests and `decision.go`. I did not open them, since the finding names only the package doc and it now mentions `AppendDecisions`.
- **F5:** The only remaining "required" statement for the per-entry flags is the fixed sentence at `internal/loomcli/decision.go:876`. `decisionWayFromAlone` and the `contracts/` stencils do not claim the flags are always required. The `contracts/stencils/bouncer/bouncer-template-escalation.md` one-entry example is correct as written.

Calls made:
1. `grep` for "are required" and `--title, --decision` across `.go`, `.md` and `.yaml`: to find other statements that the per-entry flags are required (F5).
2. `grep` for `decoder.More()`, `json.NewDecoder` and `AppendDecision`: to find other decode sites and the other doc mentions (F1, F2, F4).
3. `grep` for `decision add` in `doc.go` and `.md` files: to find other `decision add` descriptions that could be stale (F5).
