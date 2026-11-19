---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/testkit/tmuxkit/gitentry.go#runCheckingGitEntries
    summary: Goal point 1 asks to find the writer of the stray temp-dir .git and fix that test; the range adds instrumentation only and fixes no writing test, and no Added after Discussion entry licenses the deferral.
    fix: Find the test, or the code it drives, that writes .git directly into the lym temp root or the system temp directory, and change it to write only under its own t.TempDir, with a regression row in the test of that behavior. If the writer cannot be found in this run, leave the instrumentation in place and defer this finding to the run's parent so it can record the deferral with lym loom decision add; the fixer cannot record it.
    origin: lens:code-scope
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:146
    summary: The rule that fails a package seen alone still fails packages that did not write the entry, which goal point 2 forbids, through a non-test writer such as a detached daemon, a test binary that exits before the snapshot, or an unreadable /proc marked attributed.
    fix: In checkNewGitEntries, set code 1 only when the sighting positively ties the entry to this binary, for example a descendant of this binary working in dir, and report every other sighting as a machine finding that keeps the code. In snapshotProcesses, return attributed true only when the table it read holds this binary's own pid. Update the tmuxkit package doc, the gitEntrySighting field docs and the .git bullet of pattern/PATTERN-test-isolation.md to the new rule, and change the TestCheckNewGitEntries and TestRunCheckingGitEntries rows to it, adding a row where a sighting with no other test binary and no descendant in dir keeps the code.
    origin: lens:code-scope
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/testkit/tmuxkit/tmuxkit_test.go#TestClassifyProcesses
    summary: The redundancy report lists TestClassifyProcesses as contained by the live snapshot test, it is kept because it pins what that test cannot see, and PATTERN-test-speed requires a kept candidate to carry a testtiming keep directive, which it lacks.
    fix: Add the line //testtiming:keep pins the exact other-test-binary list, the exclusion of a test-binary descendant of self and the pid-name format and order of others and inDir directly above func TestClassifyProcesses in internal/testkit/tmuxkit/tmuxkit_test.go.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: consistency
    location: internal/testkit/tmuxkit/gitentry.go:167
    summary: The finding text and pattern/PATTERN-test-isolation.md state that the binary every such finding names is the writer, which is false, since the writer never names itself and a recurring neighbour stays in every list.
    fix: In describeGitEntry, replace the clause the writer is one of them or this package, and the binary named in every such finding is it with a clause saying the writer is one of them, this package or a process that is no test binary. In pattern/PATTERN-test-isolation.md, replace the binary that every such finding names is the writer with a sentence saying the binaries named are candidates running at that moment, not the writer. Change the summary.md bullet that says the binary every finding names is the writer the same way.
    origin: assist:assist
  - id: F5
    severity: LOW
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:119
    summary: stop closes the fswatch watchers before draining, so a create event queued when m.Run returns is dropped, and an entry this package wrote and another package already removed then passes with no finding at all.
    fix: In the stop closure of watchGitEntries, before closing each watcher, drain its Events channel of every event already delivered without blocking, or record a sighting when any create event arrives during stop, so a create queued before m.Run returned still yields a sighting. Add a TestRunCheckingGitEntries row whose run creates and removes the entry and returns without waiting for the snapshot, asserting a finding is printed.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/testkit/tmuxkit/gitentry.go:114
    summary: watchGitEntries keeps only the latest sighting per path, so an earlier sighting of a create is lost to a later one after a remove and re-create, with its time and process lists.
    fix: Keep every sighting per path, as map[string][]gitEntrySighting, and have checkNewGitEntries report each one and decide the code from all of them; add a TestCheckNewGitEntries row with two sightings of one path asserting both are reported.
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/testkit/tmuxkit/gitentry.go#watchGitEntries
    summary: Several new behaviors have no test row that fails when they break, namely the create-only event filter, the skipped directory whose watch cannot open, the off-Linux unattributed snapshot, the cyclic or missing parent chain in descendsFrom and the cut at maxArgvChars in quoteArgv.
    fix: In TestRunCheckingGitEntries add a row whose dirs include a directory that does not exist, asserting the run neither hangs nor loses the entry in the other directory. In TestClassifyProcesses add a test binary whose parent chain loops and one whose parent is missing from the table, both counted as others, and a process in dir whose argv exceeds maxArgvChars, asserting the cut inDir string. Pin the off-Linux branch of snapshotProcesses with a row in a test file built off Linux asserting attributed false and at kept.
    origin: lens:code-tests
  - id: F8
    severity: NIT
    class: design
    location: internal/testkit/tmuxkit/gitentry.go#quoteArgv
    summary: quoteArgv cuts the quoted string by bytes, which can split a UTF-8 rune, while its doc says it cuts to maxArgvChars characters.
    fix: In quoteArgv, cut at the last rune boundary at or before maxArgvChars bytes, for example by converting to []rune and cutting at maxArgvChars runes, so the doc's characters holds.
    origin: lens:code-correctness
---

### [BLOCKING] The writing test is not found or fixed

**Location:** internal/testkit/tmuxkit/gitentry.go#runCheckingGitEntries
**Issue:** The board entry's Idea 1 asks to find the writer and fix that test.
The range adds the watch and the `/proc` snapshot, which serve that hunt, and changes no writing test.
`_lym/landing/summary.md` says so under Design decisions, as the writer's own call rather than a parent correction.
The record holds no `Added after Discussion` entry.
The fswatch `package fswatch_test` move and the pattern edit are forced by the tmuxkit change and are no scope finding.

