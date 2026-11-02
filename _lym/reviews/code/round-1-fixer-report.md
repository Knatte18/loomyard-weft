# Fixer-report, code review round 1

## Changes

- F1: new `internal/shuttlecli/hookbatching_test.go`. `TestHookBatchingVerb` runs `hookBatchingCmd` through cobra with fixture stdin: a fired nudge (one line of hook output plus a newline), no breach (empty), an unknown engine, a malformed input and a corrupt state file (each reported on stderr, nil from `RunE`). `TestHookBatchingVerb_RunsOutsideAHub` runs the verb through `RunCLIIn` in a directory that is no git repository and pins the `PersistentPreRunE` skip. Fixed as proposed, plus the two rows the self-check named.
- F2: `writeNudgeTranscript` in `batchingnudge_test.go` now writes a message over several lines (`~`), a text block, a sidechain line and a line with no message id. New rows pin the merge by message id and the skips.
- F3: `TestReadTranscriptTail` over `readTranscriptTail`, which already takes its budget: under the budget, over it (first line dropped), and a window with no newline (nil).
- F4: `sliced` rows for limit alone, offset past 1 alone, offset 1 alone and neither (the bool became `offset` and `limit` pointers); `read_slice` and `edit_chain` off rows; lookup chain outranking the read slice; a multi-Edit message; an Edit plus Read message; a file at exactly `slice_lines`; an unterminated last line. The slice-versus-edit-chain order cannot be pinned, because one call cannot be both a sliced Read and an edit message.
- F5: rows for a malformed payload, empty `transcript_path` and `tool_use_id`, an unreadable transcript, a `tool_use_id` absent from the tail and a corrupt state file. Fixed with my own change on the bounds: the `fix` key proposed turning the refusal rows of `TestPrepare_BatchingNudgeHook` into a bounds table; I added `TestValidateBatchingNudge` over the exported `ValidateBatchingNudge` instead and kept the Prepare row, which pins the separate "refused before any artifact" behaviour.

Commits: F1 as `shuttlecli: fix F1 …` (two commits), F2 to F5 as one `claudeengine: fix F2-F5 …`. F2 to F5 share one file that I rewrote in one pass, so I did not split them.

Gate repair: the gate found the file header of `batchingnudge_test.go` wrapped at a fixed column, so no test ran. I broke it at sentence boundaries, one sentence per line; no other line changed. The repair is committed as its own `claudeengine: fix gate finding …` commit, after a gate finding that the first edit had been left uncommitted.

Gate repair, tier1: the verify run failed because `internal/fabricengine` tier1 hit the 10-minute test timeout (`panic: test timed out after 10m0s`, many git-spawning tests still running). No file of that package is in this branch's diff, which touches only shuttlecli, claudeengine, shuttleengine and shuttleengines. I changed no code for it. Rerunning `lym gate test ./internal/fabricengine --tier 1` passed: 1600 passed, 0 failed, 494s wall, so the timeout was load, not a defect of this change.

Gate: `lym gate test ./internal/shuttlecli ./internal/shuttleengine/claudeengine` passed tier0 before the self-check; `./internal/shuttlecli` passed again after its follow-up.

## Self-check

- fix-consistency, LOW: F2 to F5 in one commit. Not changed; the findings share one file written in one pass.
- fix-affected-set, LOW: the `PersistentPreRunE` skip is unpinned. Fixed with `TestHookBatchingVerb_RunsOutsideAHub`.
- fix-affected-set, LOW: no row for the engine-error path (corrupt state). Fixed with the `CorruptStateIsReportedAndExitsZero` row.
- fix-proposal, LOW: the same `PersistentPreRunE` gap. Merged with the one above; fixed.
- fix-code-correctness, LOW: the empty `transcript_path` and `tool_use_id` rows pass with their guard removed. Disputed, ground `every fix worse`: the guard and the fall-through to `errNoNudge` give the same observable result (silent, state unchanged), so a row that told them apart would assert internals.
- All four forks returned a report.

## Disputed

- fix-code-correctness LOW, empty-field rows, ground `every fix worse`: evidence in the self-check line above (`BatchingNudge` in `batchingnudge.go`: an empty path fails `readTranscriptTail` into `errNoNudge`, and an empty id makes `messageHolding` return -1).

## Deferred

Nothing deferred.
