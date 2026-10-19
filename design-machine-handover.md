# One machine takes over the hub from another: run owners, adopt, origin check, loud pushes

`machine-handover` · Task · bug, enhancement, batten, board, orch, fabric, creel

Depends on: [`ubuntu-2404-support`](design-ubuntu-2404-support.md), [`hub-workspaces`](design-hub-workspaces.md)

## Goal

Two machines of the operator work on the same hub, never at once: one machine takes over what the other was doing, easily and without a tangle.
On 2026-10-19 the NORCE machine (Ubuntu 24.04) took over from Neve with only warp `main` updated: the prime weft and Bolt were stale, both runs were resumed with no ownership check, and when Neve woke from hibernation it drove the same runs on, diverging `implementer-seam`'s weft and Bolt from origin.
The operator puts this high (2026-10-19).

## Design (agreed with the operator 2026-10-19)

1. **Every entry has its own owner machine** (operator decisions 2026-10-19).
   Each task and each beam records the machine that owns it on Bolt (for example `owner: neve`), shown by `lyx board`, `lyx batten status` and `lyx tender status`.
   Beams are not built yet; the prime is the root beam and carries an owner like a beam.
   A machine starts, resumes and lands only what it owns, and shows the rest as owned by that machine, not as stalled or landed.
2. **Takeover is an explicit verb**, working name `lyx adopt`, taking exactly the entries it names: one or more tasks, the prime, or everything with `--all`.
   Adopting a beam (today the prime) takes the beam only; its children keep their owners.
   It brings the repos the entries need up to date from origin (warp, prime weft, Bolt and the named pairs' branches, adding pairs this machine lacks), records this machine as owner without the old owner's help, pushes, and resumes the adopted runs here.
   A machine that wakes, or starts its orch, and finds an entry it drove now owned by another machine pauses that run instead of driving it on.
3. **Landing needs the parent's owner.**
   A task lands into its parent by merging in the parent's local worktree and pushing it, so it lands only on the machine that owns the parent.
   A run whose machine does not own its parent halts before the parent-side merge, naming the parent's owner and the two ways forward: adopt the parent here, or let the owner land the run from its branch on origin.
   A handover in which the parent's owner lands another machine's run on its own is not built here (see the beam section below).
4. **An origin reconcile before anything that matters** (agreed with the operator 2026-10-19).
   A machine never needs to know that it was away: the reconcile is cheap enough to run before every step where a stale view does harm.
   `git ls-remote origin` returns every branch head in one round trip with no download; one call for the warp repo and one for the records repo (prime weft, every task weft and Bolt) cover the hub, run in parallel.
   Bolt alone is not enough: it misses task-branch commits between status writes, commits in wefts that change no status, and pushes made by hand, such as warp `main`.
   The heads are compared with the local tracking refs, and Bolt's owners and statuses with what this machine drives:
   - origin moved and nothing is local only: fast-forward;
   - an entry is now owned by another machine: pause its run here;
   - an entry is `done` or gone from Bolt: it landed elsewhere, so it is never resumed; any commit on it that only this machine holds is kept under an `archive/*` tag and reported, then its local pair and run record are removed locally with no push;
   - the branches diverged: stop and name the branch, never carry on.
   The operator says remote checks cost real time, so it runs rarely, never per row and never in a loop: at orch start, on wake from `lyx hibernate`, when batten resumes a run, and before Merge-In and Finalize, which talk to origin anyway; later dobby runs it at its start and before each action.
   Measured on Neve 2026-10-19: about 0.6 s per call, about 0.7 s for both in parallel.
   It is the one synchronous remote operation the operator allows (`fabric-remote-async`: only the check whether branches exist on origin may be synchronous).
5. **A rejected push is never silent** (from #737, below): it fetches, rebases and retries, and a push that still fails reaches the orch.
6. **Open-Window counts per machine** (from #742, below), so a run adopted on a machine that never opened its window opens one there.

## Beams, not built here

Beams do not exist yet, so the beam part of the design is recorded in note `orch-any-worktree` (WS 006), not built by this task: each beam has its own owner machine, adopting a beam takes the beam only, and a branch lands into a beam only on its owner machine, which may land another machine's finished run from origin through a ready-to-land handover on Bolt.

## From issue #741

Imported from [issue #741](https://github.com/Knatte18/loomyard/issues/741).

Seen on a second Linux machine of the operator, in a fresh orch session of the loomyard hub.
The tasks hub-workspaces and implementer-seam were started on another PC.

On this machine:
- `lyx board list` showed them `running · Plan-Write` and `running · Merge-In`.
- `lyx batten status <slug>` showed `state: running` at `Run-Shed`, `error` empty, nothing since 2026-10-18 17:42 UTC.
- Neither pair worktree existed locally, and `lyx tender status` listed no session for either.

Nothing in the board, batten status or tender status says which machine owns a run.
The orch role reads a missing pair as a landed run, which is wrong here, because the board does not show either run as done.

`lyx fabric add <slug>` adopted both pairs from origin, and `lyx batten run <slug> --window` resumed both runs here without any check.
The driver lock is machine-local, so nothing stopped this machine from driving the same branches while the other PC may still have been driving them.

Workaround: the operator confirmed by hand that the other PC was not running them.

Expected:
- A run records the machine that drives it, and `lyx batten status`, the board and `lyx tender status` show it.
- A run owned by another machine is reported as such, not as a stalled or landed run.
- Resuming a run another machine owns is refused unless the operator hands it over explicitly.

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

## From issue #742

Imported from [issue #742](https://github.com/Knatte18/loomyard/issues/742).

Seen when two batten runs started on another PC were adopted with `lyx fabric add <slug>` and resumed with `lyx batten run <slug> --window` on this machine.

Batten resumed both at `Run-Shed`.
`Open-Window` was already recorded done from the first machine, so no terminal window opened on this machine (the help says that a run resumed past that row opens no window).
The operator watches each run in its own terminal window, so they had no window for either run here.

Workaround: attach by hand with `cd <pair> && lyx reed attach`.

Expected: Open-Window counts as done per machine, so a run resumed on a machine that never opened its window opens one there.