### [BLOCKING] A package with no other test binary beside it is blamed without evidence it wrote the entry

**Location:** internal/testkit/tmuxkit/gitentry.go:146
**Issue:** Idea 2 asks that a package whose own run did not create the entry is not failed.
`checkNewGitEntries` fails any package whose sighting is attributed with no other `*.test` binary.
Three paths reach that without this package writing the entry:
a writer that is no test binary, such as a detached daemon a finished test left, which the diff documents as a bound;
a test binary that writes the entry and exits before the consumer goroutine reads `/proc`;
and `Pids()` returning nil when `/proc` cannot be listed (`proc.go:15-18`), which `snapshotProcesses` still marks `attributed: true`, contradicting the field's own doc at `gitentry.go:44`.
The first is plausible here: the summary names `reedcli` and `loomcli` as suspects, and reed's watchdog daemon detaches and outlives its test.
The board entry's own alternative, a machine finding that fails no package that did not create the entry, avoids all three.

### [BLOCKING] TestClassifyProcesses lacks its keep directive

**Location:** internal/testkit/tmuxkit/tmuxkit_test.go#TestClassifyProcesses
**Issue:** The redundancy report lists it as contained by `TestSnapshotProcesses_NamesAChildWorkingInTheDirectoryAndLeavesOutThisBinary`.
Its assertions pin what the live test does not: the exact others list, a test-binary descendant of self left out, a non-test binary left out, and the order and format of both lists.
So it is kept, and `pattern/PATTERN-test-speed.md` requires a kept candidate to carry `//testtiming:keep <reason>` above its `func Test…` line.

### [MEDIUM] The finding text claims the common binary is the writer

**Location:** internal/testkit/tmuxkit/gitentry.go:167
**Issue:** Each observer leaves itself and its descendants out of others, so the writer's own finding never names it.
A neighbour that recurs beside the writer stays in every other finding, as does a non-test writer's absence.
The output line, the `.git` bullet of `pattern/PATTERN-test-isolation.md` and the landing summary state the claim as fact.

### [LOW] Stopping the watch drops a queued create event

**Location:** internal/testkit/tmuxkit/gitentry.go:119
**Issue:** `fswatch.Close` closes `done` before the forwarder finishes, and `forward` returns on `done` over a pending event or a blocked send (`fswatch.go:66-98`).
An entry this package created in its last moments, and another package removed before this check, then gets neither a sighting nor the after-run existence check, so nothing is reported.
The old before-and-after check missed that case too, so this is no regression, but the watch was added to close it.

### [LOW] A later sighting overwrites an earlier one

**Location:** internal/testkit/tmuxkit/gitentry.go:114
**Issue:** After a create, a removal by another package and a second create, only the second sighting survives.
Its time and process lists replace the first, which loses evidence the hunt for the writer needs and can change the exit code.

### [LOW] New behaviors with no pinning row

**Location:** internal/testkit/tmuxkit/gitentry.go#watchGitEntries
**Issue:** No row sends a non-create event, opens a watch that fails, runs the off-Linux branch of `snapshotProcesses`, walks a looping or broken parent chain of a test binary, or quotes an argv longer than `maxArgvChars`.
Removing any of those branches leaves every test green.

### [NIT] quoteArgv cuts by bytes

**Location:** internal/testkit/tmuxkit/gitentry.go#quoteArgv
**Issue:** `quoted[:maxArgvChars]` slices bytes, so a multibyte rune at the cut is split, and the doc promises characters.

## Rejected

- Handler candidate, `testBinarySuffix` declared in both `internal/testkit/tmuxkit/gitentry.go` and `internal/gatecli/watchdog.go` against PATTERN-code-conventions' named-constant rule: not a violation, since `.test` is the Go toolchain's fixed suffix and not a lym name or path that can go stale, and testkit may not import a `*cli` package.

## Merged

- F2 merges lens code-scope's MEDIUM non-test-writer finding, lens code-correctness's MEDIUM `Pids()`-nil finding and the assistant's F2.
- F5 merges the assistant's F4; F6 merges the assistant's F5; F3 repeats the assistant's F3 and the code-tests lens's NIT, raised to BLOCKING as a PATTERN-test-speed violation.
- F7 merges the code-tests lens's five LOW findings and the assistant's F7 and F8, kept at LOW since each branch is small and its doc states it.

## Focus outcomes

- A `.git` created before the watch opens, or one no watch saw, takes the documented machine-finding path and keeps the code; the queued-event drop is F5, and the wrong-blame races are F2.
- `/proc` reads that fail midway skip the process, which only loses blame; an entirely unreadable `/proc` is F2. Descendants of this binary are left out correctly, including a re-executed helper and its `git` child.
- The fswatch `testmain_test.go` move is forced by the new import of fswatch and fixes no writer; the pattern edit is forced by the new rule. Neither is unlicensed scope; the missing writer fix is F1.
- Both `Loosened-Check:` trailers name the loosened assertion and a measured cause, 22 gate runs between 2026-11-06 and 2026-11-18, which the assistant reports finding in the gate logs. The rewritten tests still fail a lone package's run in `RunCreatingAnEntryWhileAloneFails` and `EntrySeenWhileNoOtherTestBinaryRanFailsAndIsRemoved`.
- The off-Linux fallback is borne out by `snapshotProcesses` and the Windows call site in `Main`; its missing test is F7.
- The move into `gitentry.go` keeps every behavior of the old check: the before snapshot, the removal of an empty directory and the by-hand line for a non-empty directory, gitfile or symlink. The new doc comments follow the repository's conventions.
