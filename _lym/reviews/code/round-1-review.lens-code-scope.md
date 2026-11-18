```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It traces each change in the diff `0b42877c..HEAD` to the board entry's goal points.

**Decisions.** These are the goal points under `## Goal` in `_lym/discussion/decision-record.md:17-27`. The record has no `Added after Discussion` entries.

- **G1:** find why the cap sometimes does not apply.
- **G2:** a capped command never runs uncapped silently; if the cap cannot be set, the call fails or says so loudly.
- **G3:** every probe and test hog is bounded in size and time.
- **G4 (Done when):** the memory cap tests pass on many repeated runs, with the count stated and other gate work running beside them.
- **G5 (Done when):** the change description names the root cause and how the fix closes it.

**Changes, and the decision each traces to.**

- **C1, holder and read-once-ended** (`memorycap_linux.go`): the holder part of `scopeStartScript`, the `holderEnd`/`release` pipe, `ExtraFiles` gaining `holderEnd`, the watch goroutine and `watchForOOMKill` removed, and `scopeHadOOMKill` read after `wait`. This traces to G1. It fixes the root cause found for the seen failure: the kill report raced the scope's removal (`summary.md:12-18`). Dropping `watchForOOMKill`, `poll` and `errors.Is(EINTR)` is forced by C1.
- **C2, the scope checks its own `memory.max` before running** (`memorycap_linux.go`): `cgroupMemoryMaxFile`, the `max`/`$1` check with exit 125, `wantMax` passed as `$1` with `shift`, and the `!ranUnderCap` error naming the expected `memory.max`. This traces to G2: a user manager that accepts `MemoryMax` and ignores it now fails with `ErrMemoryCap` instead of running uncapped.
- **C3, doc and comment updates** (`doc.go:57-60,75-76`, the doc comments of `runMemoryCapped`, `scopeStartScript` and `scopeCgroup`): forced by C1 and C2, which made the old text stale.
- **C4, the new row in the tree-kill test** (`AStartThatReturnsOnceTheCommandHasEnded`, `memorycap_linux_test.go:125-133`): forced by C1, as its regression row.
- **C5, the scope-start test renamed with the new row `AScopeThatIgnoresTheCap` and its stand-in starter** (`memorycap_linux_test.go:190-225`), plus the file header (`memorycap_linux_test.go:4-5`): forced by C2.
- **C6, the `export_linux_test.go:1` header**: forced by C5, since the stand-in starter now covers a scope that ignores the cap.

**Decisions, and the changes that build each.**

- **G1:** C1. The root-cause finding itself is in `summary.md:6-22`, which is not part of the diff.
- **G2:** C2. The `RLIMIT_DATA` fallback (`memorycap_linux.go:148-149,265-281`) is untouched and still caps each process. `MemoryCapCoversTree` reports it, so it is not a silent uncapped path. No change is owed there.
- **G3:** no code change is needed. The hogs are still fixed at `hogOverCap`/`hogWithinCap` and exit by themselves (`memorycap_test.go:21-24`, `testmain_test.go` `hoardMemory`). The new rows add only `echo` commands and the existing bounded hog.
- **G4:** this is a claim in `summary.md:52-54`, not a diff subject. The 50 runs are consistent with 16 results per run.
- **G5:** this is in `summary.md`, not the diff. It names the root cause (`summary.md:6-18`) and how the fix closes it (`summary.md:26-39`).

**Findings:** none. Every change in the diff traces to G1 or G2, or is forced by a change that does. No change is unrequested, no goal point is left unbuilt, and the range has no `Pattern-Departure:` trailer.

**Tool calls:** none. The reading manifest held everything this lens needed.
