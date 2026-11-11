```yaml
lens: fix-code-correctness
outcome: reported
```

Task: lens fix-code-correctness, run on the F1–F10 change set.

**Findings**

1. **BLOCKING (unverified by a run):** `internal/boardcli/cli.go`, the F6 line added to upsert's `Long` text (`  --record-file <path>  replace the entry's stored decision record ... never stdin.`). It likely pushes upsert's `Long` over the 2000-byte cap that `cmd/lym/clitree_test.go:167-281` enforces (`maxLongBytes = 2000`, checked by `longProblems`).
   - Commit `51214823` removed the earlier `--record-file` block from this `Long` text for exactly this reason, "keep upsert help under the cap".
   - My awk count of the upsert `Long` literal in source (a character count, not bytes, and without `slugLimitNote()`'s output) was 1991 at `51214823` and is 2118 now.
   - The gate run covered only `./internal/boardengine`, `./internal/boardcli` and `./internal/fabricengine`, so `cmd/lym` was never run.
   - Fix: drop the `Long` line and keep only the flag help text and `doc.go`, or run the `cmd/lym` test to confirm the size.

2. **LOW:** `internal/boardengine/recordupsert_test.go`, the F2 rows. Dropping the `filed.Project == slug` clause from `familyFiled` (`recordupsert.go:122`) passes every row.
   - `FiledPartWorktreeStands` is kind task, so the `Kind == KindProject` clause already refuses it.
   - `UnfiledProjectWorktreeStands` fails `held`.
   - `FamilyProjectHeldByARun` is refused by the earlier `RunHolds` case, before the exemption is evaluated, so it cannot detect an over-broad exemption.
   - The row the review asked for, a kind-project filed entry whose `Project` differs from its slug, is missing.

The remaining changes held up when traced:
- **F1:** the replay row's `stamp-2` check fails without the new `replayFields` row.
- **F3:** `store.go` reads `removedEntry` before the removal, and the extended merge test fails without the carried stamp.
- **F5:** the CLI `--board-path` row reaches the probe closure's nil-location error (the gate passed on it). No stale "once origin answers" text remains in `boardengine` or `boardcli`.
- **F7:** the new fabric row would error if the local branch did not decide alone.
- **F8:** the non-Sketch row would warn if the `IsSketch` gate were removed.
- **F10:** comment-only change.

**Further tool calls (4)**
1. `git show 51214823 --stat`, plus a grep for help caps. My context lacked why upsert's help was trimmed earlier.
2. `git show 51214823 -- internal/boardcli/cli.go`, filtered. It showed the removed `--record-file` block and that the removal was to stay under the cap.
3. `git show 51214823 -- internal/boardcli/cli.go | sed -n 50,80p`, plus a grep for `.Long` size tests. It showed the removed block and located `cmd/lym/clitree_test.go`.
4. A grep of `maxLongBytes` in `cmd/lym/clitree_test.go`, plus awk byte counts of the `Long` literal at `51214823` and now. It gave the cap value and the sizes.
