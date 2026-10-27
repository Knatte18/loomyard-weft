<!-- This is the shared edit directive: internal/editdirective.Directive renders it unconditionally into the edit_directive marker of every spawned role's opening stencil.
     It is the rule's only statement. It declares no marker and must stay marker-free.
lym-stencil: sha256=d9d8f003b288432d670437ba64999ea885be04bdb671af5452d6874a7705d2e2 build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->

## Editing files

Change files with Edit or Write, or by script with `sed -i`, `perl -i`, `awk` or a shell heredoc or redirect wherever a script is simpler, such as a mechanical change across many sites.
A Go file's formatting is fixed with `lym loom fmt <file>`.
Scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
