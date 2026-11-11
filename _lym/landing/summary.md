# Beam orch refusal points to the parent orch

A beam's orch that tried to write outside its beam was refused with a way forward naming the operator, the hub orch or the beam's orch.
None of those fits a beam's orch, so the operator had to repeat the order in the parent's session.
The boundary stays: a beam's orch writes the board only under its own beam, and no flag lets it write outside.
The refusal and the orch role now send the beam's orch to pass the wish to its parent orch through creel.

Closes #55

## Changes

- `contracts/orchrole/orch-role.md`: a rule in the messaging section.
  Work that belongs outside the beam, such as a project a beam's motif spins out, goes to the beam's parent orch as a wish with `lym creel post <parent orch> --expect-reply "<wish>"`.
  The wish says what to make and why.
  The parent decides placement from what only it sees, may place the work differently, and tells the operator when something collides.
  A wish that belongs higher up is passed on to the next parent the same way.
- `internal/boardcli/runsettings.go`: `refuseBeamOrchOutsideBeam` builds the refusal for a beam's orch writing outside its beam.
  It names `<shortname>:<parent_beam>:orch`, or `<shortname>:orch` when the beam's parent is the prime, and gives the `lym creel post` command.
  The parent is read with `boardengine.Board.ParentBeamOf`.
- Other refused sessions keep the old way forward: the operator, the hub orch or the orch of the entry's beam.
- `internal/boardcli/doc.go`: describes both refusals.
- `internal/boardcli/verbs_test.go`: each refusal case now pins its own way forward, and a nested beam's orch is added, which is told to ask its enclosing beam's orch.

## To check

- The refusal text is built from the beam's `parent_beam`; the unit test covers the prime and one nested level only.
