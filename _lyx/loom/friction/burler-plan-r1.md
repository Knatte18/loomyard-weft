# Fixing plan round 1: a proposed fix the gate refuses

The review's F11 proposed adding `internal/loomengine/config.go` and `internal/loomengine/discussion.go` as file targets beside member glyphs in those same files.
The gate's `redundant-file-target` check refuses that shape, so the proposal broke the rubric's "Proposed fixes" rule.
The reviewer had no way to see this without running `lyx loom validate-plan` on a trial edit.
It cost a validate round trip to find, and the fix had to fall back on the check's own way forward: target the whole file in place of the member.

F10's card cut also renumbered two cards, which meant deleting their old files, and no Edit or Write operation can delete a file.
