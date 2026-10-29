# Support log — pattern-spawn-and-links-scans

## Interview

A darn run holds no interview; the board entry is the whole statement of the change, and the writer settled the open points alone.

## Rejected alternatives

- Scanning test files too, for both scans: rejected because tests plant symlinks as hostile fixtures across fabricengine and other packages, and a spawn-scan fixture holds the forbidden text as data.
- A raw-substring scan for `claude -p`: rejected because the launch line is assembled from fragments and a substring scan would trip on the doc comments that state the rule.
- Routing `tools/deploy/main.go`'s symlink copy through fslink: rejected because the copied link may point at a file, and fslink offers no file-link primitive.
- Placing the links entry under "Fabric and git": rejected because the rule applies to any package that creates a link, not to fabric alone.

## Review rounds

_No rounds yet._

## Question ledger

- Entry names: self-picked `PATTERN-interactive-spawn` and `PATTERN-fs-links`. Resolved.
- Scan file names: self-picked `cmd/lym/interactivespawn_test.go` and `cmd/lym/fslinkonly_test.go`. Resolved.
- Whether to touch `internal/fabricengine/doc.go`'s mention of the CLAUDE.md rule: self-picked no, since CLAUDE.md still holds the rule until the follow-up task deletes it. Resolved.
