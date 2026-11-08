# Reed buttons: a beam's session reads as a beam before Beam-Up marks it

A beam's reed session could show the task button group (HALT, CANCEL, code, terminal, agent) instead of the beam group with distill, refresh, merge-in and land.
Its `@lym_button_dir` also pointed at the prime instead of the beam's own worktree.

## Cause

The button context in `internal/hubgeom/buttons.go` took the place from the pair's origin record alone: beam when the record carries the beam mark, task otherwise.
Batten's Worktree-Create makes a beam's pair without that mark, and the pawl recipe's Beam-Up row writes it later through `fabricengine.MarkBeam`.
A reed session booted before Beam-Up rendered as a task.
A task's run directory is its parent beam's anchor, and the beam's parent is the prime.
The row is re-rendered only at boot and attach, so a session kept attached since boot held the task group for good.

## Change

`pairIsBeam` decides a pair's place.
A pair reads as a beam when its origin record carries the mark, or when the local board's entry for its slug is a project with the beam flag.
`lym beam create` sets that flag before it starts the pawl run, so the session renders the beam group from its first boot, whether or not Beam-Up has run.
Any other entry, or no entry, reads as a task as before.
A board read error is returned, as an origin read error already was.
reed then logs it and renders no buttons.

The `hubgeom` package doc states the rule.
`TestReedGeometry_ButtonContext` gains rows for these cases:

- a board beam flag on an unmarked record;
- a project without the flag;
- a board that does not parse.

## Design decisions

- The board flag is read in the button context instead of marking the pair at creation or re-rendering the session after Beam-Up.
  Batten's Worktree-Create deliberately leaves the pair unmarked until Beam-Up, and reed exposes no re-render a pawl row could call.
  The board flag already exists before the session boots, so reading it closes the gap with a single local read at boot and attach.
- An existing session that rendered the task group picks up the beam group at its next attach.
