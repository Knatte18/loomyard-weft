```yaml
lens: walkthrough
outcome: reported
```

The walkthrough review is done. It found one MEDIUM finding (D6) and one NIT (D7); the other seven decisions meet their purpose.

**Walkthrough list (decision, the case I ran, purpose met or not):**
- **D1**, `lym gate test --map internal/x` on this hub (tier0 and tier1 are both `go-json` with no `memory_mb`), first plain, then detached with a plain call beside it: met. The scope gives the package set, and keying detached runs on `--map` keeps the map run and the plain run apart. The help output lists `--tags` as already refused, and the refusal list covers every other flag that conflicts with `--map`.
- **D2**, a package whose map already exists and one of whose tests was deleted, with a successful build: met. The whole map is replaced, so the deleted test's entry is gone, and the one-ahead build is cancelled and waited for when the call ends early.
- **D3**, a tier1 package (`prebuilt_lym: true`) holding one plain integration test and one test that calls `lymbin.Build`: met.
  - `ScanSpawns` marks the `lymbin` test as spawn-marked (`spawnscan.go:355`), and the run skips it (`testmappackage.go:139`).
  - The plain test runs, and `lymbin.prebuilt` (`lymbin.go:49`) is never reached.
  - So `runTest`'s `lymBin != ""` branch (`testmappackage.go:671`) never matters, and every D3 criterion describes a test that can run.
- **D4**, a package holding tier0 and tier1 tests, both tiers at `parallel: 4`, tier1 set to `memory_mb: 4096`, a 2048 MB per-test cap, cgroup not limited:
  - tier0 runs 4 tests side by side (`clampParallelToMemory`, `redundantmutants.go:544`).
  - tier1 runs min(4, 4096/2048) = 2, so the 4096 MB cap bounds the processes together.
  - The builds take tier0's `-p`. Setting `memory_mb: 1024` is refused before the quotient could be 0.
  - Purpose met.
- **D5**, a tier1 test killed by the guard while 3 siblings run, then passing alone: met. It gets a pass entry and is listed in the call's report. With the count clamped to 1, nothing runs beside it, so no rerun happens, which is consistent.
- **D6**, a worktree with an uncommitted edit to package A's file `a.go` that adds an import of a package not yet created: **not met**. Finding 1 has the steps.
- **D7**, a tier with `slots: all` and another at the default of 1: the purpose holds as written, but a step in it misleads. Finding 2 has the details.
- **D8**, a map run in the prime and another in a side worktree: met. Each writes its own map location as the verb does today.
- **D9**, `lym gate redundant A` run on a worktree with dirty edits to A: met. The refusal says to commit or revert and then run `lym gate test --map A`, and after that D6 lets the write through.

**Findings**

1. MEDIUM, class `design`, D6 ("A build whose listing of the linked packages fails records its failure with no check").
   - **Summary:** a failed listing caused by an uncommitted edit is recorded as a failed build at HEAD, with no HEAD or dirty check. That breaks D6's own rule: "a map is written only when its files are clean".
   - **Case:**
     1. The dirty `a.go` imports a package that does not exist yet.
     2. `testcover.Toolchain.LinkedPackages` (`internal/testcover/runner.go:95`) runs `go list` and fails.
     3. D6 skips both checks, so the run writes the map.
     4. `writeFailed` (`internal/gatecli/testmappackage.go:857-873`) stamps the failure with `Commit: r.head`. If the old map holds no test entries, it replaces the map with one recording only that build failure.
     5. HEAD itself builds fine, yet `lym gate redundant A` is now refused for "recording only a failed build" (D9).
   - **Fix direction:** neither check needs the linked set for this case. HEAD can still be checked, and so can the files package A owns under its own directory. D6 should run both before a listing failure is written.

2. NIT, class `consistency`, D7 ("as many gate slots as the larger of tier 0's and tier 1's slot asks") and the matching line under `## Constraints` (the `PATTERN-gate-slot` constraint).
   - **Summary:** "larger" does not say that `all` wins. `slots: all` is stored as `Tier.Slots == 0` (`internal/gateslot/tiers.go:42-43`, `:263`), so taking the larger number of tier0's and tier1's slot values gives 1 when one tier asks for `all` and the other for 1. That contradicts D7's own Bound, which says such a run holds every slot.
   - **Fix direction:** state that `all` beats any count.

**Tool calls**
- `internal/gatecli/detachfront.go` around `phaseSlots`, plus grep in the testmap files and `upkeep.go` for stale-map pruning: my context did not show whether the detached front takes slots before the run does (it only reports the phase), or whether today's verb prunes the maps of deleted packages (it does not).
- `internal/gatecli/testmappackage.go:840-905` and grep of `testcover/runner.go` for `LinkedPackages`: my context did not show what a failed build writes or which commit stamps it, which finding 1 needed.
