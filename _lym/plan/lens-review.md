# Lens review

Every lens fork returned a report; none is missing.
Duplicates across lenses are merged into one line, naming each lens that raised it.

- guard-reach, BLOCKING — card 10's re-exec test recreates `.lym/beam` through `state.ReadJSON`'s mkdir: fixed differently; the proposed `ReadJSONNoMkdir` was not used, since it still takes a read lock whose file it would create in an existing directory and fails on a missing one; instead card 7's `ReadsAsReexec` stats the record's directory first and reads false creating nothing, with a `TestReadsAsReexec` row asserting the directory stays absent.
- way-forward MEDIUM, guard-reach MEDIUM, coverage MEDIUM — card 10 rotates `dobby.out` before `Run` holds the lock, and no row reaches the rotation: fixed as proposed; the rotation and reopen run inside the `Locked` callback, and the tier1 binary test seeds `dobby.out` past 1 MiB and asserts `dobby.out.1` after the start.
- coverage MEDIUM, guard-reach MEDIUM — card 9's heartbeat sits in `drivePasses`, which a standing move or a failing runs read skips: fixed as proposed; `loop` writes it after the first tick past the interval whatever that tick did, `heartbeat` reads `Deps.Runs` itself, and `TestRun_Heartbeat` gains a standing-move row and a failing-runs row.
- coverage MEDIUM, row-validity MEDIUM — `checkAlive`'s start-ticks comparison is untested, and the differing-ticks rows of `TestEnsureCheck` and `TestCheckView` repeat the dead rows: fixed as proposed; card 14 adds `TestCheckAlive` over the test process and drops both repeat rows.
- coverage MEDIUM, row-validity MEDIUM — no row reaches card 8's replacement of the kept `State.Tidy`: fixed as proposed; rows where tick 1 adds a Tidy, or clears one, with `.git` present and tick 2 removes the worktree.
- coverage MEDIUM — no test pins `PrimeTidyShuttle`'s wiring over the prime: fixed as proposed; card 8 adds tier1 `TestHubDobbyDeps_PrimeTidyShuttle`, set over the prime for a beam and nil for the prime.
- coverage MEDIUM, row-validity LOW — card 8's rows assert files while card 7's harness `Log` is a recording fake: fixed as proposed; card 8's test sets `Deps.Log` to `dobbyengine.AppendLog` and card 7's fake records path, line and `rotate`.
- coverage MEDIUM — card 7's `start fresh pid` line is never asserted: fixed as proposed; a fresh-start row asserts it.
- coverage MEDIUM, row-validity MEDIUM — the criterion that a no-location refusal's envelope lands in `dobby.out`, and `SpawnDetached`'s output file, are unreached: fixed differently; the proposal named no change, and an end-to-end spawn is refused under `go test`, so card 6 splits `spawnDetached` with a `start` seam and adds `TestSpawnDetached_SendsOutputToOutPath`, and card 10's loose-directory row names it as the second half.
- way-forward MEDIUM, coverage LOW — card 10's post-re-exec refusal notice names `lym beam start` without "fix the cause", and its content is not asserted: fixed as proposed; the notice says `fix the cause, then run lym beam start` and the rows assert the error and that text.
- way-forward MEDIUM — an undecodable `check.json` blocks every later check and status read: fixed as proposed; card 13's `ReadCheck` reads it as absent with one Warn, and `TestRunCheck` gains that row.
- way-forward MEDIUM, coverage LOW — card 7's state-read error has no line, contradicting step 5, and no row: fixed as proposed; step 1 writes the stop line with `rotate` false and posts nothing, reached by a row whose `state.json` is a directory.
- row-validity MEDIUM — card 12's observer subtest can meet a carried tier: fixed as proposed; it edits a file first.
- row-validity MEDIUM — no row reaches `Ended` for a carried or blocked tier: fixed as proposed; card 12's subtest gains a carried second call and a failing shared build.
- row-validity MEDIUM — card 7's second-read row can pass without the second read: fixed as proposed; the rewrite lands 500 ms into a one-second wait.
- row-validity MEDIUM — card 8's failed-stop row needs an error field on `tidyShuttleFake`: fixed as proposed; the fake and its `StopStrand` move to `Edit` with `stopErr`.
- row-validity MEDIUM — card 10's binary test needs a Windows skip and a future `Deadline`: fixed as proposed.
- guard-reach LOW — `CheckBinary` runs before the next iteration's worktree-gone test: fixed as proposed; the loop also tests it after each tick before `CheckBinary`, with a row asserting no `CheckBinary` call after a removal.
- guard-reach LOW — card 10's binary test bound of a minute depends on the watch: fixed differently; the proposal named no change, so the bound is three minutes, two sightings at the loop's one-minute idle ceiling.
- row-validity LOW — card 7's failing `CheckBinary` fake must fail once: fixed as proposed.
- row-validity LOW — card 7's 150 ms release margin is thin: fixed as proposed, through the wider-margin option; the release comes one second after `lock.ProbeGrace` under a three-second wait, since the seam option adds a production seam only a test calls.
- row-validity LOW — card 11's `--gate adopt` subtest repeats the `finalize` one: fixed differently; the subtest was not dropped, since the record's acceptance criterion names `lym gate test --gate adopt`; the existing scope-all subtest runs over both keys instead.
- row-validity LOW — card 4's Tidy-in-flight row has no future `Deadline`: fixed as proposed, with the settle step it then reaches.
- row-validity LOW, coverage LOW — card 3's failed machine-name read in `Landings` has no row: fixed as proposed; card 3 extracts `machineLandings` with tier0 `TestMachineLandings`.
- way-forward LOW — that failure's tick line names no way forward: fixed as proposed; `machineLandings`' error names `give the machine a hostname`.
- row-validity LOW — card 10's binary test leaves the dobby behind on failure: fixed as proposed; a `t.Cleanup` kills it.
- way-forward LOW — card 7's refused-re-exec notice names `lym beam start` while the lock is still held: fixed as proposed; it names `lym beam status`, then `lym beam start` once the dobby reads not alive.
- way-forward LOW — `ended without a verdict` never clears: fixed as proposed; card 15's view names the next `lym adopt` with an owner write.
- way-forward LOW — card 7's owner-exit line names no next step: fixed as proposed; it names `lym adopt root` or `lym adopt <slug>`, asserted by the owner-exit row.
- coverage LOW — `checkCmd`'s exit 0 and `already_running` envelope while one runs are untested: fixed as proposed; card 13 adds tier1 `TestCheckCmd_FromABeamExitsZeroWhileOneRuns`; the spawn's stdout on `check.out` stays unasserted, since `startSuite` refuses under test and no other process starts the check.
- coverage LOW — a gate that ends before any tier has no row: fixed as proposed; card 13 ends it `failed` with the exit code and adds the row.
- coverage LOW — `checkCmd` from a beam is never run: fixed as proposed, by the same tier1 test running from the beam.
- coverage LOW — card 10's wiring failure whose resolve fails has no row: fixed as proposed; a `wire` fake removes the worktree link first.
- forced-edits LOW — card 7 `Extends` `internal/dobbyengine/doc.go` while rewriting its loop bullets: fixed as proposed; moved under `Edit`.
- forced-edits LOW — card 3 leaves `Deps.Landings`' doc and the board-tidy bullet saying every Land Done counts: fixed as proposed; card 3 lists `internal/dobbyengine#Deps` and `internal/dobbyengine/doc.go`.
- forced-edits LOW — card 9 leaves `passKind.String` dead and `Deps.Log`'s doc stale: `passKind.String` fixed as proposed, deleted on card 9; `Deps.Log` fixed differently, its event list dropped on card 6, which already rewrites that field's doc, instead of adding `Deps` to card 9.
