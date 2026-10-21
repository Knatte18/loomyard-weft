# Support log — deploy-prod-cleanup

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Version probe (`lyx --version`) to recognise lyx: runs an unknown program from PATH, and the Ubuntu LyX binary would start a GUI process or block.
- Comparing file content or size with the production binary: a stale lyx copy differs from the fresh build, so it would read as foreign.
- Deleting `origin/prod` from the deploy tool: an outward-facing, one-time action better left to the operator.
- A separate warning for a foreign binary that shadows production lyx: not asked for, and the existing PATH check already covers a missing Go bin dir.

## Review rounds

_No rounds yet._

## Question ledger

- Should the pushed-to-main refusal go with `prod`? Self-pick: no, it guards a clean, pushed `main` independent of the pointer.
- Which identity marks a lyx binary? Self-pick: the main package path in Go build info.
- Should the remote `prod` branch be deleted? Self-pick: no, left to the operator.
