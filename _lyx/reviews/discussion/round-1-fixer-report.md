# Fixer report — discussion round 1

## Changes

All in `_lyx/discussion/decision-record.md`.

- F1: D3's Bound gains the sentence that a free lock means the daemon has left its loop and released the lock, and that the exiting process may still run D3a's re-check listing and at most one successor spawn when `down` returns.
  D3's Rationale now reads "a free lock is the daemon's own proof it has left its loop".
  The first Acceptance criterion is reworded to the lock being free and the exiting daemon ending after its post-release re-check without spawning a successor.
- F2: D5 gains a Bound paragraph: only `LYX_BIN`'s parent directory is prepended, the same directory the pane prelude already puts first, so the Bash tool shadows exactly what the pane shell shadows; nothing is added when `LYX_BIN` is unset or `cygpath` is absent on Windows.
- F3: D8's first bullet is split.
  The reed suite keeps the pane-shell wording; a new creel bullet says its strands all start fresh shells, so its `command -v lyx` bullet says the check does not hold there, pointing at the Scope exclusion and `SANDBOX-SHUTTLE-SUITE.md`, with the `LYX_BIN`-is-set bullet unchanged.
  D8's Rationale, which the finding's summary named as false for creel, is narrowed to the reed and shuttle checks and states that creel names the gap instead of checking it.
- F4: the Goal's #680 bullet now reads "in the Bash tool of every Claude agent the Claude shuttle engine spawns".
- F5: two Acceptance criteria added: a `down` whose teardown does not confirm gone still writes the stop, returns its error and waits for nothing; a daemon leaving its loop on a cancelled context spawns no successor whatever its post-release listing.

## Disputed

None.

## Deferred

None.
