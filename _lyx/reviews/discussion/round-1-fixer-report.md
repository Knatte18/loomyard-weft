# Round 1 fixer-report

## Changes

- F1 (BLOCKING), fixed as proposed: the Constraints bullet on `internal/selfexec`'s import set is replaced by two bullets: `PATTERN-leaf-packages` gains an `internal/selfexec` section and the package gets a `leaf_enforcement_test.go` through `scankit.AssertImportAllowlist`; `internal/statuspane/doc.go`'s import list gains `internal/selfexec`.
- F2 (MEDIUM), fixed as proposed: D1 selects the replacement's window only when the replace ends the current window; a shared-window replace switches nothing.
  The second acceptance criterion gains the shell-window case.
- F3 (MEDIUM), fixed as proposed: D2's hidden-window command targets the highest-index window, read at open time under the op lock.
- F4 (MEDIUM), fixed as proposed: D2's no-shell fallback opens own windows after the highest-index non-hidden window and hidden windows as in the bullet above; D6's step sentence names the psmux exception.
- F5 (MEDIUM), fixed as proposed: D10 states the check runs under the op lock, logs a failed rename or move at Warn, never fails the open or attach, and retries at the next one; D10 gains a Bound line.
  Consequent edit: the `PATTERN-log-levels` Constraint now lists D10 among the Decisions with Debug and Warn levels.
- F6 (LOW), fixed as proposed: Open risks gains the stale-sidecar bullet.
- F7 (LOW), fixed as proposed: Scope Out's first bullet is cut to the exclusion; the lock-gap sentence moves to support-log Interview item 1, which already carries the commit hash.
- F8 (NIT), fixed as proposed: the migration criterion says the strands' window moves after the last non-hidden window.

## Disputed

None.

## Deferred

None.
