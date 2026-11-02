---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: handler
    location: docs/benchmarks/test-suite-timing.md:780
    summary: The new report's revision line, "c536908e plus the run-directory name fix of this task", names no revision a reader can check out
    fix: In the `revision:` line of the new section's Setup block, replace the text with the commit hash of the tree the six runs measured; that is `e1770b2a` when the measured tree equals that commit's tree. Part 2 of the decision record asks for the revision to be recorded.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: docs/benchmarks/test-suite-timing.md:812
    summary: The pair 2 explanation (the 14:24 to 14:30 window, another worktree holding a gate slot) and the "8704 files and 35 MB" figure follow from no table in the section
    fix: In the new section, add the start and end wall-clock time of each run to the Runs table, or add a sentence giving the slot holder's window in a form a reader can check, so the "ran alone from about 14:24 to 14:30" claim follows from the report's own data. Give the source of the 8704-file and 35 MB figure, such as the `find` and `du` sample, in the line that states it.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/lymtemp/run.go:24
    summary: runNameDigits counts letters as well as digits, and its comment pins a derivable tally ("over a million names")
    fix: Rename `runNameDigits` to `runNameLength` in `internal/lymtemp/run.go` (the declaration, the `for range` in `randomRunName` and the doc comment of `randomRunName`). In its comment, delete the line "Four characters still leave over a million names for the retry bound to draw from.", since `len(runNameAlphabet)` and the length derive it.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/testkit/tmuxkit/tmuxkit_test.go:34
    summary: The row "run directory of the widest uid" still builds its path with the retired six-hex name r0a1b2c
    fix: In `TestCheckSocketPath`, change the row's directory from "/tmp/lym-4294967295/r0a1b2c" to "/tmp/lym-4294967295/r0a1b" so it has the shape `lymtemp` makes now.
---

### [LOW] Report revision names no checkable revision

**Location:** docs/benchmarks/test-suite-timing.md:780
**Issue:** Part 2 asks the report to record the revision. The new section gives "c536908e plus the run-directory name fix of this task", the parent tip plus an uncommitted change, which no one can check out. The report is the record of one measurement, so its revision is what lets a later reader tie the numbers to a tree.

### [LOW] Pair 2 explanation and shm file figures are not derivable from the section

**Location:** docs/benchmarks/test-suite-timing.md:812
**Issue:** The section says pair 2's default run "ran alone from about 14:24 to 14:30" because another worktree held a gate slot. The Runs table carries no clock times, so the window cannot be checked against the report. The "at most 8704 files and 35 MB" line also names no table or sampled quantity it comes from. The answers do not depend on pair 2, so this weakens the report's traceability only, not its conclusion: the two concurrent pairs' ratios (0.982 and 0.932) follow from their own rows.

### [NIT] runNameDigits names digits but holds letters too

**Location:** internal/lymtemp/run.go:24
**Issue:** The constant counts every character after the prefix, and the alphabet is letters and digits. Its comment also states a derived count that goes stale if the alphabet or the length changes, which the prose rule on perishable specifics bars.

### [NIT] A widest-uid fixture keeps the retired run-directory shape

**Location:** internal/testkit/tmuxkit/tmuxkit_test.go:34
**Issue:** The row passes either way, since a `/tmp` root is far under the limit, but its name `r0a1b2c` is the six-hex shape `lymtemp` no longer makes. The new row below it pins the real shape at the limit, so the old row only misleads.

## Rejected

- `internal/lymtemp/root_test.go:153-154` uses `runPrefix+"123456"` (code-tests, code-affected-set): the fixtures exercise the prefix-only inherited-directory rule, no path length rests on them, and the name is an arbitrary string.
- `docs/benchmarks/test-suite-timing.md:736` and `:767` still show `r49ee2f` and `r<hex>` (code-affected-set): they sit in the earlier measurement report, which records what was true when it ran.
- Name space of four characters against the retry bound (focus item 3): 36^4 names against 16 draws leaves a negligible chance of 16 taken names with any realistic count of live and stale directories, and the exhausted bound still fails with an error naming the root, which `TestNewRun_RetriesATakenNameUpToTheBound` pins.
- The new `TestCheckSocketPath` row duplicates the widest-uid row (focus item 4): the rows differ in root length (`/dev/shm` against `/tmp`), and only the new one sits at 107 bytes and fails on a 109-byte name; building it from `lymtemp.Dir()` makes it fail if the name grows.
- `lymtemp` doc and constant comments name `/dev/shm` and the 107-byte limit (code-affected-set): the comments state why the length is four, a hidden constraint the code-quality rule on comments allows.
- Part 1's cut is not justified in the code (focus item 5): the change description records why the run name was the cheapest cut, and the doc and PATTERN entry for the new format both land in the diff, which is all the repository's rules require for a run-name change.
- Comparison at 107 bytes (focus item 1): the byte count from the code is 8 + 15 + 6 + 16 + 62 = 107, and `checkSocketPath` refuses only above `maxSocketPath`, so 107 is accepted; not a defect.
