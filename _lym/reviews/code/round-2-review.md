---
verdict: BLOCKING
findings:
  - id: F8
    severity: BLOCKING
    class: consistency
    location: internal/proc/memorycap_linux_test.go:178
    summary: Two new tests the redundancy report lists as contained pin what their covering tests never check, and neither carries the testtiming keep marker PATTERN-test-speed requires of a kept candidate
    fix: In internal/proc/memorycap_linux_test.go, add a line //testtiming:keep pins that the scope is removed once a capped run with no kill returns, which no covering test checks directly above func TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns. Add a line //testtiming:keep pins the ErrMemoryCap refusal of a scope held to the cap whose PATH has no setsid, a shell branch coverage cannot see directly above func TestRunMemoryCapped_AScopeWithoutSetsidIsAnErrorAndNeverRunsTheCommand. Leave both test bodies unchanged.
    origin: assist:assist
  - id: F9
    severity: BLOCKING
    class: consistency
    location: internal/proc/memorycap_linux_test.go:185
    summary: TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns repeats the literals /proc/self/cgroup and /sys/fs/cgroup that proc declares once as selfCgroupFile and cgroupMount, against PATTERN-code-conventions Named constants
    fix: In internal/proc/export_linux_test.go add exported test aliases, const SelfCgroupFile = selfCgroupFile and const CgroupMount = cgroupMount, each with a one-line doc comment. In TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns in internal/proc/memorycap_linux_test.go, replace the cat argument /proc/self/cgroup with proc.SelfCgroupFile and the first argument of filepath.Join, /sys/fs/cgroup, with proc.CgroupMount.
    origin: handler
prior:
  - id: F1
    status: resolved
    answer: cgroupMemoryMaxFile is declared once in memorylimit.go, which has no build constraint, and readMemoryLevel, scopeStartScript and the ranUnderCap error all use it; no memory.max literal is left outside comments.
  - id: F2
    status: resolved
    answer: The tier command is head -c 268435456 /dev/zero piped into tail, bounded to 256 MB above the 64 MB cap and by TimeoutMin 1; the sleep and its two comment lines are gone, and the remaining comments describe the current command.
  - id: F3
    status: resolved
    answer: Records 20261118T142443Z-tier1-444250 (950 passed, 0 failed, 15:24:25 to 15:24:43 local) and 20261118T142527Z-tier1-444438 (15:24:27 to 15:25:27 local) bear out the count and the 16 s overlap. That run predates the setsid check of 6bdced24; a run of the same command at HEAD, 20261118T150648Z-tier1-1030999, passed 1000 of 1000 results.
  - id: F4
    status: resolved
    answer: TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns skips without a scope and fails when the scope directory outlives a one-minute deadline after a run with no kill.
  - id: F5
    status: resolved
    answer: The holder now runs under setsid inside the /dev/null subshell, and the scopeStartScript doc comment says it leaves the command's process group and tree.
  - id: F6
    status: resolved
    answer: TestRunMemoryCapped gains the row AChildWithinACapThatIsNoPageMultipleEndsOnItsOwn under testMemoryCap plus 1, which expects no kill and no error.
  - id: F7
    status: resolved
    answer: TestRunMemoryCapped_TheHolderIsNeverAChildOfTheCommand runs exec cat /proc/self/task/*/children in the scope and expects empty output and no error, skipping where /proc lists no children or no scope exists.
---

### [BLOCKING] Mark the two kept contained tests with testtiming keep

**Location:** internal/proc/memorycap_linux_test.go:178

**Issue:** The redundancy report lists `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns` as contained by `TestRunMemoryCapped_TheHolderIsNeverAChildOfTheCommand`.
It lists `TestRunMemoryCapped_AScopeWithoutSetsidIsAnErrorAndNeverRunsTheCommand`, at line 282, as contained by that test and `TestRunMemoryCapped_AScopeNotHeldToTheCapIsAnErrorAndNeverRunsTheCommand`.
Their assertions differ from those of the covering tests.
The holder test asserts that the scope directory disappears after the run returns, which the children test never checks.
The setsid test reaches the script's `command -v setsid` refusal on a scope whose `memory.max` matches.
The cap-mismatch rows exit at the earlier `memory.max` check, and Go coverage cannot see either shell branch.
Both tests stay, so PATTERN-test-speed requires each to carry `//testtiming:keep <reason>` directly above its `func Test…` line, and neither does.

### [BLOCKING] Use proc's cgroup path constants in the holder test

**Location:** internal/proc/memorycap_linux_test.go:185

**Issue:** The test runs `cat /proc/self/cgroup` and joins `"/sys/fs/cgroup"` with the printed cgroup at line 194.
`internal/proc/tasklimit.go` declares both paths once, as `selfCgroupFile` and `cgroupMount`, and production code reads them only through those names.
A change to `cgroupMount` would leave this test reading the old mount, and nothing would flag it.
PATTERN-code-conventions requires one named constant, referenced everywhere, never a repeated literal.
The package already exposes unexported internals to its external tests through `export_linux_test.go`.

## Focus verdicts

- `cgroupMemoryMaxFile`: declared once in the platform-neutral `memorylimit.go` and used by every reader; F1 is resolved.
  The holder test repeats two other path constants, filed as F9.
- The holder runs in its own session through `setsid` inside the `/dev/null` subshell, and `defer release.Close()` ends it on every return.
  `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns` pins that it ends.
- The three new scope-only tests skip where `MemoryCapCoversTree` is false.
  The children test asserts that the holder is no child of the command, and `TestRunMemoryCapped` runs a cap that is not a page multiple.
- The gatecli hog is bounded to a fixed 256 MB above its 64 MB cap and its comments match the command.
- The summary's run ids, count and overlapping spans match the gate records.
  Those runs predate commit `6bdced24`, which added the setsid check.
  I re-ran the same 50-count command at HEAD: 1000 passed, 0 failed (`20261118T150648Z-tier1-1030999`).

## Rejected

- The assistant's carry of F5 as open, for a race between the backgrounded `setsid` and the command's start, is rejected.
  F5's prescribed fix landed.
  The window lasts only until the holder runs `setsid`, so only a command that signals its own group at once could hit it.
  Closing it would need a startup handshake the board entry does not ask for.
- The assistant's F9 repeats this review's own finding on the cgroup path literals and is merged into F9.
