Task: check the fix change set for consistency with its surroundings (lens fix-consistency).

Findings:

1. **LOW**, `internal/loomcli/smoke_drivingsurface_test.go` file header (lines 1–8). The header lists what the file pins: an llm-seeded start keeps one status strand, steps add or remove none, and the follow state. The new top-level subtest "start beside a runs strand adds the status strand in its own window" pins a fourth property that the header does not mention. A reader choosing a file from the header would not learn that it covers the status strand's window placement beside a runs strand. The header is the file's table of contents under the code-quality skill's file-header rule, so it should gain one line for this.

2. **LOW**, same file, the new subtest's placement. It is a sibling of "go driver" and "llm driver" inside `TestSmokeStatusStrandAcrossDriverSeeds`. The test's doc comment says it "drives the step verbs, and an llm-seeded start, over one pair per driver seed", but the new subtest uses its own fresh pair and no driver-seed pair. The doc comment is now slightly inaccurate.

Checked and consistent:
- `surfaceBeamUpWarnings` has two callers, `pawlwiring.go:110` and `pawlwiring_test.go:66`, and both use the new signature.
- The `Notify` dep and the Beam-Up step share the one `notify` closure.
- The `statusStrandPane` refactor onto `strandPane` leaves its callers unchanged.
- The `docs/overview.md` wording matches `internal/loomcli/doc.go:53` and `internal/orchcli/doc.go:13`.

Calls made:
1. `grep -rn "surfaceBeamUpWarnings(" --include=*.go .` and `git diff HEAD~4 --stat`: my context did not hold a repository-wide caller search or the committed set of changed files.
