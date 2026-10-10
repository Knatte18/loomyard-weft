# Card 1 — gitexec-run-stdin

**Edit:**
- `internal/gitexec#runCore` -> `func runCore(args []string, cwd, stdin string) (stdout, stderr string, exitCode int, err error)`
- `internal/gitexec#RunGit`
- `internal/gitexec#Run`
- `internal/gitexec/gitexec_test.go`
- `internal/gitexec/doc.go`

**Create:**
- `plan:internal/gitexec#RunStdin` -> `func RunStdin(args []string, cwd, stdin string) (string, error)`

**Intent:** Give gitexec a checked form that feeds the git command its stdin, so a caller can hand `git cat-file --batch-check` its object names in one process instead of one `cat-file -t` per object (D4).
`runCore` takes the stdin text and sets it on the command when it is non-empty; `RunGit` and `Run` pass the empty string and keep their behaviour byte for byte.
`RunStdin` is `Run` with stdin: a non-zero exit is a `*GitError` carrying the args, the directory, the exit code and stderr, and an exec-level failure is returned unwrapped, exactly as `Run` documents.
A remote subcommand given stdin runs under the same deadline, process group and low-speed environment as through `Run`, since the two share `runCore`.
The `doc.go` package comment names `RunStdin` as a second checked form beside `Run`, `Run` with stdin, while `RunGit` stays the raw form, so PATTERN-gitexec-checked-call keeps one description of the forms a call site may use.
`TestRun`'s table gains a `stdin` field and a `viaStdin` flag that dispatches a row to `RunStdin` in place of `Run`, and its rows cover `RunStdin`: stdin reaching git (`git hash-object --stdin` echoing a known blob id), a non-zero exit as a `*GitError` carrying the args, directory, code and stderr, and an empty stdin behaving as `Run`.
`TestRun_StdoutOnError` gains the `RunStdin` case that a non-zero exit keeps its stdout.

**ImpactSummary:** One package: runCore gains a parameter and its two callers in gitexec.go pass the empty string; no caller outside gitexec changes.
