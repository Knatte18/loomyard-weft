```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its surroundings, through the fix-consistency lens.

**Findings**

1. **MEDIUM**, `internal/gatecli/cli.go:242`, the `WatchMalformed` case of `guardScanRefusal`.
   - Summary: the F7 double way-forward was fixed in the `GuardMisplaced` case only. The twin case still has it.
   - Evidence: the case formats `%v` of `err` and then appends its own `; way forward: give the marker one or more path.Match patterns…`.
   - The scan error text from `internal/impactset/watches.go:138` already ends in `; way forward: write the marker as \`//lym:watch &lt;pattern&gt;...\` with \`path.Match\` patterns`.
   - A malformed-watch refusal therefore still states the way forward twice, which is the premise of F7.

2. **LOW**, `pattern/PATTERN-test-speed.md:118`.
   - Summary: the F6 sentence was added as a new bullet, but the bullet above it still says the guard step "runs every marked test by name under its file's tier, whatever the impacted set".
   - Evidence: that wording stays unqualified, so the two adjacent bullets contradict each other.
   - A reader of the first bullet alone still gets the stale rule. Qualify it, for example "every marked test not dropped by `//lym:watch`", or fold the new sentence into that bullet.

3. **LOW**, `internal/gitexec/export_test.go:1-3`.
   - Summary: the file header lists what the file exposes and does not mention the production deadline hook added by F4.
   - Evidence: the header covers the remote deadline, the stall check, the production stall timer, the version probe and start failures. It is silent on `ArmDefaultDeadlineForTest`.

I found no other twin or cross-reference left unchanged. The `isWatchLine`, `scanGuards` and `moduleRelative` docs agree with the new tests. The `TestRun_RemoteBounds` doc comment already names the deadline seam. The F3 exception matches how the `remoteDeadline`, `stallBound`, `startStallTimer` and `armDeadline` seams are declared.

**Tool calls:** none. My context already held every file I checked.
