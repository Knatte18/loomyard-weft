# Webster-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each, with new `TestClassifyViolation_FabricReferenceByCommand` rows for F1–F4.
After the last fix, `go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass.

| Finding | Commit | Change |
|---|---|---|
| F1 | `a235907f7` | `readOnlySegment` resolves its program through the new `programName`: a bare name is trusted, and a path-spelled program only from `/bin`, `/usr/bin` or `/usr/local/bin` (`systemBinDirs`), so `./cat` and `/tmp/tools/grep` are mutating while `/usr/bin/cat` and `/bin/bash -c` keep their class. |
| F2 | `ea5c8d7c1` | git's `-c`, `--config-env` and `--config-env=` global options make the command mutating, and so does `git grep` with `-O` in a short-option cluster or any `--open` long option (`grepOpensPager`, which stops at `--`). |
| F3 | `e706ae5e9` | `fi` and `done` join `shellKeywords`, `for` and `select` headers are read-only (`loopHeaders`), and `[[` and `read` join `readOnlyTools`; a loop that runs `rm`, or whose word list substitutes one, stays correctness. |
| F4 | `fc4bda709` | `commandWrappers` maps each wrapper to its value options, which the new `skipOptions` skips along with each value, as it also does for `timeout` and for `xargs` (which gains GNU's long value options); GNU `time -o` is mutating; the shell branch reads the first operand as the `-c` string or a script and skips the values of `-o`/`-O`/`+o`/`+O`/`--rcfile`/`--init-file` and of a short cluster ending in `o`/`O`. |
| F5 | `085c17adb` | `readOnlySegment`'s doc, the `ClassifyViolation` fabric-reference line, the `shedadapters/doc.go` WebsterProducer bullet, `webster.go`'s new inline comment, the unlisted-writer test doc and the refusal-spec webster audit sentence now use semantic line breaks. |

## Deferred

Nothing deferred.
