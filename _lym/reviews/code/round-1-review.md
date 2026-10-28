---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/burlerengine/cluster.go:52-65
    summary: -v and --verbose are marked list mode, so git branch -v foo and git branch --verbose foo pass as read-only although git creates foo
    fix: In gitBranchReadOptions set "--verbose" to false and in gitBranchReadShortOptions set 'v' to false, so -v passes only with no operand or beside a true list-mode option (-l, --contains, --no-contains, --merged, --no-merged, --points-at, --show-current); update the list-mode comments, and in doc.go's fork-audit paragraph name -v among the options that alone do not allow an operand. Add TestFirstMutatingGitCommand rows "git branch -v foo", "git branch --verbose foo" and "git branch -vv foo" as mutating, plus "git branch -v" and "git branch -v -l feat*" as not mutating.
    origin: handler
  - id: F2
    severity: NIT
    class: design
    location: internal/burlerengine/cluster_test.go:309
    summary: No row pins the separate-word value skip for --format and --points-at, or the trailing-double-dash branch of gitBranchReadsOnly
    fix: Add rows "git branch --format -x" and "git branch --points-at -x" as not mutating, and "git branch --" as not mutating plus "git branch foo --" as mutating, in TestFirstMutatingGitCommand.
    origin: lens:code-tests
---

### [BLOCKING] -v and --verbose wrongly treated as list mode

**Location:** internal/burlerengine/cluster.go:52-65
**Issue:** `gitBranchReadOptions["--verbose"]` and `gitBranchReadShortOptions['v']` are true, which the decision record's list-mode set copies from the board entry. The record's own Bound and second decision say the exception must follow git's own mode selection, so the audit never passes a command that creates a branch. Run against git here, `git branch -v zzv1` and `git branch --verbose zzv2` created branches `zzv1` and `zzv2`, while `git branch --contains HEAD zzv4` and `git branch --show-current zzv5` created nothing. `-v` therefore behaves like `-a`, which the diff correctly keeps mutating beside an operand. `git branch -v foo` and `-vv foo` thus pass the audit, and a fork can create a branch unseen. No test row covers it, which is why the test run does not catch it. The record's acceptance criteria themselves do not name `-v` with an operand, so the fix conforms to the record's stated principle.

### [NIT] Value skip and trailing double dash unpinned

**Location:** internal/burlerengine/cluster_test.go:309
**Issue:** The `--format %(refname) foo` row is mutating with or without the value skip, and `--points-at HEAD` passes whether or not `HEAD` is skipped; only `--sort -committerdate` pins the skip. The `--` branch `!hasOperand && i+1 == len(args)` has no row for a bare `git branch --`.

## Rejected

- Optional-value options such as `--abbrev 8` read their separate-word value as an operand (code-correctness LOW): fails safe by classifying mutating, and no acceptance criterion needs it.
- `internal/fabricengine/refscanner.go` has its own `git branch` classifier (code-affected-set note): a different purpose, untouched by the entry.
