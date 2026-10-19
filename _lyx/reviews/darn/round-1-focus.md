---
round: 1
focus:
  - "internal/burlerengine/cluster.go `commandRunsMutatingGit`: does the segment split at `;`, `&`, `|` and newlines plus the first-word-is-git rule still refuse every mutating git call the old regex refused, other than the read-only lookups the board entry names? Settle whether any mutating form (quoted separators, `&&`, path-prefixed git, global options before the subcommand) now slips through."
  - "Global-option skipping: are the value-taking options (`-C`, `-c`, `--git-dir`, `--work-tree`, `--namespace`, `--config-env`) handled for both the separate-word and `=` forms, and can a skipped value word hide a mutating subcommand?"
  - "The `-h`/`--help` escape: does a segment carrying those words ever pass while still mutating, for example when `-h` is an argument value rather than a flag? Check the claim in the change description that the escape is safe."
  - "Scope against the board entry: does the diff change the set of mutating subcommands or anything beyond the guard, its tests and the `doc.go` fork-audit paragraph?"
  - "Tests in `TestCommandRunsMutatingGit`: do the new rows duplicate existing rows, and is any behavior the diff adds (newline separation, `-c key=value` skipping) left uncovered?"
  - "Docs: does the `internal/burlerengine/doc.go` fork-audit paragraph match the implemented rule, and does any other doc or PATTERN text still describe the old regex?"
---

Seed focus from the change description and the repository rules; no round has been judged yet.
