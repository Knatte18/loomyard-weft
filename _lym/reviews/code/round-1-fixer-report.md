# Fixer-report, round 1 (code)

All nine findings fixed; the tier0 gate over `./internal/shuttleengine/...` passed after the final change.

## Changes

- **F1** `docs/overview.md`: the Codex bullet now states the `*`-matcher question deny on autonomous launches and the `*`-matcher `PostToolUse` batching hook, as proposed.
- **F2** `command_test.go`, `guards_test.go`, `guards_run_test.go`: a `//testtiming:keep` marker above each of the three tests, with the reasons the review gave.
- **F3** `batchingnudge.go` `localPath`: strips the leading slash of a file URL drive path.
  Fixed differently from the proposal: the strip runs on Windows only, because on other systems `/c:/x` is a rooted path and stripping it would make it relative (self-check, fix-proposal lens).
  `TestLocalPath` pins it, with the drive-path row's expectation following `runtime.GOOS`.
- **F4** `TestBatchingNudge`: rows `UnreadableRolloutIsSilent` and `PayloadWithoutAToolUseIDIsSilent`.
  The second row leaves the last item out of the rollout, so it fails without the `tool_use_id` guard (self-check).
- **F5** `TestBatchingNudge`: a `configure` field and one row per switch (`LookupChain`, `ReadSlice`, `EditChain`) switched off.
- **F6** `prepare_test.go`: `TestLoadConfig_AcceptsBatchingNudgeEdges` with an off block holding `lookup_run: 0` and `context_tokens: 0` with the block on.
- **F7** `filetail_test.go` row `StopsAtTheLimit`.
  Fixed with an extra change: the row failed because `CountLinesUpTo` returned 5 for limit 2, as it counted a whole read chunk before checking the limit.
  `filetail.go` now returns `min(lines, limit)` on both returns and its doc says so.
  Both callers only compare the count against the limit, so they are unchanged in behaviour.
- **F8** `TestBatchingNudge` fixture: `plainCwd`, `absoluteReads`, `directArgv` on a nested call and `after` on a call, with rows for a plain cwd, an absolute read path, a non-shell argv and an item after its cell's output.
- **F9** `bashlookup_test.go`: row `HeadLongCountSeparate`.

Each finding is its own commit, except F4, F5 and F8, which share one commit because they share one test file, and the self-check fixes, which are one more.

## Self-check

- fix-code-correctness, LOW: the `tool_use_id` row passes with the guard removed. Fixed as proposed (`itemMissing: true`).
- fix-proposal, LOW: the same row, merged with the one above.
- fix-proposal, LOW: the drive-path strip runs on every OS. Fixed as proposed (gated on `runtime.GOOS == "windows"`).
- fix-consistency: no findings.
- fix-affected-set: no findings.

Not covered: no row pins an empty `transcript_path`, which the same guard also checks. The review did not ask for one.

## Disputed

None.

## Deferred

None.

## Rulings

None.
