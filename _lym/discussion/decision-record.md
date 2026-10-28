# Decision record — hub-pull-verb

## Goal

One operator verb brings a whole hub up to date from origin: Bolt, the prime pair and every local beam pair.
It is safe to run while the orch, the dobbies and runs are live, and it reports per side what moved or why that side stopped.

## Scope

- In: `lym fabric pull --hub`, run from any warp worktree of the hub.
- In: Bolt pulled under the board write lock and the board's push lock, the same pull-first path a board write takes.
- In: the prime pair and each local beam pair pulled through `Fabric.Pull`, each under the parent lock of its code branch, the lock a landing into that branch takes.
- In: a per-side report, and a failure exit when any side stopped.
- In: the fabric CLI and engine docs, `docs/overview.md` only if a module row changes.
- Out: task pairs, which their own runs keep level; pairs of another machine; pushing anything.
- Out: changing what `lym fabric pull` without `--hub` does.

## Decisions

### The verb is a flag on `lym fabric pull`

`lym fabric pull --hub` pulls the hub instead of the acting pair.
Rationale: the operator already reaches for `lym fabric pull`, and the flag keeps one pull verb whose scope the flag names.
The flag adds no new reach: it runs only paths `lym fabric pull` and a board write already run, and the role guard that refuses `pull` from a task session refuses it with the flag too.

### Sides run in a fixed order and each stands alone

Bolt first, then the prime pair, then each local beam in `git worktree list` order.
A side that stops never stops the next side.
Rationale: Bolt carries the board every later reader trusts, and a stop on one side is no reason to leave the others stale.

### Bolt is pulled through a new exported `Bolt.Pull`

`Bolt.Pull` takes the board write lock and then runs the existing pull-first path, which takes the push lock in the board dir.
It returns the same skip and way-forward detail a pull-first write reports.
Rationale: the board write lock is what the orch, the dobby and every board write hold, so a pull under it never interleaves with a write; the push lock keeps a coalesced push from rebasing the board mid-pull.

### Each pair is pulled under the parent lock of its code branch

The CLI's lock seam, the one `lym fabric anchor` uses, is shared: it queues behind a holder with no time limit, prints who holds the lock while it waits, and releases after the pull.
The lock owner's run id names the hub pull.
Rationale: a landing moves the parent branch only under that lock, so a pull under it never moves the branch under a landing in progress.
Bound: the wait ends on Ctrl-C through the command context; the lock is held for one pair's pull only and never across sides.

### A local beam whose code branch has no upstream is skipped as up to date

Rationale: a beam made with no push has nothing to pull from, and reporting it as a stop would fail every hub pull until the beam is pushed.

### The report carries one entry per side

Each entry names the side kind (`board`, `prime` or `beam`), the worktree, the outcome (`moved`, `up_to_date` or `stopped`) and a detail.
A stop's detail is the way forward its own path already names.
Any stop makes the envelope an error envelope that still carries every side's entry, as `lym fabric reconcile` does for its pairs.

### PATTERN-parent-lock departure

PATTERN-parent-lock says a local parent branch moves only in a run's Finalize.
The plain `lym fabric pull` in the prime already moves the prime's branch outside any run;
the hub pull moves it the same way, but under that branch's parent lock.
The commit that adds the pair pull carries a `Pattern-Departure: PATTERN-parent-lock` trailer naming this.

## Constraints

- Git runs only in `internal/fabricengine` (PATTERN-fabric-git); the hub walk lives there and the CLI passes the lock seam.
- The beam list reads origin records through fabricengine, never the acting worktree (PATTERN-pair-parent is untouched: no parent is picked).
- Tests are hermetic hubforge fixtures, tier tagged by PATTERN-test-speed.
- A refusal or stop names its way forward (PATTERN-refusal-way-forward).

## Auto-mode assumptions

- "Local beam" means a pair present on this machine whose origin record carries the beam mark, whatever machine owns it.
- The prime's dirty warp stops the prime side with the existing ErrWarpDirty text; the hub pull adds no stash.

## Open risks

- A long landing holds the parent lock for minutes, and the hub pull waits for it; the wait message names the holder.

## Acceptance criteria

- Bolt, the prime and a beam that origin moved are all moved by one hub pull, each reported `moved`.
- A hub already level reports every side `up_to_date` and exits 0.
- A side that stops, such as a dirty prime warp, is reported `stopped` with its way forward, the other sides still move, and the verb exits non-zero.
- A beam with no upstream is reported `up_to_date`.
- The pair pull waits while another owner holds the branch's parent lock and runs once it is released.
- Bolt's pull runs under the board write lock.
