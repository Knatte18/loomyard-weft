No findings.

The searches for F1, F2 and F13 now find each name declared once, and the F13 verb lists agree.

- **F1:** outside tests, `"CLAUDE.local.md"` and `"AGENTS.override.md"` are spelled only at their declarations in `internal/heddleengine/target.go:30` and `:34`. The `claudeengine` and `codexengine` sites use the constants.
- **F2:** outside tests, `"claude"` and `"codex"` are spelled only at their declarations in `internal/heddleengine/target.go:24` and `:25`.
- **F13:** the verb list at `docs/overview.md:1982` now matches the module entry at line 640. Line 640 also names `await-forks`, which line 1982 never did, so that difference predates this fix.

Calls made:
1. Grep for the two file-name literals in non-test Go under `internal` and `cmd`, to cover F1's set.
2. Grep for the two engine literals in non-test Go under `internal/heddleengine`, to cover F2's set.
3. Grep for the shuttle verb lists in `docs` and `README.md`, to cover F13's set.

All three were one combined Bash call, because my context held no search results for these sets.
