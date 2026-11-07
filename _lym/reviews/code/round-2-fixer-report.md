# Fixer-report, code review round 2

## Changes

- **F1 (LOW, consistency), fixed as proposed.**
  In `_lym/landing/summary.md`, the "Failed resolutions are dropped, not kept" decision is now "Refused resolutions are dropped, not kept".
  It says that `hubregistry.ErrUnresolved` and `hubregistry.ErrNoRepository` drop the entry,
  and that any other resolve error, such as an unreadable per-user state directory or a corrupt record, ends the rewrite with that error, writes no marker and retries on the next walk.
  The Tests section's `creelcli` `hubs` bullet now names a listed shortname, a hub with no origin and a hub with a forged repository name, each untrusted with its reason.
  A new bullet records the `read --envelope` assertions for `sending_repository` present and absent.
- No commit: the file lives under `_lym`, where the round's rules forbid git commands.
  The change is in the working tree only.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
