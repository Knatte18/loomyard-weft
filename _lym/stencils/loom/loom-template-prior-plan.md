<!-- This is the loom Plan producer's prior-plan block.
     It is read at call time by the Plan-Write rotator (via loomengine) and appended to the end of the Plan prompt on a respawn, rather than rendered through a marker in loom-template-plan.md.
     Both markers below are top-level {{.X}} substitutions and both are required: archive_dir is the archive directory's absolute path, and moved_files is the list of file names moved into it.
     There are no {{if}}/{{range}} conditionals anywhere in this file.
lym-stencil: sha256=4ef6487bcc9720e0829d40a4aba19e0df2901f7c07d6d0178807111e40ce2619 build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->

## Prior plan

An earlier session of this run wrote a plan.
That plan was moved out of the plan directory into this archive directory:

{{.archive_dir}}

The files moved there are:

{{.moved_files}}

Before you write anything in the plan directory:

- Read the archived plan before writing anything.
- Check it against the decision record and the current tree.
- Carry forward every card that still holds by copying it into the plan directory rather than re-deriving it.
- Re-verify every glyph you carry forward with `lym quarry`.
- Revise only what is wrong, missing or stale.
- Never modify the archive directory.
- Never copy `00-overview.md` from the archive; write it fresh, last, with `approved: false`.
