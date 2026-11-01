# Fixer-report, round 1

## Changes

Commits 42f85d68 (F1-F6, one commit for all six since the edits overlap in rows.go) and a follow-up for self-check findings.

- F1: editToolsRule in internal/shellguard/rows.go names Read for reading and grep for searching, no awk, cat or sed -n; internal/editdirective/doc.go and settings-golden.json updated (literal replace, no update flag exists). docs/overview.md holds no sed -n or awk text, so nothing changed there.
- F2: edit-directive.md states the override of harness text and names Read and grep; editdirective/doc.go mirrored; editRulePhrases still match.
- F3: documentation branch, in the shell-edit row comment and claudeengine/doc.go.
- F4: documentation branch, in the sourceFilePattern comment and edit-directive.md (added in the follow-up).
- F5: deny rows for >| and every sourceFilePattern extension, pass row for a quoted target.
- F6: shell-edit deny and allow rows in hookcmd_integration_test.go.

Gate: lym gate test tier 0 and tier 1 green on the touched packages.

## Self-check

- fix-consistency, LOW, F4 twin in directive: fixed (directive and editdirective/doc.go).
- fix-proposal, LOW, same F4 gap and loose wording: fixed ("those three extensions").
- fix-consistency, LOW, `sed -n` named in the override sentence: not changed; it is F2's proposed wording, naming it as text to override.
- fix-proposal, LOW, F3 looped spellings still pass: not changed; the review offered documentation as the option, and the row comment and claudeengine/doc.go state the gap. Fixed as proposed.
- fix-proposal, LOW, F1 Read named twice: fixed (steer now ends "search files with grep").
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Gate repair

The gate failed `TestGofmt_ModuleFilesAreFormatted`: the F5 edit to `internal/shellguard/rows_integration_test.go` dropped the space after `"sed_read",` in the `sed_read` row.
`lym loom fmt` restored it, the tier 0 run of `./cmd/lym` and `./internal/shellguard` passes, and the fix is its own commit.
