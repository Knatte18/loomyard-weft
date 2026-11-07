```yaml
lens: fix-proposal
outcome: reported
```

Findings: F5's `sleep 30 &amp; echo up; wait` holds up, and I found one LOW for F7.

**F7 — LOW.** The changed text is the `GuardMisplaced` case in `guardScanRefusal` (`internal/gatecli/cli.go:240`). The fix drops `%v` of the scan error. The refusal now says only "a marker … not directly above a top-level func Test line" and no longer names which marker it is. The way forward ("move the marker directly above its func Test line") is also stricter than the `//lym:watch` rule. The watch marker only has to sit in the comment block above the test, and `PATTERN-test-speed.md` says other `//lym:` lines may lie between. Its own scan error (`internal/impactset/watches.go:153`) says exactly that. A reader fixing a misplaced watch marker is told to move it "directly above", which over-constrains the fix but does not mislead. The integration test only checks the file:line and "way forward", so nothing fails.

Proposals I tried to break and could not:
- **F5:** `sleep 30 &amp; echo up; wait` is killed with the shell's whole process group on cancel (`internal/verifyrun/run.go:43`). The backgrounded sleep sits in that group, so the kill reaches it and the pipe closes. `echo up` runs after the fork, so the first write does mean a child exists. On Windows the `ping` banner is its own first write.
- **F4:** `defaultArmDeadline = armDeadline` is captured at package init, before any test replaces the var. The new row sits inside the serial parent, so it does not race the `SetDeadlineForTest` rows.
- **F3 and F6:** both are wording-only edits in `PATTERN-test-speed.md`, and I found nothing they contradict.

Tool calls made:
1. Searched `internal/verifyrun/*.go` (non-test) for how the shell and its kill are set up. My context held only the test file, and I needed it to judge whether the F5 change still kills the child.
