# Round 3 fixer report — discussion

Every premise was checked against the code (`internal/logger/logger.go` Level policy, `strandWindowTarget` in `internal/reedengine/windowtarget.go`, `recordedPaneIDs` in `selvagepane.go`, `warnMismatchedClientsLocked` in `attach.go`) and holds.
All findings fixed as proposed in `_lyx/discussion/decision-record.md`.

## Changes

- **F1** (BLOCKING): D10's failure logging now follows the episode shape: Warn once per failure episode, Debug while it persists, Info when a later check succeeds, Debug only on psmux; the session option records an open episode and a successful check clears it.
  D2's psmux Debug is unchanged.
- **F2** (MEDIUM): D10 acts only on a window holding the recorded Selvage pane, alive or dead, and renames and moves nothing, logging at Debug, when the resolver falls back.
  D3 excludes an overview strand's pane from the resolver's window-0 strand fallback, as an own-window strand's is.
  New acceptance criterion: with no recorded Selvage pane, an attach or window open renames and moves no window.
  For consistency with this, D2's "no shell window resolves" parenthetical now reads "no recorded Selvage pane, alive or dead, in the session's listing", matching the resolver's own presence test in place of "recorded or alive".
- **F3** (MEDIUM): D5's Attach bullet appends no selection when another client is already attached, decided by the pre-flight's existing client listing; the Attach acceptance criterion gains the matching clause.
- **F4** (LOW): D1 states that the lowest index is best-effort on psmux, where D2's fallback applies.
- **F5** (LOW): the migration acceptance criterion now holds when Selvage is the only pane in the strands' window and when no client size is known.

`support-log.md` needed no change.

## Disputed

None.

## Deferred

None.
