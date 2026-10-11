<!-- This is the shared edit directive: internal/editdirective.Directive renders it unconditionally into the edit_directive marker of every spawned role's opening stencil.
     It is the rule's only statement. It declares no marker and must stay marker-free.
lyx-stencil: sha256=9c22802f8766f60596fa2de176acf13a6f6e48de1c8f564cfa4e4cf67ab7f4e3 build=92b979c402253a9624a4bf569c424dc9d8beb79a time=2026-10-11T08:53:00Z -->

## Editing files

Change files only with Edit or Write, never with a script that rewrites them: never `sed`, `awk` or `perl`, and never `python` or `node` in an inline heredoc.
A bulk change is one Edit per site, or `replace_all` for an identical string.
