# Card 1 — cli-and-output

**Create:**
- `tools/forkaudit/forkaudit.sh`
- `tools/forkaudit/testmain_test.go`
- `tools/forkaudit/forkaudit_integration_test.go`
- `tools/forkaudit#TestMain`
- `tools/forkaudit#runForkaudit`
- `tools/forkaudit#TestForkauditRefusals`
- `tools/forkaudit#TestForkauditReport`

**Edit:**
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitkit#HermeticGitEnv`
- `internal/testkit/tmuxkit#Main`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/testmain_test.go`

**Intent:**
Create the tool's entry script with its command line, its output and work-directory bounds, project-directory discovery and the report header, so later cards only add extraction and sections (decisions D1, D2, D5's header and zero-fork case).

`tools/forkaudit/forkaudit.sh` (bash, `set -u`, `pipefail`):
- Its header comment is the tool's design doc, as `tools/tokencount/tokencount.sh`'s is: purpose, usage lines, every flag with its default, the outputs, the exit codes (0 success, 1 environment or bounds failure, 2 usage), the work-directory bound and the one-run-per-work-directory rule.
  Every later card that adds a flag or an optional section updates this comment in the same card.
- A variable block right after the header spells each run-record literal once: the webster state path `_lyx/webster/state.json`, the plan directory `_lyx/plan/`, the worktree logs directory `.lyx/logs`, and the records suffix (the repo root's sibling name is the repo root plus that suffix).
  The block also derives the run-record root and the logs root as the first path segment of the plan directory and of the logs directory, with bash parameter expansion and no literal, and the script hands the plan directory and both roots to the jq filters with `--arg`; no filter spells any of them.
  No other line of `forkaudit.sh` or of a `.jq` file under `tools/forkaudit/` spells these literals; the Go tests build the paths from `internal/lyxdirs`'s `LyxDirName` and `DotLyxDirName`.
- The required-command check runs first, before any external command: the script directory comes from `${BASH_SOURCE[0]}` through bash builtins only, the repo root is two levels above it, and `command -v` checks `jq`, `git`, `find`, `awk`, `mktemp`, `realpath` and `date`, plus GNU `date` by `date -d @0`.
  A missing one exits 1 with `forkaudit: missing command: <name>` on stderr.
- Flags this card adds: `--since <date>`, `--until <date>`, `--runs <name>[,<name>...]`, `--projects <dir>` (default `$HOME/.claude/projects`), `--hub <dir>` (default the repo root's parent), `--out <path>` (default `.scratch/fork-audit.md` under the repo root) and `--work <dir>` (default `.scratch/fork-audit/` under the repo root).
  Each flag takes its value as the next argument.
- Exit 2 with the usage text on stderr, writing nothing, on: neither or both of `--since` and `--runs`; `--until` without `--since`; an unparsable date (`date -d` fails); `--until` earlier than `--since`; an empty `--runs` list or an empty name in it; a run name holding anything outside `[a-z0-9-]`; an unknown flag; a flag missing its value.
- Paths are canonicalized with `realpath -m`.
  `--out` naming an existing directory exits 1 naming `--out`; `--out`'s directory missing or not writable exits 1 naming `--out`.
  The directory-missing refusal applies only to an explicit `--out`: a defaulted `--out`'s `.scratch/` directory, absent from a fresh worktree, is created with `mkdir -p` after every check has passed, beside `--work`'s own creation, and the header comment says so.
- `--work` is checked before anything is deleted, and each failure exits 1 naming `--work` and deletes nothing, with its own reason on stderr, checked in this order: it equals or is an ancestor of the repo root, the hub or `--out`'s directory (reason `--work overlaps <which>`), which also refuses a `--work` holding `--out`, since `--out`'s directory is then `--work` or a descendant of it; it exists non-empty without the `.forkaudit-work` marker file (`--work is not a forkaudit work directory`); it cannot be created or written (`--work is not writable`).
  The path comparisons run before the marker check, so a comparison is reached even for a directory the marker rule would also refuse.
  A passing `--work` is created if missing (`mkdir -p`, which creates `.scratch/` for the defaults), marked with `.forkaudit-work`, and cleared of the script's own named intermediate files and subdirectories only, by literal names the script lists, never by a glob over the directory.
- Then the report temp file is created with `mktemp` in `--out`'s directory, and an `EXIT` trap removes it unless the final `mv` onto `--out` has run, so every later failure leaves an existing report intact and no temp file behind.
  The report is written to the temp file and moved onto `--out` last; the script prints the `--out` path on stdout.
- Discovery: the hub's canonical path is encoded as Claude Code names project directories, every character outside `[A-Za-z0-9]` replaced by `-`.
  Under `--since` the project directories are those under `--projects` whose name starts with the encoded hub followed by `-`; under `--runs` a name matches only the directory named exactly the encoded hub, `-` and the name, so `alpha` never matches `alpha-two`.
  The project directory list goes to the work directory.
- The report's header: generation time (one line, `Generated: <UTC time>`, the only line two runs over the same input differ in), the exact arguments, the projects, hub and work paths, the project directories matched, and a line listing each `--runs` name that matched no project directory.
  With no forks the report says `No Webster implementer forks matched.` under the header and the script exits 0.

`PATTERN.md`: replace the `PATTERN-lyxdirs-single-declarer` entry line with exactly the line the decision record's "Added after Discussion" decision gives:

```markdown
- `PATTERN-lyxdirs-single-declarer` — Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs` declares them, bar one variable block heading a `tools/` script.
```

