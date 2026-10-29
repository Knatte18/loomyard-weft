<!-- This is the shared edit directive: internal/editdirective.Directive renders it unconditionally into the edit_directive marker of every spawned role's opening stencil.
     It is its rules' only statement. It declares no marker and must stay marker-free.
lym-stencil: sha256=8b49e078fba507c96d6c2eb549ca44349003b6113d76831ab4c7fb23d2b2c21a build=c15d6390d28286d8e0b70fcac8ec021f9ef46d59 time=2026-10-29T14:15:15Z -->

## Editing files

Change files with Edit or Write, or by script with `sed -i`, `perl -i`, `awk` or a shell heredoc or redirect wherever a script is simpler, such as a mechanical change across many sites.
A Go file's formatting is fixed with `lym loom fmt <file>`.
Scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
Work only in your own worktree: never edit, commit or push in another, and never create one, unless your parent or the operator says so for that case.
When work belongs in another worktree, say so to your parent instead.
