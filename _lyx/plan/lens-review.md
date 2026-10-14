# Lens review — run-window-bugs plan

Every fork returned a report.

- criteria-coverage, MEDIUM, D1 Started not asserted persisted after an events-ready start: fixed, card 1's `wantStarted` branch loads `run.json` and asserts `Started`.
- criteria-coverage, MEDIUM, D2 no-skills color branch unreached: fixed, card 2 adds "ALaunchWithoutSkillsOrPromptLineIsColoredAfterStartup".
- criteria-coverage, LOW, D3 production wiring of `Resized` untested: rejected, the two callers are one-line field assignments, and the wake path is pinned in statuspane.
- criteria-coverage, LOW, D6 no row with two refused files: fixed, card 6 adds a two-refusal row.
- criteria-coverage, LOW, D6 no symlinked-root row: rejected, root and paths go through the same `EvalSymlinks` and the escaping-symlink row pins the comparison.
- criteria-coverage, LOW, D7 record checked only below `Topology.Remove`: rejected, `Remove` passes its own `rec` straight through, as the existing fallback tests rely on.
- criteria-coverage, LOW, D7 Windows `ERROR_DIR_NOT_EMPTY` unreached: rejected, a refilled directory cannot be produced deterministically; the sharing-violation row covers the Windows retry path.
- criteria-coverage, LOW, D6 missing-path refusal unlicensed: rejected, it follows from D6's resolve-before-check rule (now with a row).
- test-economy, LOW, card 3 shorter-frame row repeats the exact-bytes check: fixed, the redundant substring assertions are dropped and the row keeps the criterion's shape.
- test-economy, LOW, card 8 new top-level test: rejected, the lens itself found no duplicate; its budget raise is listed.
- forced-edits, LOW, stale claudeengine comments (`maxLaunchPromptBytes`, `launchPointer`, `buildLaunchCmd`, `skillload.go` header, `TestPrepare_PromptLaunchLimit`): fixed, all listed on card 2 with their corrections.
- forced-edits, LOW, stale color row name and `TestStartup_NeedsSessionStartLine` doc: fixed, card 2 renames the row and card 1 updates the doc comment.
- forced-edits, LOW, `fabricengine/doc.go` dirtiness-window sentence names two fallbacks: fixed, card 8 adds the leftover removal as a third.
- forced-edits, LOW, `pattern/PATTERN-daemon-wakeups.md` silent on the resize wake: rejected, the background's bound still holds and a resize only shortens a wait.
- forced-edits, LOW, `Topology.Remove` doc silent on the leftover removal: fixed, card 8 adds the sentence.
- row-reachability, MEDIUM, card 6 missing-path and named-`.go`-resolving-to-non-`.go` refusals unreached: fixed, card 6 adds both rows.
- row-reachability, LOW, card 6 mode row fails on Windows: fixed, the row compares against the mode read before the call.
- row-reachability, LOW, card 1 events-before-capture order unpinned: fixed, the row asserts `CallLog` holds no `CapturePane`.
- row-reachability, LOW, card 8 Windows error value unverified: fixed, card 8 says a different observed value fails the row and is reported, never silently admitted.
- row-reachability, LOW, card 8 already-gone leftover branch unreached: rejected, it returns git's error unchanged into today's not-a-linked-worktree message, the branch the gate-refused row's message shape already pins.
- row-reachability, LOW, card 2 new Prepare rows lack `wantErrContains`: fixed, they name "launch limit" and "line break".
- row-reachability, LOW, card 3 new row lacks waits and reports: fixed, `[1s, 1s]` and `[3, 1]`.
