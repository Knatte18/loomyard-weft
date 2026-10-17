# Fixer-report — darn round 1

## Changes

Both findings were fixed as proposed, in one commit, `0e77c8563` (`boardengine: fix F1, F2 — ...`).
The two fixes share `render.go` and the `render_test.go` fixture row, so they were not split into two commits.

- **F1** (`internal/boardengine/render.go`, `settingValue`): encodes non-string values with a `json.Encoder` that has `SetEscapeHTML(false)`, writing to a `strings.Builder`, and trims the trailing newline.
- **F2** (`internal/boardengine/render.go`, `entryCell`): each setting line now goes through a new `codeSpan` helper.
  It fences the line with a backtick run longer than the longest run in the line, and pads one space on each side when the line starts or ends with a backtick.
- **Tests** (`internal/boardengine/render_test.go`): the `top` fixture's `skip` list became `["a&b", "<c>"]` and gained a `note` setting holding ``use `x` ``.
  The expected row was updated to match.
  These are assertion changes inside the existing README render test, not new tests.
- The gate then flagged the `codeSpan` doc comment as a fixed-column wrap.
  It is now one sentence per line, in a second commit.
- `lyx gate test ./internal/boardengine` passes.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- fix-consistency: not spawned.
  I launched only the three lenses above, in error, and the one fork pass per session was then used.
  Its ground (siblings that depend on the changed text) is covered in part by the affected-set fork's search of the other code-span and marshal sites, which found none affected.
