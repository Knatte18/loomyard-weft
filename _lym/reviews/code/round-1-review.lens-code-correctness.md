```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each changed function and branch with a concrete case.

**Functions and branches, the case run for each, and the result:**
- `RunRecord.SlotWaitSeconds` with a zero `WaitStart` returns 0. Correct.
- `RunRecord.SlotWaitSeconds` with `WaitStart` set 300 s before `Start` returns 300. Correct.
- `TierSummaryLine`, plain slotted tier: `runWithSlots` sets `waitStart` before `acquireSlot` (run.go:678–700), and `runTierCommands` sets `Start` when the command begins (tierrun.go:130). The wait is the slot wait plus the `untrackedBeforeRun` listing (strayfiles.go:37–41, a git listing of untracked files). Correct up to the cost of that listing.
- `TierSummaryLine`, first tier that opts into the shared build: the body runs `runBuild` before `runTierChecked` (run.go:640–644). `runBuild` starts its own clock at build.go:41, and the tier's `Start` is only taken after the build ends. `report` sets `WaitStart = waitStart` (run.go:729), so the printed `wait` holds the slot wait plus the whole build. **Wrong.**
- `TierSummaryLine`, `--copies` run with a prebuilt lym: `runTierCopies` prebuilds lym (copies.go:64–79) before any copy's `runTierCommands` takes `started`, and `mergeCopies` keeps the earliest copy's `Start` (copies.go:135–137). The prebuild time is counted in `wait` and not in `wall`. On a run without copies, the same prebuild runs inside `runInvocations`, after `started` (tierrun.go:559), so it lands in `wall`. **Wrong, and inconsistent between the two paths.**
- `TierSummaryLine`, `--copies` run whose prebuild failed under a cancelled context: the record is `Start: now` (copies.go:72) and `report` sets `WaitStart = waitStart`. The wait is the slot wait plus the prebuild attempt, the same defect as the case above.
- `TierSummaryLine`, rerun: `retryTier` writes the first record with `WaitStart = waitStart` (run.go:649) but prints no summary line for it. The rerun's record gets `WaitStart = Start` (run.go:730–733), so the only line printed shows `wait 0.0s` although the call waited for slots. This matches the change description's stated intent. Against the goal, though, the call's output never shows the slot wait it spent. **Wrong against the goal.**
- `TierSummaryLine`, carried tier: `reportCarried` passes `now` as both `waitStart` and `Start` (run.go:662–664), giving `wait 0.0s`. Correct.
- `TierSummaryLine`, empty scope: no slot is taken, and the wait is the gap between `waitStart` and `started`, about 0. Correct.
- `TierSummaryLine`, inherited or unslotted call: no slot is taken, and the wait is about 0. Correct.
- `reportBlocked`: prints no tier summary line, and its record has `WaitStart = Start = now`, so its wait is 0. Correct.
- `runReport`: `SlotWaitSeconds` gives the same figure as the inline code it replaced, on both the abandoned and the normal branch. Correct, a refactor with no behaviour change.
- `tierSummaryPattern` and `tierDiscoveredPattern` on a line from the new writer: `wait \S+s` matches `300.0s`, and a negative exit still matches through `-?\d+`. Correct for lines this binary writes.
- The same two patterns on a line in the old format (`skipped 0 wall 1.5s exit 1`): no match, so `FailedTier` returns `ok=false` and `TierDiscovered` returns `found=false`. These are silent misreads. Whether such a line can reach a reader is the affected-set lens's to judge.

**Findings:**

1. **MEDIUM, design.** Location: `internal/gatecli/run.go:729` (`report`), with the shared-build body at run.go:640–644.
   - Summary: on the first tier that opts into the shared build, the summary line's `wait` includes the whole build duration, and `wall` excludes it.
   - The help says `wait` is the tier's slot wait (cli.go, the line the diff adds), and so does the package doc (doc.go:372–374).
   - A caller who adds `wait` and the build line's own `wall` counts the build twice. A caller who reads `wait` as contention sees the build as contention.
   - Evidence: build.go:41 sets the build's own `started`, the tier's `started` is at tierrun.go:130, and `waitStart` is taken at run.go:678.
   - The change description's sentence "the tier's record already holds that wait" (`_lym/landing/summary.md`, Design decisions) misstates this.
2. **MEDIUM, design.** Location: `internal/gatecli/copies.go:64-79` and copies.go:135–137.
   - Summary: under `--copies`, the lym prebuild runs before the copies' `Start`, so the line counts it in `wait` rather than in `wall`. A run without copies counts the same prebuild in `wall` (tierrun.go:559).
3. **MEDIUM, design.** Location: `internal/gatecli/run.go:730-733`, together with `retryTier` at tierrun.go:522.
   - Summary: when a tier is rerun, the only summary line printed carries `wait 0.0s`. The slot wait the call spent is recorded only on the first run's record, which prints no line.
   - So the call's output does not separate the slot wait from the run time, which the goal asks for, in exactly the case of a long call.

Tool calls:
1. Read `runBuild`'s head, `untrackedBeforeRun` and `runTierChecked`, to confirm where the build's clock and the tier's `Start` are taken relative to `waitStart`. My context held only the record construction in `runBuild` and the call sites, not those functions' bodies.
