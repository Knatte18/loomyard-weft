# Let a beam's orch set profile and run_settings on its beam's entries

A beam's orch could file tasks under its beam but not set their `profile` or `run_settings`.
`lym board upsert` refused the write because the orch's strand name carries a slug, which the guard read as a task session.
The orch had to create the task bare and ask the operator or the hub orch to set the profile.

Closes #12

## What changes

`prepareRunSettings` in `internal/boardcli/runsettings.go` now hands the caller check to the new `refuseRunSettingsCaller`.
A session with no slug in its strand name, the operator or the hub orch, still passes.
A beam's orch, a strand carrying the beam's slug and the `orch` role (the numbered `orch-2` included), passes when the entry belongs to that beam.
Every other slugged session is refused as before, and the refusal now names the orch of the entry's beam as a way forward.

## Design decisions

- **Both ends of the move are checked.** The entry must be in the orch's beam as stored and as the payload leaves its project link.
  An orch cannot pull an outside entry into its beam to set its profile, nor move a member out and change it on the way.
  A new entry is checked on the project link it carries, so a new entry under no project is refused.
- **The beam is resolved through the project link.** The new `Board.BeamUnder` in `internal/boardengine/parentbeam.go` answers the beam for a project link without persisting anything.
  It returns the project itself when that is an open beam project, and otherwise what `ParentBeamOf` answers; an empty, unknown or closed link reads as the prime.
  A group project nested under the beam therefore counts as inside it.
- **The beam entry itself is not "under" its beam.** A beam's orch cannot change the profile of the beam project entry.
- **An unparseable strand name passes.** `refuseRunSettingsCaller` treats it like a slug-less session, as the old `agentname.TaskSlug` check did.
- The guard still reads outside the write lock and stops a mistaken write, not a determined one.

The package docs in `internal/boardcli/doc.go` and `internal/boardengine/doc.go` describe the new rule.
Tests cover `BeamUnder` and the accepted and refused callers in `stepRunSettings`.

## To check by hand

- The issue also asks that the orch role tell every orch to name a profile before a batten run starts.
  This diff changes only the board guard; check whether the role text already says so.
- Nothing was run against a live beam orch session; the behavior is covered by the unit and CLI tests only.
