# Drop the ban on scripted edits: the sed -i and redirect-write guard rows and the edit directive

`scripted-edits-allowed` · Task · enhancement, shuttle, stencil · running · Code-Burler-Fix

## Seen (operator, 2026-10-26)

Every agent session lyx spawns, the orch included, still refuses scripted edits, although the operator has said more than once that the ban is no longer needed.
The refusals come from shell guard rows in `internal/shellguard`, installed on every run with no config key (see `internal/shuttleengine/claudeengine/doc.go`):
- the in-place edit row, which denies `sed -i` and `perl -i`;
- the redirect-write row, which denies a shell redirect into a project file outside `.scratch/`.
The edit directive in the agent prompts says the same: change files only with Edit or Write, never with a script.

## Idea

- Remove the in-place edit row and the redirect-write row, and the edit directive's ban on scripted edits, so an agent may edit with `sed`, a heredoc or a script where that is simpler.
- This is the way the rename in `rename-binary-lyr` is meant to be done: mechanically, by script.
