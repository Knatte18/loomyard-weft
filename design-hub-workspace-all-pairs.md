# The hub workspace file lists every pair, beams' pairs included, named with their project handle

`hub-workspace-all-pairs` · Note · enhancement, fabric, ide

## Seen (operator, 2026-11-11)

The hub workspace file `<hub>/<repo>.code-workspace` lists the prime, Bolt and every pair whose parent is the prime, beams included,
but no pair under a beam: those are only in the beam's own `<repo>-<beam>.code-workspace`.
The operator rarely opens VS Code, and when they do they want to see all of a repository's code in one window,
so a beam's tasks missing from the hub file sends them to a second window.
A folder is named by its slug alone, such as `adopt-worker-record`, so nothing shows which project it belongs to.

## Idea

- The hub workspace file lists every pair of the hub, the pairs under each beam included, at any depth.
- Each pair's folder name leads with its project's handle, `P1: adopt-worker-record`, when its board entry belongs to a project;
  a pair with no project keeps its slug alone.
  The handle comes from the board at regeneration, so a regeneration after a board change brings it current.
- The beam workspace files stay as they are.

The machine restart launcher (`machine-restart-shortcut`) opens only this hub file, so it relies on it listing every pair.
