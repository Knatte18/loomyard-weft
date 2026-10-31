```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F4 change set for stale siblings and twins left unchanged (lens fix-consistency).

**LOW — `internal/adoptcli/deps_test.go` file header is stale**
- Changed text: the header reads "covers the hub-only pause reading ... the adopt check start's guard under go test, its alive skip and its liveness reading."
- The fix added `TestAdoptGateTiers` to this file, but the header does not mention the adopt gate tiers reading. Under the code-quality file-header rule, header and filename should tell a reader what the file holds.
- Evidence: the F1 commit touched only the test body and imports, not the header line.

**LOW — `checkFixtureNow` comment in `internal/adoptcli/check_test.go` no longer matches the clock**
- Changed text: `// checkFixtureNow is the time a fixture's clock reads.`
- The fixture clock now advances one second per read through `tick` and `checkFixtureAfter`. `checkFixtureNow` is only the time of the first read, so the comment is wrong. `TestRunCheckMarks` still uses `checkFixtureNow` for the starting mark, which stays correct because the check's start is read 0.
- Evidence: `tick` returns `checkFixtureAfter(f.clockReads)`, and `checkFixtureAfter(0)` equals `checkFixtureNow`.

No other inconsistencies:
- The `Long` help text in `cli.go` ("suite with message, or suite_warning") still matches the fields `adoptFields` builds.
- The `outcome` struct comment matches.
- The `adoptFields` extraction moved the field set unchanged, so no sibling differs from it.
- The F4 assertion matches the `bar.go` doc, "check mark ... before the clock".
- The `TestRunCheckMarks` comment, "tier n starts at read n+1", matches the `checkFixtureAfter(1)` and `checkFixtureAfter(2)` rows.

Tool calls made: none, because my context already held every changed file.
