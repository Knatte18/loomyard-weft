```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, I list everything that calls, reads or asserts it, and judge whether the diff leaves any of it wrong.

**Per change, the set found:**

- **`gateslot.TierSummaryLine` output** (the new `wait <s>s` field before `wall`):
  - Readers in the same binary: `FailedTier` and `TierDiscovered` (`internal/gateslot/runlog.go:46,49`). The diff updates both. Covered.
  - `judgeDotnetRedFirst` (`internal/gatecli/redfirstdotnet.go:312`) reads output captured from the in-process `runTiers` (`redfirstdotnet.go:117-131`), so writer and reader are the same binary. Covered.
  - `redfirstdotnet_test.go:28` builds its line through `TierSummaryLine`. Covered.
  - `testFailedTier` and `TestTierDiscovered` (`runlog_test.go:165,171,210`) also build lines through `TierSummaryLine`. Covered.
  - Reader of a log another process wrote: `verifyGate`'s two `FailedTier` calls (`internal/landingshed/verifygate.go:99,394`). They read `result.Log`, the output of a `lym gate test --wait` command that `Verify` runs through the shell (`internal/verifytree/verify.go:334`). That `lym` is the one resolved on PATH, not necessarily the binary doing the reading. **Not covered** (F2).
  - Literal fixture `internal/gateslot/startedlines_test.go:36`. **Not covered** (F1).
  - `docs/research/speed/test-targets-bench-2026-11-07.md:28` quotes an older line. It is a measurement report, so it is exempt. No finding.
  - Stencils and specs: none spell the line or its fields. No finding.
- **`RunRecord.SlotWaitSeconds`** (new):
  - Callers: `runReport` (`report.go:147,157`) and `TierSummaryLine`.
  - The report help's "summed slot wait" and `abandoned_wait_seconds` (`report.go:35-37`) keep their meaning, because the formula is the same as the inline one it replaces. Covered.
  - `callSpan` (`detachfront.go:560-566`) reads `WaitStart` with its own zero fallback. The diff leaves it alone and it is still consistent. Covered.
- **`report.go` inline wait replaced by `SlotWaitSeconds`:** same result, and no reader changes. Covered.
- **`lym gate test` Long help:**
  - The new wait/wall sentence is covered.
  - Dropped `--copies` paragraph: every fact it held (with `--gate` and explicit targets, N copies of the key's one tier side by side) is still in the flag help (`cli.go:132`) and the package doc (`doc.go:44`). Covered.
- **`gatecli` package doc** (`doc.go:372-375`): agrees with the help. Covered.
- **Repository rules:**
  - No PATTERN entry is declared changed, and no `pattern/` background lists the summary line among its sole-parser formats (`pattern/PATTERN-sole-parsers.md`). No finding.
  - The test follows the `PATTERN-test-speed` shape (`tiergate.Require(t, 0)` then `t.Parallel`).
- **Commit messages:** the range carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed. No finding.

**Findings:**

- **F1. LOW, consistency.** Location: `internal/gateslot/startedlines_test.go:36`. The `OtherLinesAreSkipped` fixture still spells a tier summary line in the old form, `... skipped 0 wall 0.1s exit 0 log x`, with no `wait` field. So it no longer represents a line the gate prints. Evidence: `startedlines_test.go:36`, compared with the new format at `runlog.go:573`.
- **F2. MEDIUM, design.** Location: `internal/gateslot/runlog.go:46,49`.
  - Both patterns now require ` wait \S+s` between `skipped` and `wall`.
  - `verifyGate` (`verifygate.go:99,394`) runs them over verify logs that a shell-spawned `lym gate test --wait` wrote (`verify.go:334`), and that `lym` comes from PATH.
  - If the binary that wrote the log differs from the one reading it, in either direction, no line matches. `FailedTier` then returns not found, so the landing failure record loses `Build` and the failing-tier reason falls through to the generic path.
  - This contradicts the change description's claim that "the writer and readers of the line ship in the same binary". It also bears on the `two-way` Merge danger call, since a log format another binary may read changes.
  - Making the `wait` field optional in both patterns would accept lines in either form.
  - Evidence: `runlog.go:46,49`, `verifygate.go:99,394`, `verify.go:334`, `_lym/landing/summary.md` (Design decisions and Merge danger).

Tool calls:
1. Searched the repository for literal summary-line spellings in docs, fixtures and specs. The manifest held only `startedlines_test.go`.
2. Searched stencils and specs for summary-line descriptions, and found the `report.go` wait mentions. Doc and stencil readers were not in the manifest.
3. Read the other "summary line" mentions and the `lym gate report` help, to check every doc that describes the line or the wait figure.
4. Searched for the `WaitStart` field doc and the `pattern/` background references to the line, to check the rule documents.
5. Read the `RunRecord` field block and the sole-parsers background list, to confirm the line is not a declared sole-parser format.
