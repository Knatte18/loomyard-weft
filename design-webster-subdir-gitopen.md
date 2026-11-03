# lym webster begin-batch fails: go-git cannot open a worktree root that is a subdirectory of the repository

`webster-subdir-gitopen` · Note · bug, webster, git

Imported from [issue #35](https://github.com/Knatte18/loomyard/issues/35).

What happened: `lym webster begin-batch 1` was refused before any fork: `websterengine: head sha in <worktree>/<subdir>: gitrepo: open go-git handle at <worktree>/<subdir>: repository does not exist`. The loom run halted blocked (escalation-to-human) at Webster.

Why abnormal: the task's working directory is a subdirectory of the git worktree (the repository root is higher up), and the go-git open is called on that subdirectory without repository discovery, so HEAD sha lookup fails. Every earlier verb worked from the same directory.

What to do: open the repository with detection of the enclosing `.git` (go-git `PlainOpenWithOptions` with `DetectDotGit`), or resolve the repository root before calling the head-sha lookup in `websterengine`.

Provenance: task `commonlib-fluid-report-selection`; notes loom-halt-4.md, webster-refusal-begin-batch.md.
