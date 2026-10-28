```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it.

**Per behaviour, the pinning test (all rows in `TestFirstMutatingGitCommand`, `cluster_test.go`)**
- Bare `git branch` passes: "bare git branch is not mutating".
- No-operand listing with non-list-mode options passes: `-a` and `-r` rows; `-avv` covers the letter-by-letter cluster read.
- List-mode option makes operands patterns:
  - `-l 'feat*'`, `--list feat*` and `-l -- feat*` pin `-l`/`--list`.
  - `--merged main` and `--no-merged main` pin those two.
  - `--points-at HEAD` pins `--points-at`.
  - `-a --contains 71a46b11` pins `--contains` as a separate word.
  - `--show-current` pins `--show-current`.
  - `-v`/`--verbose` as list mode with an operand: **none**.
  - `--no-contains` with an operand: none, but it shares the `--contains` map path, so it is not a separate finding.
- `--name=value` form: `--contains=71a46b11` row pins it (the pass relies on the `=` cut). `--set-upstream-to=origin/main` pins that the `=` form of an unknown long option stays mutating.
- Valued option skips its value, so `-committerdate` is not read as an option: the `--sort -committerdate` row pins `--sort`, since without the skip `c` is an unknown letter and the command would be mutating.
  - `--format <value>` and `--points-at <value>` skip: **none**. The `--format %(refname) foo` row is mutating either way, and `--points-at HEAD` passes through list mode whether or not `HEAD` is skipped.
- Writing forms stay mutating:
  - `new`, `-D`, `-m`, `-c`, `-u`, `-f`, `--unset-upstream` and `--set-upstream-to=` each have a row, and the existing `-d old` row remains.
  - `-a foo` and `--format ... foo` pin "non-list-mode option plus operand creates".
  - `-l -d old` and `-aD old` pin an unknown letter or option beside a read-only one.
  - `-- new` pins the post-`--` operand with no list mode.
- `case "branch"` leaving the help and fork-created-repository exceptions unchanged: existing rows, none changed by the diff.
- Doc and comment edits: static prose, exempt.

**Findings**

1. LOW, design, `cluster_test.go` rows 298–312 and 357–370 (new rows), against `cluster.go` `gitBranchReadShortOptions` (`'v': true`) and `gitBranchReadOptions` (`"--verbose": true`).
   - `-v` and `--verbose` as list mode with an operand have no row, although the decision record lists them as list-mode options.
   - Changing `'v'` or `"--verbose"` to `false` would break `git branch -v foo`, a list in git's own mode selection, and no row would fail.
   - The focus directive names this case. Add rows `git branch -v foo` and `git branch --verbose foo` as not mutating, if git lists there.

2. NIT, design, `cluster_test.go` row "branch --format with a branch name creates it", against `cluster.go` `gitBranchValuedReadOptions`.
   - The row passes with or without the value skip, so it does not pin the skip for `--format`. No row pins it for `--points-at` either.
   - Only `--sort -committerdate` does.
   - A row like `git branch --format -x` as not mutating would pin it.

3. NIT, design, `cluster.go` `gitBranchReadsOnly`, `case arg == "--"`: `return listMode || !hasOperand && i+1 == len(args)`.
   - The trailing branch (a bare `git branch --` passes, `git branch foo --` fails) has no row.
   - The `-- new` and `-l -- feat*` rows cover only the `listMode` and operand-after-`--` sides.

**Tool calls:** none. The diff and `cluster.go` were already in context.
