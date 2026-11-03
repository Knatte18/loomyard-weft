# Fixer-report, code round 1

## Changes

- **F1 (MEDIUM): fixed differently.**
  Commit `d44d6a60`.
  Added step `RemoveCommandParses` (`stepRemoveCommandParses` in `internal/boardcli/verbs_test.go`, registered in `internal/boardcli/cli_test.go`).
  For three slugs it takes `boardengine.RemoveCommand(slug)`, strips the prefix and the closing quote, runs the real `remove` verb with the payload, and asserts the entry is gone.
  Proposed change not made: a table test in `internal/boardengine` through an `export_test.go` seam.
  The parser (`lookupPayload`, `resolveLookup`) lives in `internal/boardcli/cli.go`, so a seam in `boardengine` could not reach it; driving the real verb from the `boardcli` tests covers the same parse end to end.
- **F2 (LOW): fixed as proposed, by its stated alternative.**
  Commit `cc42f0b9`.
  `fronttask.Remove` returns only nil or a `*fronttask.RemoveError` (`internal/fronttask/fronttask.go:222-247`), so the fallback branch in `removeFixTask` (`internal/adoptcli/green.go`) was unreachable.
  Deleted it; the error is now asserted to its type, so a violation of that contract fails loudly.

Gate: `lym gate test ./internal/boardcli ./internal/adoptcli` passed (tier0, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
