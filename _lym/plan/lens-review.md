# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
One line per finding, merged across lenses where two named the same defect.

- row-validity, BLOCKING, card 6 failing-`LoadConfig` row aborts through `passHarness.tick`: fixed as proposed; the row calls `d.gateHolds` directly on a probe-due daemon, and the `Holders` failure row stays separate.
- guard-reach, MEDIUM, card 3 episode stamp misses a folded repeat (`internal/shedengine/run.go` `appendOrFold`): fixed as proposed through the parent, who recorded the Added after Discussion entry "Episode stamp counts folded repeats"; card 2 adds `LastRepeats` to `RunView` and `ChildReading` with rows in `TestDobbyRuns_…` and `TestReadNoticeChild`, card 3's stamp adds it, its episode row now folds a repeat, and the overview's Shared Decisions names the entry.
- coverage, MEDIUM, card 3 failed level read after a success has no row: fixed as proposed with a `rows`-then-fail row.
- coverage, MEDIUM, card 1 refusal rows do not name the three values: fixed as proposed; both rows want `[stops rows off]`.
- coverage, MEDIUM, card 1 `CheckLocalValues` has no reaching test: fixed as proposed with a new `TestCheckLocalValues` table.
- coverage + row-validity, MEDIUM, card 3 child-read branches untested and rows not moving `ChildStatusModified`: fixed as proposed; rows for an unchanged mod time and a failing `ReadChild`, and the test note that each changed child reading moves `ChildStatusModified`.
- coverage, MEDIUM, card 3 `pairRelative` and one-line cut untested: fixed as proposed with a reason row holding a path under `PairDir` and a line break.
- coverage, MEDIUM, card 3 entry upkeep untested: fixed as proposed with drop and keep rows.
- coverage, MEDIUM, card 4 no assertion of an empty topic and run tag: fixed as proposed; the test asserts both empty on the watch line's mail.
- coverage + row-validity, MEDIUM, card 7 `dobbyGateHolders` mapping and load failure untested: fixed as proposed with a new tier-1 `TestDobbyGateHolders` over a hubforge hub.
- coverage, MEDIUM, card 6 `Tree`/`MarkedPIDs` failure, `TaskWorktree` failure and failed-post retry untested: fixed as proposed with one row each.
- row-validity, MEDIUM, card 3 retry row cannot tell a probe-interval wait from every tick: fixed as proposed; the row ticks through `tickAfter` below the probe interval.
- row-validity, MEDIUM, card 3 cleared run's `Watches` drop passes vacuously: fixed as proposed; a seeding field puts an entry owing nothing on every cleared row.
- row-validity, MEDIUM, card 7 no-supersede assertion confounded by the two existing run notices under one topic: disputed, premise false; `supersedeLocked` in `internal/creelengine/verbs.go` takes `from` and supersedes only messages from the same sender, and the test's two rows post from the prime's dobby and a beam's dobby, two senders.
- coverage, LOW, card 6 non-probe tick unasserted: fixed as proposed with a row that wants no `Holders` call.
- coverage, LOW, card 3 anchor move holding the watch unasserted: fixed as proposed with a `MoveRecords` row.
- coverage, LOW, card 3 child done and cancelling unasserted: fixed as proposed with one row covering both.
- coverage, LOW, card 3 a held clear that then clears unasserted: fixed as proposed with a `TestClearDoneRuns` row that posts and clears.
- way-forward, LOW, card 3 level Warn lacks the lookup error: fixed as proposed; the Warn carries it.
- way-forward, LOW, card 6 step gated by the scheduling block: fixed as proposed by stating the bound in the Intent; the step stays there since it matches against `d.listedRuns`, which only the scheduling pass step lists.
- guard-reach, LOW, card 6 holder tree unbounded in the line: fixed as proposed by bounding it; `gateHoldProcessMax` caps each list at 20, most CPU first, with `; and <n> more`.
- forced-edits, LOW, card 4 orch-role prefix rule at `contracts/orchrole/orch-role.md:81`: fixed as proposed; the rule names `[run watch]`.
- forced-edits, LOW, hub stencil copy of the orch role out of reach: fixed as proposed; the overview's Shared Decisions records it.
- forced-edits, LOW, cards 4 and 7 stale test comments: fixed as proposed; both cards rewrite the test's doc and file comment.
- forced-edits, LOW, card 1 stale `tasksetting_test.go` file comment: fixed as proposed; card 1 names it.
- forced-edits, LOW, card 5 `gateslot` doc does not name the marker: fixed as proposed; `internal/gateslot/doc.go` joins card 5's targets.
- forced-edits, LOW, card 1 board run_settings reader omits `run_watch`: disputed, premise false; `internal/boardengine/store.go`'s `BeamSettingsByEntry` is read only by `internal/dobbyengine/reinterpret.go:181`, the reinterpretation of the retired `pull_request` and `notify_on_land` keys, and `run_watch` has no retired form; D2's chain reads the worktree's local layer, which batten fills from `run_settings`.
- row-validity + guard-reach, LOW, card 5 new tier-1 test repeats `TestGateTest_DetachedRun`'s setup and its stand-in waits on no release file: fixed as proposed; the new test is cut and `TestGateTest_DetachedRun`'s first subtest reads the holder's marker before `release()`, with `newRelease` and `releaseEnv` under `Uses`.
- row-validity, LOW, card 3 not-posted row's tick count unstated: fixed as proposed; four not-posted ticks.
- row-validity, LOW, card 6 fake `Tree` shape unstated: fixed as proposed; it answers by PID.
- row-validity, LOW, card 2 shared constants gain history and reason: fixed as proposed; card 2 says so.
