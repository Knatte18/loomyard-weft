---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/burlerengine/cluster_test.go:292
    summary: TestCommandRunsMutatingGit lacks rows for the separate-word --git-dir and the --namespace and --config-env value skips
    fix: In the table of TestCommandRunsMutatingGit, add mutating rows `git --git-dir x commit -m x`, `git --namespace ns commit -m x` and `git --config-env name=VAR commit -m x`, each want true, next to the existing --attr-source, --git-dir= and --work-tree rows. Without them, deleting the map entry for any of these three options goes unnoticed.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/landing/summary.md:9
    summary: The change description says a segment carrying -h or --help passes, but the code and tests pass only a -h or --help among the global options or directly after the subcommand
    fix: In _lyx/landing/summary.md, reword the sentence "A segment carrying `-h` or `--help` passes" to say the escape applies only to a -h or --help among git's global options or as the first word after the subcommand. Also add `--attr-source` to the list of skipped value options, and say that the segment splitter is quote-aware and honours backslash escapes, comments and backslash-newline continuations. The Design decisions line "The `-h`/`--help` escape covers only those literal words" should name the same position bound.
---

### [LOW] Missing rows for three valued global options

**Location:** `internal/burlerengine/cluster_test.go:292`
**Issue:** `gitValuedGlobalOptions` holds seven options.
The table pins the value skip for `-C`, `-c`, `--attr-source` (separate word), `--work-tree` (separate word) and `--git-dir` (`=` form only).
No row exercises `--namespace` or `--config-env` in either form, nor `--git-dir` as a separate word.
Because `git --namespace ns commit` would classify `ns` as the subcommand and let the commit pass, a missing map entry would not be caught by any current row.
The `=` forms share the generic `-` prefix branch of `gitSubcommand`, so one `=` row already covers them and none is requested.

### [LOW] Change description overstates the help escape

**Location:** `_lyx/landing/summary.md:9`
**Issue:** The summary states that a segment carrying `-h` or `--help` passes.
`commandRunsMutatingGit` passes only when the flag is a global option (`helpRequested`) or is `rest[0]`, the word directly after the subcommand.
The test rows `git commit -m -h`, `git tag -m --help v1` and `git checkout -- -h` expect true, which contradicts the summary's wording.
The summary also omits `--attr-source` from the skipped value options and describes the splitter as a plain split at `;`, `&`, `|` and newlines, although it is now a quote-aware tokenizer.
This is a claim the diff does not bear out.

## Focus questions settled

- The `-h`/`--help` escape cannot be triggered by an option value or by a word after `--`.
  It tests only the global-option region and `rest[0]`, and a valued global option's value is skipped, so `git commit -m -h`, `git tag -m --help v1`, `git checkout -- -h` and `git -c -h commit` all classify as mutating.
  The first three have rows; `git -c -h commit` follows from the same code path.
- Quoted `-c` and `-C` values stay one word, single or double quoted, so the subcommand is found.
  Rows cover all three quote shapes.
- A backslash-newline between `git` and its subcommand is dropped without ending the word or segment, so the call stays one segment and classifies as mutating.
  Rows cover both the break between words and the break inside a word.
- Every option in `gitValuedGlobalOptions` has its value word skipped.
  Row coverage is incomplete, as F1 records.
