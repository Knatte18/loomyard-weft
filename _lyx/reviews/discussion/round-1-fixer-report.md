# Fixer report — discussion, round 1

## Changes

- F1 (MEDIUM), fixed as proposed: D5's third bound bullet no longer claims an operator watches the window.
  It now says the session takes no turn until text reaches its pane, by the operator or through `lyx shuttle send <guid>`, and that a caller driving it through `send` holds an unguarded bypass session bounded only by the explicit flag.
  Premise checked: `internal/shuttlecli/send.go` accepts "a shuttle run's or a live reed strand's" guid, and `Runner.Open` adds the strand with `render.AnchorOwnWindow`.
  The new bullet's sentence is split at its semicolon onto continuation lines, per semantic line breaks.
- F2 (LOW), fixed as proposed: Scope's docs bullet now names the `buildOpenCmd` and `Claude.OpenLaunch` doc comments, and the `buildOpenCmd` bullet under Notes for the plan writer is removed.
  Premise checked: `internal/shuttleengine/claudeengine/command.go` says the open line passes "no permission flag", and `OpenLaunch`'s comment lists what it does "before anything else".
- F3 (NIT), fixed as proposed: AC4 now reads "before a strand is added or the marker directory is created"; D3's ordering sentence is unchanged.

## Disputed

None.

## Deferred

None.
