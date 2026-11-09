# An interrupted adopt leaves no account of what it did and what is left

`adopt-resume-after-break` · Note · enhancement, batten, fabric

## Seen

On 2026-11-09 an adopt died in its deploy step: the owner was not written, the lym build was cut off and dobby was gone.
No command said which steps had run, and the operator had to run adopt again from a shell.

## Idea

Adopt records its progress per step, `lym beam status` or a status verb shows an unfinished adopt, and a second adopt resumes from the first step not done.