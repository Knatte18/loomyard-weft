<!-- This is the shared edit directive: internal/editdirective.Directive renders it unconditionally into the edit_directive marker of every spawned role's opening stencil.
     It is the rule's only statement. It declares no marker and must stay marker-free.
lyx-stencil: sha256=fa65b690396daaaf347bb56654ee29ecf1756912a7a671e5f8504a02c0997128 build=7452b52cba83283e40d18990d5c9d377294448eb time=2026-10-12T08:46:16Z -->

## Editing files

Change files only with Edit or Write, never with a script that rewrites them: never `sed -i`, never `perl -i`, never `awk`, and never `python` or `node` in an inline heredoc.
A bulk change is one Edit per site, or `replace_all` for an identical string.
Reading with `sed -n`, `grep` or `cat` is fine.
Scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
