# Let a beam's orch set profile and run_settings on its beam's entries

A beam's orch, strand `<shortname>:<beam>:orch`, could file a task under its beam with `lym board upsert` but was refused when the payload named a `profile` or `run_settings`.
The board's caller guard refused every session whose strand name carries a slug, and treated the beam orch as a task session.
The orch role tells every orch to name a profile before the batten run starts, so the beam orch had to ask the operator or the hub orch to do it.

## What changes

- `boardcli.prepareRunSettings` now calls `refuseRunSettingsCaller` instead of checking `agentname.TaskSlug` alone.
  A session with no slug may still change either value.
  A session whose role matches `orch` may change them when the entry belongs to the orch's own beam both as stored and after the write.
  This covers a new task filed under the beam or under a grouping project inside it.
  Every other session that carries a slug is refused: another role in the beam, the orch of a different beam, an entry outside the beam, the beam project itself, and an entry the payload moves into or out of the beam.
- The refusal names the session and adds the orch of the entry's beam to its way forward.
- `boardengine.Board.BeamUnder(project)` returns the beam that an entry with a given project link belongs to.
  It returns the project itself when that project is an open beam, and otherwise what `ParentBeamOf` answers for the project.
  The guard needs it because a new entry, or one whose project link the write changes, is not yet stored, so `ParentBeamOf(slug)` cannot answer for it.

## Design decisions

- An entry's beam is read before and after the write, so a beam's orch cannot change the settings of an entry that leaves or enters its beam.
- `BeamUnder` resolves through `ParentBeamOf`, which uses the `hubgeom.ParentOf` seam when one is set, so the guard uses the hub's single parent resolver (PATTERN-pair-parent).
- The guard still reads outside the board's write lock.
  It stops a mistaken write, not a determined one, as before.
- `contracts/orchrole/orch-role.md` already tells every orch to name a profile, so it is unchanged.

## Tests

- `TestCLI/RunSettings` gets a subtest in which a beam's orch sets a profile on a new member, on a task under a grouping project inside the beam and on an existing member.
  A table of refusals checks the cases listed above, and each refusal names the session and the new way forward.
- `TestParentBeam` gets `BeamUnderAnswersFromTheProjectLink`, covering a beam, a grouping project inside a beam, a done beam, a plain project, an unknown slug and an empty project.
