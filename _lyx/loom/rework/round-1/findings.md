# PR-Gate review: agent-naming

## F1: rename the repo's short code from "code" to "shortname"

The operator finds "code" too vague: it reads as source code and does not say it is the repo's short name.
Rename it to "shortname" everywhere it appears, with no behaviour change:

- The verb `lyx fabric code [<code>]` becomes `lyx fabric shortname [<shortname>]`.
- The clone flag `--code` becomes `--shortname`.
- The record file `.lyx-code` becomes `.lyx-shortname`.
- Every refusal, warning and help text that names the verb, the flag or the record names the new spelling.
- Go identifiers, file names and test names follow (for example `CodeFileName`, `ReadCode`, `WriteCode`, `ValidateCode`, `StandaloneCode`, `NameCode`, `codebinding.go`, `codecommit.go`, `fabriccli/code.go`), so no "code" spelling for this concept is left.
- Docs follow: `docs/overview.md`, `CONSTRAINTS.md`, `contracts/specs/refusal-spec.md`, the module docs and package doc comments.

The grammar `[a-z][a-z0-9]{1,5}`, the record's place on weft:main, and the name forms `<shortname>:<slug>:<role>` and `<shortname>:<role>` stay as they are.
No other finding: the rest of the PR meets the task's goal.
