```yaml
lens: fix-claims
outcome: reported
```

Task: the fix-claims lens. I checked every bound, behaviour and fact that this round's changes added or rewrote against the code at HEAD.

**Findings:**

1. **LOW.** decision-record.md, D5 Bound, the new sentence "A dependency-drift miss lasts until testmap rebuilds the map by its present rules: the package's own files changed at HEAD, the age limit, or a failed build at another HEAD."
   - Summary: the text after the colon reads as the complete list of rebuild triggers, but a testmap call that names the package also rebuilds the map and ends the miss.
   - Evidence: `internal/gatecli/testmap.go:351` says "A named package needs one whatever its map holds". At `testmap.go:381`, `if !named && !r.stale(...)` skips the stale check for a named package. `stale` at `testmap.go:456` also rebuilds a map that is missing or unreadable.
   - Fix: write "such as" before the list, or add "or a testmap call naming the package".

2. **LOW.** decision-record.md, D3. The fix limits only the union's second member (unrecorded declared tests) to test files the host's build constraints admit.
   - Summary: the fourth member, "every `//lym:guard` test of the tier the package declares", still reads tests without build constraints. A guard test in a file constrained to another GOOS would therefore be named in `-run` and match nothing on the host. That is the same zero-test route F2 describes.
   - Evidence: `ScanTestFuncs` does not read build constraints (`internal/testcover/testfuncs.go:36`). Today no test file that holds `lym:guard` carries any `//go:build` line, so the case is theoretical. The same host constraint could apply to every member of the union.
   - The third member, out-of-process or unresolved tests, is safe. `runTest` records a test only when `result.Action == testcover.ActionPass` (`testmappackage.go:198`), so a test the host never compiles never enters the map and cannot be marked.

**Claims that hold:**
- D2's fresh-read wording and its Bound: the gap between the first dirty read and the second is the only window the HEAD comparison in `changedFiles` does not cover.
- D3 matches `holdsTierTest`: it uses `build.Default` with the tier tag and `MatchFile` (`tierrun.go:399-401` and `tierrun.go:470`).
- D6's claim about the call's args against the run's own args matches `tierrun.go:382-388`.
- "About twice a day": 15 of 26 commits over 7 days.
- The `go:embed` exemption errs safe: Go's own directory-pattern rule leaves out `.` and `_` files, so "every file below it" exempts fewer files than it could, which can only add misses.
- The predeclared-identifier miss and the Rationale's "no predeclared identifier" qualifier: a dot-imported name that clashes with a package-scope name fails to compile, so only predeclared names rebind silently.
- The new Open risks wording, that testmap rebuilds a map without the marker only by its present rules, holds.

**Tool calls:**
- One Bash call: it grepped for `lym:guard` test files that carry a `//go:build` line, to see whether finding 2 exists in today's code, and read `runTest`'s return paths to settle whether the third union member is safe. My context had neither.
