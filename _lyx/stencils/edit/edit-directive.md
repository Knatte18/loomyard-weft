<!-- This is the shared edit directive: internal/editdirective.Directive renders it unconditionally into the edit_directive marker of every spawned role's opening stencil.
     It is the rule's only statement. It declares no marker and must stay marker-free.
lyx-stencil: sha256=99488f0cc10d94ccfcd9be7788b4c73d6f33440055a9b3c279795598f5950bd5 build=e17b29219d8d6f7980439bea0ae83ea4f9d9458e time=2026-10-13T08:49:51Z -->

## Editing files

Change files only with Edit or Write, never with a script that rewrites them: never `sed -i`, never `perl -i`, never `awk`, and never `python` or `node` in an inline heredoc.
A bulk change is one Edit per site, or `replace_all` for an identical string.
Reading with `sed -n`, `grep` or `cat` is fine.
A Go file's formatting is fixed with `lyx loom fmt <file>`.
Scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
