---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/cluster_test.go:381
    summary: The & consumption after a redirection operator, which lets 2>&1 keep a later mutating subcommand in its segment, has no row that fails without it
    fix: In TestFirstMutatingGitCommand add the row {"fd duplication before the subcommand hides no mutating verb", "git 2>&1 commit -m x", true} beside the row for git 2>/dev/null commit -m x. Without the & skip in splitShellSegments (cluster.go:422-424) the & ends the segment after git, so this row flips to false.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/cluster.go:412
    summary: The file-descriptor check reads word.String() after quote and escape removal, so a quoted or escaped digit operand before a redirect is dropped as an fd and git branch with a quoted 2 then a redirect passes as read-only
    fix: In splitShellSegments add a wordQuoted flag that is set in the quote-open case (r == '\'' || r == '"') and in the backslash-escape case, and cleared wherever word.Reset() runs and where inWord is set false. In the r == '<' || r == '>' case treat the current word as a file-descriptor number only when wordQuoted is false. Then add the row {"quoted digit before a redirect is an operand", `git branch "2">x`, true} to TestFirstMutatingGitCommand.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/cluster_test.go:380
    summary: Three D1 behaviours of the redirect case have no row that fails without them a dangling redirect before a separator, a detached digit operand and a backslash-escaped redirect character
    fix: In TestFirstMutatingGitCommand add three rows after the quoted redirect row. First, {"redirect with no target before a separator drops no later word", "git branch -a >; git commit -m x", true}, which pins the dropNextWord reset in endSegment (cluster.go:380). Second, {"digit word detached from the redirect is an operand", "git branch 2 > x", true}, which pins that the fd check needs the digits attached. Third, {"escaped redirect character is an operand", `git branch \>`, true}, which pins that a backslash-escaped redirect stays in its word.
---

### [MEDIUM] Redirect `&` skip is not pinned by any row

**Location:** internal/burlerengine/cluster_test.go:381 (code at internal/burlerengine/cluster.go:422-424)
**Issue:** The `2>&1` row passes with or without the code that skips an `&` attached after the operator: the `&` then ends the segment, the `1` lands in a non-git segment and `git branch` is still read-only.
The one case where the skip changes a verdict is a redirect ahead of the subcommand, `git 2>&1 commit -m x`, which without the skip is split after `git` and the mutating `commit` goes unseen.
D1 names the attached `&` as part of what is dropped and D2 asks for a redirection that cannot hide a later mutating subcommand, so this behaviour needs its own row.

### [LOW] A quoted or escaped digit operand before a redirect is dropped as a file descriptor

**Location:** internal/burlerengine/cluster.go:412
**Issue:** The file-descriptor test looks only at the built word, after the quote and escape cases have stripped their marks, so `git branch "2">x` and `git branch \2>x` lose the operand `2`.
In a shell those spell the operand `2` plus a redirect, and the command creates a branch; the guard now passes it where it used to flag it.
This breaks D1's bound that only the redirection target leaves the words, and the `splitShellSegments` comment's claim of an attached file-descriptor number.
The reach is small, one digit-only branch name beside a redirect, which is why it is LOW.

### [LOW] Three D1 behaviours of the redirect case have no row

**Location:** internal/burlerengine/cluster_test.go:380
**Issue:** No row fails if `endSegment` stops resetting `dropNextWord`, which would swallow the first word of the next segment after a redirect with no target.
No row fails if the file-descriptor check stopped needing the digits attached, so `git branch 2 > x` would read `2` as a descriptor and pass.
Only the quoted form of an operand redirect character is pinned, never the backslash-escaped form D1 also names.
Each is a behaviour the diff adds with no coverage.

## Rejected

- `>|` and `&>` leave the redirect target as a stray one-word segment (code-correctness): the same gap existed before the diff, no command that was flagged is now passed, and D1 lists neither form.
- `<(...)` and `>(...)` process substitution read as words (code-correctness): the inner command was never judged before, and the decision record's open risks cover forms the splitter does not model.
- Here-document bodies split into segments (code-correctness): named in the decision record's open risks as keeping their current reading.
- Rows 380 to 385 duplicating each other (code-tests): each pins a distinct behaviour; the issue chain row at 385 passes without the fix, but D2 and its acceptance criterion require it.
- `firstMutatingGitCommand` doc comment not mentioning redirections (code-affected-set): the comment describes segments, which are unchanged, and the new `doc.go` and `splitShellSegments` text carry the redirect rule.
- Missing PATTERN entry or `docs/overview.md` change: no entry governs this lexical guard, and CLAUDE.md asks for those only on a new module, a CLI change or a cross-cutting invariant.
- D3 and D4 touched by the diff: `forkViolation` and the `git branch` list-mode rules are unchanged in the diff, so scope holds.
