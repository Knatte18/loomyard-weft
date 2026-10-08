# Webster round 2 — fixer report

The review's one finding was validated against the code and is fixed.
Verify: `go build ./...` and `go test ./...` pass on the final commit.

## Fixed

### F1 [NIT] The pane-binary invariant's index entry still names strand panes only

Confirmed: `panebin.go` serves both `launchStrandLocked` and `OpenWindow`, and the background file's bullets already said so, but the `PATTERN.md` index line and the background's opening sentence scoped the rule to strand panes.

- `PATTERN.md`: the entry now applies to "Creating a strand pane or a detached `lyx` window in reed".
- `pattern/PATTERN-pane-binary-resolution.md`: the opening sentence now reads "A strand pane or detached `lyx` window reed creates resolves `lyx` to the binary that spawned it."

No test change: the finding is a wording gap, not a coverage gap.
Commit `8529b5c06`.

## Disputed

None.

## Deferred

None.
