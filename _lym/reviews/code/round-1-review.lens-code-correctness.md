```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness lens (code-correctness). For each function or branch the diff changes, I ran one concrete case and record whether it is correct.

## Per-function list

1. `retryOnShortage` (`internal/gatecli/tierrun.go:215`) is correct.
   - A first run that passes returns at once.
   - A failure with no marker returns at once.
   - A timed-out run returns at once, through `failedOnShortage`.
   - A refused or build-blocked result returns at once.
   - A ctx already cancelled returns the first result.
   - A ctx cancelled during the delay returns the first result.
   - A marker followed by a pass returns the retry's result, with `shortage` empty.
   - A marker twice returns `retried.shortage` set. The scan runs before the announcement line is prepended, so the line's own quoted marker cannot match.
   - It makes at most one extra attempt.
   - `printLine` ignores a nil stderr.
2. `resourceShortage` and `failedOnShortage` (`tierrun.go:194`, `205`) are correct.
   - "Cannot fork" matches `cannot fork`, and "Resource temporarily unavailable" matches case-insensitively.
   - A log with no marker returns "".
   - An exit code of 0 returns "".
   - The scan lowercases a copy of the whole log, which only costs memory.
3. `failedRunLabel` and `runEnd.shortage` (`internal/gatecli/run.go:106`) are correct.
   - An empty shortage gives the old `(exit N)` text.
   - A set shortage gives the resource-shortage label with the way forward.
   - `report` copies `result.shortage` into `runEnd`.
   - The verdict stays the retry's exit code and is not marked transient.
4. `printFailedRuns` and `gateslot.FailedRunsPrefix` are correct. The constant text is identical to the old literal.
5. `withBuildServerOff` (`run.go:620`) is correct.
   - An inherited `MSBUILDDISABLENODEREUSE=0` is removed and `=1` is appended.
   - Both variables are applied once to the shared `env`, which runs, builds, vet and guard use.
   - Tier and build `env` are concatenated after it, so they can still override.
   - Matching is case-sensitive, which matters only for odd Windows casing. This is not a defect.
6. `Config.WithDefaultDir` (`internal/gateslot/tiers.go:344`) is correct.
   - It fills only empty tier dirs, so a set `dir` stays root-relative.
   - It fills the build dir only when a build command exists.
   - It copies the tiers map, so the caller's `Config` is not mutated.
   - An empty or nil `Tiers` becomes an empty non-nil map. This is harmless.
   - A root anchor fills `.`, which equals the old default, so green-record module keys are unchanged.
   - `loadGate` serves both the hub and non-hub paths, and `location` is non-nil whenever the refusal is empty. The fill at `run.go:77` is therefore safe.
   - Other `LoadConfig` callers (hubgeom, loomcli, webstergeom, claudeengine) read no `Tier.Dir` or `Build.Dir`.
   - `resolveRunDir(root, "")` equals the root, so a stray empty dir would not break.
7. `buildServerExports` and `composePaneLaunchLine` (`internal/reedengine/panebin.go`) are correct.
   - Quoting goes through `sh.ExportEnv` and `sh.Chain`, so it follows each shell flavour.
   - The exports survive a failed executable lookup.
   - An empty command yields the exports alone.
8. `failuresSection` (`internal/loomshed/verifygate.go:426`) has the two defects below.
   - Opaque failures are skipped, so a log that parses to opaque falls back to the log tail.
   - The byte bound is applied between whole entries, so it cannot cut mid-rune.
9. `outputAboveHeader` (`verifygate.go:462`) has defect F1.
10. `failedRunsLine` is correct. It takes the last matching line and returns "" for an empty log.
11. `verifyFailureFindings` is correct.
   - The `Failures:` branch replaces the tail.
   - The tail branch is kept when the parser names nothing or only an opaque failure.

## Findings

**F1: MEDIUM, design.** `internal/loomshed/verifygate.go:462-477` (called from `:435`).
- `outputAboveHeader` searches the whole log for the first `--- FAIL: <test> ` line and ignores the failure's package.
- The test name comes from `TrimPrefix(ID, Package+".")`.
- A common name such as `TestParse` failing in a later package then gets the output above an earlier package's header, or "".
- That earlier header can be a failing test whose output the parser found nested, or a passed or other test of the same name.
- The re-prompted agent is then handed another package's output as this failure's evidence.
- Fix: scope the search to the failing package's block, for example by stopping at the preceding `ok`, `FAIL` or `?` package line, or by tracking the block start the way `parseVerifyFailures` does with `blockStart`.
- Evidence: the parser's `Failure.Package`, the `packageBlock` logic in `websterengine/verifyparse.go:80-111`, and the unscoped loop at `verifygate.go:464-467`.

**F2: LOW, design.** `verifygate.go:437-442`.
- "The first failure is quoted whole whatever its size" leaves it unbounded in bytes.
- The parser's tail is capped at 40 lines, and `outputAboveHeader` is capped at 40 lines, but a single line can be arbitrarily long, for example a test dumping a huge blob.
- The `verifyFailuresBytes` bound can then be exceeded without limit by the first entry.
- The focus asks for a byte bound that does not cut mid-rune or drop all failures.
- Fix: truncate an oversized entry on a rune boundary (`utf8.RuneStart`), with a marker, before the `b.Len() > 0` check.

**F3: LOW, design.** `verifygate.go:469-475`.
- When a test's output sits above its header, the upward scan stops only at a line starting with `=== ` or `--- `.
- If the header follows a package-closing `FAIL\t<pkg>`, `ok  \t<pkg>` or bare `FAIL` line, it pulls those lines, and anything else back to the nearest marker, into the test's output.
- This is noise rather than a wrong verdict.
- Fix: also stop at `FAIL`, `ok  \t`, `?   \t` and `FAIL\t` package lines.

## Tool calls
1. Read `internal/websterengine/verifyparse.go`, because I needed the parser's `Tail` and `Package` semantics.
2. Grepped `.Dir` readers in gatecli, gateslot, verifytree and loomshed, because I needed every consumer of the now-empty `Tier.Dir` and `Build.Dir`.
3. Grepped `LoadConfig` callers and `Build.Dir` and `tier.Dir` readers, then read `build.go` and `resolveRunDir`, because I needed to rule out unfilled-dir readers outside gatecli and see the build's dir handling.
4. Read `buildRoot` and `loadGate`, because I needed to confirm `location` is non-nil at the fill point and what root the dir resolves against.
