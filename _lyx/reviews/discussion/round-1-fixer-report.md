# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed): added D10 for the default run selection: candidates are the archive-tag slugs, the prime left out, located at `<hub>/<slug>` through every engine; recency is the newest last-modified time the locator returns per session, the main transcript's file time for Claude, so ranking reads no transcript; a candidate with no located session is skipped while a locator error still fails; the no-runs error names the hub and the engines asked.
  D1 and the Scope In locator bullet now say the locator returns each session's last-modified time.
  D7 limits the "no sessions" error to named runs and points at D10.
  Scope Out permits the reworded no-runs error.
  Added an acceptance criterion for the default selection over a fake read set.
- F2 (fixed as proposed): D2 defines the session's repeat count as collapsed streamed records plus, in a fork, the skipped replayed parent messages; the Claude reader scenario asserts the skipped replay counts there. D9 unchanged.
- F3 (fixed as proposed, first option): D7 drops the `/clear` and teardown examples, names the provider's transcript expiry as the cause, and fails the count on a vanished session like an unreadable one, so a successful count drops nothing. The matching acceptance criterion now asserts the failure.
- F4 (fixed as proposed, first option): D6 states the set's capability values are per-capability adapters in `shuttleengines` holding the engine in an unexported field, so no type assertion recovers an `Engine`; added an acceptance criterion that the values do not assert to `shuttleengine.Engine`.
- F5 (fixed as proposed): Scope Out names the `-no-cutoff` header line and help text rewording (every located session of each slug counts) as a permitted wording change.
- Support log: added the round's rejected alternatives and Q9–Q11 to the question ledger.

## Disputed

None.

## Deferred

None.
