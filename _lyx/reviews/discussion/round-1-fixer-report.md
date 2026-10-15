# Round 1 fixer report — discussion

All changes are in `_lyx/discussion/decision-record.md`.
Each premise was checked against `internal/orchcli/start.go`: the state loads before the launch and is saved through `ResetForFreshLaunch` after it, `adoptStrand` rebinds the state onto a different live strand without launching, and the watcher-stopping branch overwrites `fields["hint"]`.

## Changes

- **F1** (fixed as proposed): D1's hint now names `lyx orch stop` then `lyx orch start --adopt <id>` with the previously recorded id inline, read before the launch's save, and D1 states that the hint is the one place that id survives D4's write.
  The Acceptance criterion and Scope's In bullet now name the previous id's `--adopt` command instead of `--continue`.
- **F2** (fixed as proposed): the Out bullet on reed resume now limits recovery to the window before a watcher ticks on the reborn strand.
  A new Open risks bullet says the watcher records the replayed pre-clear id, the post-clear id is then recoverable only from the provider's session list, and `lyx orch status` shows the pre-clear id.
- **F3** (fixed as proposed): D4 now says a live-strand rebind without a launch clears the id until the watcher's first reading fills it.
  D3's Bound names a bound live orchestrator strand among the sessions `--continue` may resume.
  A new Acceptance criterion pins the rebind clearing the id and `--continue` refusing when the strand has no shuttle run.
- **F4** (fixed as proposed): an Open risks bullet records the dependency on Claude Code naming the parent session id in subagent and fork hook lines.
- **F5** (fixed as proposed): D1 says the envelope hint carries both texts, the watcher-stopping one first, and the hint Acceptance criterion covers a launch that also reports `watcher_stopping`.
- **F6** (fixed as proposed): the Constraints docs bullet is deleted.

## Disputed

None.

## Deferred

None.
