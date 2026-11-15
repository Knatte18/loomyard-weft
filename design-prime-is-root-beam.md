# The prime is the root beam: parent_beam names it, never empty

`prime-is-root-beam` · Note · enhancement, board, dobby, orch

## Seen (operator, 2026-11-15)

`lym board list` and `lym board get` give `parent_beam` empty for an entry whose nearest beam is the prime, and `lym board list --text` shows no `P0`,
while the board view and window titles show the prime as `P0`.
The prime is a beam, the root one; an empty value reads as "no beam".

## Idea

- `parent_beam` names the root beam, as `P0` or its slug, for every entry under the prime, never empty.
- `lym board list --text` shows `P0` with the other projects, so every view of the board shows the same tree.
- Every reader of an empty `parent_beam` (orch role, dobby, board verbs) moves to the named root in the same change.