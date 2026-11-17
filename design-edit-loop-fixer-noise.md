# Edit-loop notices fire on a fixer editing one file

`edit-loop-fixer-noise` · Note · bug, shuttle

## Seen (orch, 2026-11-17)

The edit-loop watch landed in `c167e803` (four edits of one file within five minutes sends the parent a notice).
Within its first hour it sent two notices for `gate-test-map-mode`'s Plan-Burler-Fix agent,
which edited `_lym/plan/01-gate-test-map-mode.md` four times within five minutes each time.
That agent was applying the round-2 review's findings, one Edit per site, to the one plan file it owns; it was making progress.
Minutes later `codex-standing-instructions`' Code-Burler-Fix agent drew one for `instructionfiles.go`, fixing several findings in that file.
A fixer applying several findings to one file, or a writer whose whole output is one document,
edits that file many times by design, so the watch reads ordinary work as a loop.

## Idea

Count an edit toward a loop only when it undoes or repeats an earlier one,
for example the same `old_string` edited back and forth, or the file's content returning to an earlier hash,
instead of counting every edit of the file.
Failing that, exempt the roles whose output is one document, or give them a higher count.
