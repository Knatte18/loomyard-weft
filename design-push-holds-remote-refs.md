# A weft push holds the remote-refs lock over its whole transfer, so board claims queue behind it

`push-holds-remote-refs` · Task · bug, fabric · running · Code-Burler-Fix

## Seen (worktree-create-serial, 2026-11-04)

`worktree-create-serial` took the prime lock off the network part of Worktree-Create.
One more queue stands behind it: the weft repository's remote-refs lock.

- The Bolt checkout is a worktree of the weft repository, so the Bolt and every pair's weft side share one `lym-remote-refs.lock` in its common git dir.
- `gitrepo`'s pushes hold that lock for each whole attempt, `git send-pack` included.
- On 2026-11-04 a detached push of `dobby-zombie-children`'s weft held it for 60–80 s per attempt, while GitHub answered `Internal Server Error` and the push retried for minutes.
- Every board claim (Bolt pull and push), every Bolt write and every other weft push or fetch waited behind it, on every create.

## Idea

- A push runs its transfer outside the remote-refs lock and writes the tracking ref under it afterwards.
  One way is a push to the remote's push URL, which writes no tracking ref, followed by a compare-and-swap `update-ref` and the upstream setting under the lock.
- PATTERN-remote-refs-lock says a judge of origin holds the lock to its verdict, so check every judge before a push may land outside it.

## Open-Window

On 2026-11-04 Open-Window took 69 s for `dobby-zombie-children` (12:16:45–12:17:54).
The step left no trace at Info, so the cause is not known.
The row runs `lym reed window` and the desktop window command, neither of which goes to origin.
The time overlaps the first attempt of the same pair's weft push.
Next time, read the step's trace at Debug.
