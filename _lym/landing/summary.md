# Document that .scratch is a link and tell agents to use absolute paths

Agents were told to put scratch files under `.scratch/` in the working directory, but `.scratch` in a task worktree is a link into the weft worktree.
A project built under it resolves relative project references from the link's target, not from the working directory, so the build fails.
Nothing in the agent instructions said so, and an agent worked around it by copying the project into a scratchpad nested in another git repository.

The edit directive (`contracts/stencils/edit/edit-directive.md`) now says two things:

- `.scratch` is a link to a directory kept for the task alone, so a file under it stays in the agent's own task.
- A build under `.scratch/` resolves a relative reference from the link's target, so a project built there names the working tree's files by absolute path.

The `internal/editdirective` and `internal/fabricengine` package docs record the same fact.
The fabric doc now lists it as a cost of the link, beside the existing reason for choosing a link.

Closes #120

## Design decisions

- The issue offered two fixes: make `.scratch/` a real directory, or document the link.
  This change documents it.
  The fabric doc already explains why the link is a link: a real directory would change its modification time with every file an agent adds or removes,
  and that time feeds the `go test` cache key of every source-scan test that opens the module root.
  Replacing the link would bring that cache churn back.
- The issue's second complaint, that the link puts a task's scratch files in another worktree's tree, is answered by the directive text.
  The target is the task's own scratch directory in the weft worktree, so the "work only in your own worktree" rule is not broken.
- The change is documentation only; no code behaviour changes.

## To check by hand

- Read the added directive lines as an agent would, and confirm that they say what to do, not only what goes wrong.
- No regression check ran for this task, and no tests were added, since only prose and doc comments changed.
