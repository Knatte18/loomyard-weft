---
verdict: APPROVED
rationale: "All five round-1 findings are legitimate and fixed at HEAD 639596451 (gofmt -l clean over the range, no references to the deleted oracle symbols, gitkit and hubforge docs match their exports, remaining wrapped comments all exist verbatim at merge base 518dc7400); card 17's Delete group passes assert-no-callers, and build, vet (plain and smoke), go test and the integration tier all pass."
---

# Webster-Bouncer verdict — round 1

The review range is correct.
`_lyx/shed/test-suite-change-amplification/status.json` names `product.parent` `main`, and no `_lyx/loom/rework/` exists, so the subject is `git diff $(git merge-base main HEAD)..HEAD` from merge base `518dc7400`.

The report has no `## Focus departures` section, so there is no departure to ratify or reject.

## Findings judged

None of the five findings is over-flagged.
Each one names a defect in the diff that the diff alone can fix, and none falls under `Do not flag`.

- **F1 (MEDIUM): comments.** Legitimate.
  CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks on new and changed Go comments.
  At HEAD, a detector run over the range's added comment lines still finds mid-phrase breaks, but every hit exists verbatim at `518dc7400`: the `clone_hubscratch_integration_test.go`, `export_integration_test.go`, `loomrecipe/fixture_test.go`, reedengine and `shuttleengine/wait_test.go` comments.
  Those are moved legacy comments, which the convention exempts.
  The new breaks in `gitkit/doc.go` and `hubforge/doc.go` fall at a semicolon or before an independent `so`/`and` clause, which the convention allows.
  Resolved.
- **F2 (LOW): gofmt.** Legitimate.
  `gofmt -l` over every added or modified `.go` file in the range now reports nothing.
  Resolved.
- **F3 (LOW): stale oracle names in failure messages.** Legitimate.
  The plan-spec itself names this string-reference blind spot of `assert-no-callers`.
  Nothing in the tree matches `oracleCurrentSHA`, `oracleSHAExists`, `oracleChangedFilesSince`, `oracleCurrentBranch` or `errOracleNoCommits`.
  The `oracleGot`/`oracleErr` hits are local variable names.
  Resolved.
- **F4 (LOW): gitkit doc.** Legitimate, because the diff's new paragraph contradicted the old export lists.
  The doc now opens with the helper groups and says every export but `CopyRepo` is unpinned.
  Resolved.
- **F5 (LOW): hubforge doc.** Legitimate.
  The "only exports" sentence now lists everything `hub.go` and `seed.go` export: `Hub`, `TestShortname`, `NewHub`, `AddPair`, `AddPairWith`, `OpenFabric`, `SeedConfig` and `SeedFabricConfig`.
  Resolved.

## Per-card mechanical check

Card 17 holds the plan's only Rename, Delete or Move group.
That `Delete` group targets `gitrepo/oracle_test.go`.
The file is gone, and `assert-no-callers` over its five former symbols returns zero hits.
`checkedCallPinnedRawSites` gains exactly one row, `"internal/gitrepo/internal/gitoracle": 4`.
Every other group is `Create` or `Edit`.

## Independent evidence at HEAD

`go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all exit 0.
