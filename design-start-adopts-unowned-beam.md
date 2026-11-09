# lym s in a worktree this machine does not own adopts it

`start-adopts-unowned-beam` · Note · enhancement, shortcut, batten

## Seen

On 2026-11-09 the operator moved to Neve after working on osl-1046 and had to run `lym adopt` by hand; run from the orch session, it killed itself.

## Idea

`lym s` (beam start, then reed attach) on a machine that does not own the worktree adopts it first: the beam in a beam, root in the prime.
It then starts the beam and attaches.
Adopt thus starts from a plain shell, never from the orch.
When a pair holds local commits origin lacks, `lym s` stops with the keep-local and drop-local choices adopt gives today.