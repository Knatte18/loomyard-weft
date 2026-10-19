---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: 'internal/burlerengine/cluster.go:43'
    summary: 'The -h/--help escape tests every word of the segment, so a mutating call where -h or --help is an option value or follows -- passes (git commit -m -h)'
    fix: 'In commandRunsMutatingGit, apply the help escape only to words before the subcommand and to the words after it up to a bare -- word, and never to a word that is the value of a preceding value-taking option such as -m, -F or -C. Simplest sound form: treat -h or --help as an escape only when it is the word directly after the global options or directly after the subcommand. Add rows for git commit -m -h and git tag -m --help v1 expecting true.'
    origin: 'lens:code-correctness'
  - id: F2
    severity: MEDIUM
    class: design
    location: 'internal/burlerengine/cluster.go:40'
    summary: 'Whitespace splitting breaks a quoted value of -c or -C, so git -c "user.name=A B" commit reads a stray half of the value as the subcommand and passes; the old regex refused it'
    fix: 'Tokenize each segment with quote awareness (a double or single quoted run is one word) before gitSubcommand skips option values. Add rows for git -c "user.name=A B" commit and git -C "/path with space" commit expecting true.'
    origin: 'lens:code-correctness'
  - id: F3
    severity: MEDIUM
    class: design
    location: 'internal/burlerengine/cluster.go:36'
    summary: 'A backslash line continuation (git, backslash, newline, commit) now splits into two segments, so the mutating call passes; the old regex spanned the newline and refused it'
    fix: 'Before splitting segments, replace each backslash-newline pair in the command with a space (strings.ReplaceAll(cmd, "\\\n", " ")), then split on ; & | and newlines. Add a row with a continuation line before commit expecting true.'
    origin: 'handler'
  - id: F4
    severity: LOW
    class: scope
    location: 'internal/burlerengine/cluster.go:29'
    summary: 'gitValuedGlobalOptions omits --attr-source, which takes a separate value word, so git --attr-source HEAD commit reads HEAD as the subcommand and passes'
    fix: 'Add "--attr-source": true to gitValuedGlobalOptions and a row for git --attr-source HEAD commit expecting true.'
    origin: 'lens:code-correctness'
  - id: F5
    severity: LOW
    class: scope
    location: 'internal/burlerengine/cluster_test.go:276'
    summary: 'No row covers the = form of a valued global option or the separate-word forms of --git-dir, --work-tree, --namespace and --config-env'
    fix: 'Add rows for git --git-dir=x commit -m x expecting true and git --work-tree x commit -m x expecting true in TestCommandRunsMutatingGit.'
    origin: 'lens:code-tests'
---

### [MEDIUM] Help escape matches option values

**Location:** internal/burlerengine/cluster.go:43
**Issue:** `slices.Contains(words, "-h")` runs over the whole segment before the subcommand is read, so `git commit -m -h` (where -h is the commit message), `git tag -m --help v1` and `git checkout -- -h` all pass while mutating. The change description's claim that the escape is safe holds only for -h or --help used as a flag.

### [MEDIUM] Quoted option values break the subcommand skip

**Location:** internal/burlerengine/cluster.go:40
**Issue:** `strings.Fields` splits `git -c "user.name=A B" commit` into `-c`, `"user.name=A`, `B"`, `commit`; the skip consumes only the first half, `B"` becomes the subcommand and the call passes. The old regex found `commit` here, so this is a regression for the quoted `-C "/path with space"` form too.

### [MEDIUM] Backslash line continuation splits the git call

**Location:** internal/burlerengine/cluster.go:36
**Issue:** Newline is a separator, so `git \` newline `commit -m x` yields the segment `git \` (subcommand `\`) and the segment `commit -m x` (first word not git). Both pass; the old pattern's `\s+` spanned the newline and refused it. Multi-line fork commands commonly use continuations.

### [LOW] --attr-source value not skipped

**Location:** internal/burlerengine/cluster.go:29
**Issue:** `--attr-source <tree>` takes a separate value but is absent from `gitValuedGlobalOptions`.

### [LOW] Valued-option forms untested

**Location:** internal/burlerengine/cluster_test.go:276
**Issue:** Only `-C` and `-c` are exercised; the `=` form and the other long options share the branch but have no row.

## Rejected

- Stale `TestMutatingGitPattern` name at `docs/benchmarks/test-redundancy.md:396` (lens:code-pattern-fit): a point-in-time measurement report, exempt from staying current.
- Duplicate-row concern on "flag before a read-only subcommand is not mutating" (lens:code-tests): it pins a different branch than the existing `-C` row.
- Subshell, environment-assignment and `bash -c` forms staying unclassified: the old regex missed them equally, so not a regression and recorded in the decision record.
