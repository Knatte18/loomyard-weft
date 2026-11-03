<!-- This is the Codex engine's copy of the edit directive: internal/editdirective.EngineDirective renders it into the edit_directive marker of an assistant seat that runs on Codex.
     It states the shared edit directive's rule in Codex's own tool wording. It declares no marker and must stay marker-free.
lym-stencil: sha256=c086cf0b80fe0e92011b16fa2ad42ca4794ca88b9df9e26db0df4cae55309611 build=935fb407d46b918a520439401a144eb28d3de274 time=2026-11-03T17:43:20Z -->

## Editing files

Change a file only with `apply_patch`, never by shell.
Read files with shell commands that only read, such as `cat`, `sed -n` or `rg`, and search with `rg` or `grep -rn`.
A shell guard refuses a shell edit: never `sed -i`, never `perl -i`, and never a heredoc, redirect or tee into a file.
Work only in your own worktree: never edit, commit or push in another, and never create one.
