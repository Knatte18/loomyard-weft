No findings.

Task: lens fix-affected-set. I reran the premise-set searches for F2, F3, F4, F5 and F6.

- **F2 (block name repeated):** The only `"edit-directive"` literal in Go is `internal/editdirective/editdirective.go:18`. The duplicate in `stencilkit` is gone.
- **F5 (`driverPrompt` references):** No reference to the old name remains outside archives. The hits are `docs/research/.../calls.tsv`, an old research log, and the `_lym/reviews` copies, which I excluded.
- **F6 (shared-directive wording):** No "shared edit directive" text remains in `contracts` or `internal`. The one hit is `docs/research/claude/code-mode-bench-2026-11-16/report.md`, a measurement report.
- **F3 (`TidyShuttle` implementers):** No `EngineName(...) string` implementation remains in `dobbyengine` or `beamcli`.
- **F4 (producer coverage):** The producers the review named all call `RequireEditDirectiveOf`, except `seatengine`, which the fixer reverted deliberately. The other test files that assert `PromptEngine` are outside the review's list: `websterengine` planfixer, verifyfixer, runlevel and recoverbatch, `shuttleengines`, `orchcli` start, `loomcli` driverspec, and the `gatefixer` spawn and fixture tests. Their prompts come from producers already covered, so I raise nothing.

F7–F10 are not set-premise findings, so I left them unchecked.

Calls:
1. One Bash call with seven greps. It covered the stale `driverPrompt` name, the "shared edit directive" wording, `"edit-directive"` literals, `RequireEditDirectiveOf` users, `PromptEngine` test files and `TidyShuttle` `EngineName` implementers. My context did not hold repo-wide search results.
