```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens over `decision-record.md`. I checked each pair of decisions that share state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1×D2 (package set vs whole replacement): broken, F1.
- D1×D4 (both tiers in one pass vs per-tier parallel and `memory_mb` clamp): broken, F2.
- D1×D7 (detached keying vs no run record or carry): held.
- D1×D9 (missing-map refusal names `lym gate test --map <pkg>`): held.
- D2×D6 (one-ahead build vs pre-build dirty check over linked packages): broken, F3.
- D2×D6 (moved HEAD ends the call vs cancel and wait for the pending build): held.
- D2×D7 (whole replacement vs a failed build keeping old entries): held; the two decisions cover different cases.
- D3×D4 (prebuilt `lym` and tier env vs the per-test kernel cap that covers child processes): held.
- D3×D5 (outcome entries vs the alone-rerun): held.
- D4×D5 (guard or cap kill beside others vs rerun alone): held.
- D4×D7 (slot hold as the larger tier ask vs per-tier side-by-side count): held.
- D5×D7 (exit judged after the alone-rerun): held.
- D6×D7 ("not written" for a dirty package vs "still writes every map it measured"): broken, F4.
- D6×D8 (HEAD and dirty checks vs last write wins between two runs): held.
- D7×D8 (no run record vs where maps are written): held.
- AC D4 "never passes the tier's `parallel`" vs D1's two-tier pass: broken, folded into F2.
- Constraints `PATTERN-gate-slot`, `PATTERN-child-processes`, `PATTERN-test-speed` and `PATTERN-daemon-wakeups` vs D2, D4 and D7: held.

**Findings**

F1. MEDIUM, design, D1 package set × D2.
- **Summary:** D1 scopes a call to packages that hold a tier 0 or tier 1 test. A package whose last such test was deleted drops out of the set, so its old map is never replaced. Its stale entries stay, which defeats D2's promise and the D2 criterion "an entry of a deleted test is gone".
- **Evidence:**
  - `decision-record.md:39` is D1's union of test targets.
  - `decision-record.md:53` and `:176` are D2's whole replacement and its criterion.
  - `internal/gatecli/testmappackage.go:131-132`: today a package with no mapped test still gets a map of its file hashes with no tests.

F2. MEDIUM, design, D4 × D1.
- **Summary:** D1 measures tier 0 and tier 1 tests of a package in one pass. D4 bounds side-by-side processes per tier ("a tier's tests run side by side up to that tier's `parallel`", and per-tier `memory_mb` over the per-test cap). The record never says whether the two tiers' processes run at the same time. If they do, the package runs up to tier 0's count plus tier 1's count at once. Neither tier's `parallel` nor its `memory_mb` then bounds the total, so the D4 criterion "never passes the tier's `parallel`" is unchecked for the combined run.
- **Evidence:**
  - `decision-record.md:40`, `:75-79` and `:180`.
  - `internal/gatecli/testmappackage.go:604-608`: today one count comes from tier 0's parallel alone and covers every mapped test.

F3. MEDIUM, design, D6 × D2.
- **Summary:** D6 checks "before a package's coverage build" that no file is dirty under the directory of any in-module package it links. The linked set only becomes known inside the build itself: the build goroutine runs `go list` before compiling. D2 also starts the next package's build one ahead, during the current package's tests, so no point before the build knows which directories to check. Today's code makes the check after the build is taken up. The record leaves Plan to discover that the pre-build check cannot cover linked packages, and leaves the actual timing of that check open.
- **Evidence:**
  - `decision-record.md:56` and `:98-99`.
  - `internal/gatecli/testmappackage.go:224-232`: `startBuild` lists the linked packages inside the build goroutine.
  - `internal/gatecli/testmappackage.go:159-178`: the dirty check runs only after `takeBuild`.

F4. LOW, consistency, D7 × D6.
- **Summary:** D7 says the run "still writes every map it measured, failed entries included". D6 says a package found dirty, or under a moved HEAD, at the pre-write check gets no map, although its tests were measured. Read literally, D7's sentence contradicts D6 for that package. The fix is to scope D7's sentence to packages that passed D6's checks.
- **Evidence:** `decision-record.md:100` and `:113-114`.

No tool calls made.
