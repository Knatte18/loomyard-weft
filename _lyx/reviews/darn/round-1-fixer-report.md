# Fixer-report, round 1

## Changes

- F1 (`03a7d97b`): `internal/shuttleengine/claudeengine/doc.go` now describes the python, raw-go, in-place-edit and computed-rm rows as quote-aware, names the prebuilt-binary variable, and has a new redirect-write paragraph (always installed, what it refuses and allows, guardrail limits). Fixed as proposed.
- F2 (`c6a1b6b7`): fixed with the review's alternative, documenting instead of changing the patterns. `rows.go` (`commandPositionPattern`, `redirectWriteCommandPattern`, `rawGoCommandPattern` comments) and `internal/shellguard/doc.go` state that a command substitution inside a double-quoted string passes.
- F3 (`c6a1b6b7`): fixed with the review's alternative, documenting. The same comments and `doc.go` list a `<` before a quote or backslash as a pass-through, and `doc.go` states that nothing after a heredoc operator's line is read.
- F4 (`9fa3dccd`): `rows_integration_test.go` — `payloadDenied` takes a description field; a description table in the same test function pins a command word in the description for all six command-shaped rows (allowed); new rows for python backtick, `$(` and subshell denies, a redirect-write single-quoted target and a `/devx` lookalike. No new test function.
- Self-check follow-ups (`34dc00fc`): the `<<<` pass-through wording is narrowed to `<` before a quote, the engine doc states the shared pass-through limits once for every command-shaped row, and "one place" is dropped from the redirect-write paragraph.

## Self-check

- fix-consistency, LOW, `<<<` pass-through overstated: fixed as proposed.
- fix-code-correctness, LOW, `<<<` pass-through overstated (same finding): merged, fixed.
- fix-proposal, LOW, `<<<` pass-through overstated (same finding): merged, fixed.
- fix-proposal, LOW, "one place to redirect into" contradicts the allowed `/tmp/` and `/dev/`: fixed as proposed.
- fix-affected-set, LOW, per-row comments omit the F2/F3 pass-throughs: fixed with my own change. The shared limits are stated once for every command-shaped row in the engine doc and on `commandPositionPattern`, not repeated in each row's comment, so no row comment is edited.
- fix-affected-set, LOW, engine doc paragraphs omit the F2/F3 pass-throughs: fixed by the same shared sentence.

All four forks returned a report.

## Disputed

None.

## Deferred

None.

## Verification

`lyx gate test --tags integration ./internal/shellguard` passed after F4. The later commits changed comments only; `go vet` on both packages is clean.
