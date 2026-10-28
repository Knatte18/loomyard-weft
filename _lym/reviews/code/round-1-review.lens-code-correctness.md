```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens on `gitBranchReadsOnly` and the new `case "branch"` in `internal/burlerengine/cluster.go`.

**Per-branch list (case run, result):**
- **`--` handling.** Precedence is `listMode || (!hasOperand && i+1 == len(args))`. `git branch --` returns true, which is correct (bare listing). `git branch -- new` returns false, `git branch -l -- feat*` returns true, and `git branch foo --` returns false. All correct.
- **Long options.** `--contains=x` is cut on `=` and known. `--sort -committerdate` skips its value with `i++`. `--points-at` as the last word skips past the end and the loop exits safely. `--set-upstream-to=origin/main` is unknown, so it returns false. All correct.
- **Short clusters.**
  - `-avv`: every letter is known, listMode becomes true, returns true. Correct.
  - `-aD old`: `D` is unknown, returns false. Correct.
  - `-l -d old`: `-d` is unknown, returns false. Correct.
  - `-` alone and the empty string fall to the operand case, which is safe.
- **Operand and list-mode order.** `git branch foo -l` returns true because the final `listMode` wins. Git treats `foo` as a pattern there, so this is correct.
- **Final return `listMode || !hasOperand`.**
  - `git branch -a foo` returns false. Correct.
  - Bare `git branch` returns true. Correct.
  - `git branch --show-current x` returns true. Git handles `--show-current` before any create path, so this is correct.
- **`case "branch"` in `gitInvocationMutates`.** The help-request exception runs first and is unchanged. When `gitBranchReadsOnly` is false, control falls through to `mutatingGitSubcommands["branch"]`, which is still present, and then to `repos.contain`. The fork-created-repository exception is therefore preserved. Correct.

**Findings:**

1. **MEDIUM, design.** `-v`/`--verbose` is marked as list mode, so `git branch -v foo` and `git branch --verbose foo` are classified read-only.
   - Location: `internal/burlerengine/cluster.go:52-62` (`gitBranchReadOptions["--verbose"]`, `gitBranchReadShortOptions['v']`).
   - As I recall git's `builtin/branch.c`, `-v` only sets `filter.verbose`. Git enters list mode when there is no operand, or when `--list` or a `--contains`/`--merged`/`--points-at`-style filter is given. The `git branch` man page describes `-v` as acting "when in list mode", which supports this. `git branch -v foo` should therefore create `foo`, the same case as `-a foo`, which the diff correctly keeps mutating.
   - The record's list-mode set includes `-v`, and the board entry lists it too. That follows the entry rather than git's own mode selection, which is what the record's second decision claims to follow.
   - Confidence: moderate. I could not run git without writing to a repository.
   - Fix: set `-v` and `--verbose` to false, so they pass only without an operand or beside a true list-mode option.
   - Test gap: no row covers `git branch -v foo`.

2. **LOW, design.** Optional-value options given with a separate word, such as `--abbrev 8` or `--column always`, read the value as an operand. The command is classified mutating, which fails safe and is loud. It does not follow git, and no acceptance criterion needs it, so I treat it as acceptable. I report it for completeness.
   - Location: `internal/burlerengine/cluster.go:52-62`.

Everything else I traced is correct.

**Tool calls:** none made. The inherited context already held the diff and the surrounding code.
