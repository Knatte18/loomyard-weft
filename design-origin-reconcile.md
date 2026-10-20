# Origin reconcile before anything that matters, and pushes that never fail silently

`origin-reconcile` · Task · bug, enhancement, fabric, batten, orch, creel, shed

## Goal

A machine never acts on a stale view of origin, and a push that origin rejects is never silent.
On 2026-10-19 the NORCE machine (Ubuntu 24.04) took over from Neve with only warp `main` updated: the prime weft and Bolt were stale, and when Neve woke from hibernation it drove the same runs on, diverging `implementer-seam`'s weft and Bolt from origin while its rejected pushes went unreported.
This is the first half of the operator's two-machine work (high, 2026-10-19); task `machine-handover` builds owners and takeover on top of it.

## Design (agreed with the operator 2026-10-19)

1. **An origin reconcile before anything that matters.**
   A machine never needs to know that it was away: the reconcile is cheap enough to run before every step where a stale view does harm.
   `git ls-remote origin` returns every branch head in one round trip with no download; one call for the warp repo and one for the records repo (prime weft, every task weft and Bolt) cover the hub, run in parallel.
   Bolt alone is not enough: it misses task-branch commits between status writes, commits in wefts that change no status, and pushes made by hand, such as warp `main`.
   The heads are compared with the local tracking refs, and Bolt's statuses with what this machine drives:
   - origin moved and nothing is local only: fast-forward;
   - an entry is `done` or gone from Bolt: it landed elsewhere, so it is never resumed; any commit on it that only this machine holds is kept under an `archive/*` tag and reported, then its local pair and run record are removed locally with no push;
   - the branches diverged: stop and name the branch, never carry on.
   `machine-handover` later adds one more outcome, an entry now owned by another machine, which pauses its run here; the reconcile is built so that outcome slots in.
   The operator says remote checks cost real time, so it runs rarely, never per row and never in a loop: at orch start, on wake from `lyx hibernate`, when batten resumes a run, and before Merge-In and Finalize, which talk to origin anyway; later dobby runs it at its start and before each action.
   Measured on Neve 2026-10-19: about 0.6 s per call, about 0.7 s for both in parallel.
   It is the one synchronous remote operation the operator allows (`fabric-remote-async`: only the check whether branches exist on origin may be synchronous).
2. **A rejected push is never silent** (from #737, below): it fetches, rebases and retries, and a push that still fails reaches the orch.

## From issue #737

Imported from [issue #737](https://github.com/Knatte18/loomyard/issues/737).

On the operator's work PC the GitHub account in use had read-only access to the records repo.
Every detached weft and board push failed with a 403 from 2026-10-16 to 2026-10-19, and lyx said nothing outside a WARN in the detached child's trace log under `_board/.lyx/logs/`.
The prime's `main-weft` fell 552 commits behind origin unnoticed.

- **A rejected detached push is silent.**
  `lyx cleanup --apply` reported `push_error: ""` because the push was spawned, not because it landed.
  A push that fails for a non-transient reason (auth, permission, non-fast-forward after the recovery rebase) should reach the operator: a notice to the orch through creel, or a status the next `lyx cleanup`, `lyx fabric status` or `lyx board` call prints.
- **`lyx cleanup` judges runs from a stale weft.**
  It never fetches first, so it listed three runs as paused that origin had deleted days earlier, and its removals diverged the weft from origin.
  It should pull the weft first, or refuse while the weft is behind its upstream.
- **`lyx fabric pull` leaves a diverged weft alone and still reports `ok: true`.**
  `git pull --ff-only` fails, the pull logs a WARN and returns `weft_pulled: false` inside an ok result.
  The doc names `git -C <weft> reset --hard origin/<branch>` as the manual recovery.
  Instead, pull should rebase the local weft commits onto the upstream and drop those that become empty, stopping with a named way forward only on a real conflict, such as a `status.json` both sides changed.

Seen again on Neve the same day: after waking from hibernation, its two Bolt status commits were rejected because the NORCE machine had pushed eight newer ones, and nothing reported it; the orch reset Bolt to origin by hand and rewrote the status.


## From note `preflight-weft-push-sync` (folded 2026-10-20, same push path)

### Seen (orch, 2026-10-17)

The darn run `creel-c13-stacked-rewrite` spent 11.1 s in its Preflight row (trace `acc61c4a8e5651cb` in the pair's `.lyx/logs/`), almost all of it in a synchronous push of the weft branch to GitHub:

- 13:43:50.98: the row commits the weft (`fabric: mutation kind=commit_created`); 0.2 s earlier the same step spawned a detached board sync, which pushes to the same records repository.
- 13:43:58.91: the first push of `creel-c13-stacked-rewrite-weft` is refused after 8 s: `! [remote rejected] ... (failed)`.
- 13:44:01.89: the retry succeeds after 3 s, and the row is Done.

The operator saw about 22 s for the row; the trace accounts for 11 s of it.

### Idea

- Nothing in the run reads the remote weft branch until Publish, so Preflight could push it detached, as the board sync already is, and leave the row at local work only.
- The 8 s refusal looks like two pushes to the same records repository at once (the board sync and the weft push); if so, serialising them, or the detached push, would remove the retry.

### A pair begun on another machine

Operator question 2026-10-17: what if the branches already exist because the task was begun on another machine?
That case is settled before Preflight, at `fabricengine.Add` (Worktree-Create), per `internal/fabricengine/doc.go`: it probes both origins read-only, adopts a live pair (fast-forwards a local weft branch behind origin, or makes origin's weft and warp branches local tracking branches), refuses a true divergence with `ErrRemoteLeftover`, and never force-pushes or deletes a live pair's branches.
`Add` itself already pushes nothing and leaves the push to the pair's detached push (`SpawnPairPush`).
So a detached Preflight push changes nothing for that case; it only gives up seeing at once a push the remote refuses, which the next detached push logs as a Warn.
The bare `(failed)` refusal is a known server-side race: every plain fabric push retries it with backoff.

### Open

- Whether the remote refusal is a GitHub race between the two pushes or something else; check other runs' Preflight traces for the same warning.
