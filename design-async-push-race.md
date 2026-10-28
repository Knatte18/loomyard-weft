# A racing async push reports no divergence that is already gone

`async-push-race` · Task · bug, fabric · running · Describe

## A racing async push reports a divergence that is already gone (was note `async-push-race-false-divergence`)

Imported from [issue #879](https://github.com/Knatte18/loomyard/issues/879).

#### Seen (beam-start-and-ownership, 2026-10-26)

Within minutes, `ly:FABRIC` sent the orch two `not_fast_forward` notices for one run on one machine: first for its code branch, then for its records branch.
Each way forward named a `git merge` or `git reset --keep`, and the records notice said "another machine wrote the same paths" with `paths: none`.
A fetch right after each notice showed nothing on origin the local branch lacked: a concurrent push had already landed, and the next push was a plain fast-forward.
Two asynchronous pushes of the same branch raced, and the loser reported a divergence that no longer existed.

#### Idea

- A rejected push fetches origin again before it reports: when origin is now an ancestor of the local tip, it retries the push instead of sending a notice.
- The notice names another machine only when origin's commits carry another machine's name, and never with `paths: none`.