Tests, in package `forkaudit` (test files only, no other Go file in the directory):
- `testmain_test.go` is untagged and has the shape of `tools/tokencount/testmain_test.go`: `TestMain` calls `gitkit.HermeticGitEnv()` and then `os.Exit(tmuxkit.Main(m))`, so the script's own `git` children inherit the hermetic environment.
- `forkaudit_integration_test.go` carries `//go:build integration`.
  `runForkaudit` runs `bash <absolute path of forkaudit.sh>` with `cmd.Dir` set to a fresh `t.TempDir()`, so the jq filters are found only relative to the script, never the cwd, with the given arguments and extra environment, and returns stdout, stderr and the exit code; it skips the test on Windows and fails naming `jq` when `exec.LookPath("jq")` fails.
  Every call passes `t.TempDir()` paths for `--out`, `--work`, `--projects` and `--hub`, so no test touches the worktree's `.scratch/`, and every test is `t.Parallel()`.
- `TestForkauditRefusals` is table-driven; each row sets up temp directories, runs the script and asserts the exit code, that stderr names the flag or command and, for a `--work` row, its own reason, and that `--out`'s directory and `--work` hold exactly what they held before (a pre-existing report keeps its bytes, no extra file appears beside it, nothing is deleted from `--work`).
  Exit-2 rows: neither selector, both selectors, `--until` alone, an unparsable `--since`, an unparsable `--until`, `--until` before `--since`, `--runs ""`, `--runs a,,b`, `--runs Alpha`, `--runs 'a b'`, an unknown flag, `--out` with no value.
  Exit-1 rows: `PATH` set to a directory holding symlinks to `bash` and every required command but `jq` (names `jq`); the same with a `date` stub that exits 1 on `-d @0` (names GNU `date`); `--out` in a directory made read-only with `chmod 0555` (names `--out`); `--out` in a directory that does not exist (names `--out`); `--out` naming an existing directory (names `--out`).
  `--work` rows, each naming `--work` with its reason: a symlink to the repo root (overlap, reached before the marker rule); a marked, empty `--hub` directory reached through a symlink (overlap with the hub); a marked, empty directory that is an ancestor of `--out`'s directory, spelled with `..` (overlap with `--out`'s directory); a marked `--out` directory given as `--work` (overlap); a marked, empty `--work` with `--out` inside a subdirectory of it (overlap with `--out`'s directory); `--work` non-empty without the marker; an existing, marked `--work` made read-only with `chmod 0555`; `--work` under a read-only parent (each not writable).
  Rows needing permission denial skip when `os.Geteuid() == 0`.
  Each such row registers a `t.Cleanup` that restores mode 0o755 on the directory it made read-only, before the chmod, so `t.TempDir`'s removal can delete the marker file inside the read-only `--work` directory.
- `TestForkauditReport` is table-driven over argument sets; card 1 gives it the rows that need no fork placement, over an empty projects directory:
  `--runs ghost`, with `--work` naming a path that does not exist yet, exits 0, prints the `--out` path, writes a report holding the no-forks sentence and a header line naming `ghost` as unmatched, and creates `--work` holding the `.forkaudit-work` marker;
  `--since 2026-10-14`, over a `--work` already marked and holding a stale intermediate file under one of the script's own names and a foreign file `keep.txt`, gives the same no-forks report with no unmatched line, removes the stale intermediate and leaves `keep.txt` with its bytes.

`cmd/lyx/testdata/test-budget.yaml`: add the row `tools/forkaudit: 2`, in its sorted place among the `tools/` rows.

**ImpactSummary:** a new standalone script and test package nothing imports; PATTERN.md changes one entry line at net -2 bytes and the budget file gains one row.

**Verify:**
lyx gate test --tags integration ./tools/forkaudit
lyx gate test ./cmd/lyx -- -run TestTestBudget_PackagesStayWithinBudget
