<!-- This is the shared edit directive: internal/editdirective.Directive renders it unconditionally into the edit_directive marker of every spawned role's opening stencil.
     It is the rule's only statement. It declares no marker and must stay marker-free.
lyx-stencil: sha256=eda629ce5b916a9cce4e9ab11a9dcd9b8152f653347f5ecd5b49db130d1920e3 build=6bfaaa1448adcd5fda4ab769115846291406977a time=2026-10-16T12:19:49Z -->

## Editing files

Change files only with Edit or Write, never with a script that rewrites them: never `sed -i`, never `perl -i`, never `awk`, and never `python` or `node` in an inline heredoc.
A bulk change is one Edit per site, or `replace_all` for an identical string.
The one exception is a Webster implementer working a `Rename` card's symbol pair: it runs the rename tool its prompt names.
That covers only commands the prompt's `rename_tools` list names, only on a `Rename` card's symbol pairs, and only in a Webster fork or recovery strand, and the shell guard's rows still apply to the tool.
Reading with `sed -n`, `grep` or `cat` is fine.
A Go file's formatting is fixed with `lyx loom fmt <file>`.
Scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
