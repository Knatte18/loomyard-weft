# Decision record — smokestart-index-lock-flake

## Goal

A `lyx loom start` whose records commit runs while another lyx process inspects the same weft worktree never fails on `index.lock: File exists`, so `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` stops flaking and no unrelated flake halts a run at Merge-In.

## Scope

- In: the environment every git child spawned through `internal/gitexec` gets, its package doc, and a test pinning it.
- Out: the smoke test itself, the weft write lock, and git spawns outside `internal/gitexec`.

## Decisions

### Every gitexec git child runs with `GIT_OPTIONAL_LOCKS=0`

Decision: `internal/gitexec`'s child environment adds `GIT_OPTIONAL_LOCKS=0` to every git child, local and remote.
Rationale: lyx's locked commits serialize on the weft write lock, but its read-only probes such as `git status --porcelain` run without it, and git's `status` takes `index.lock` opportunistically to write back a refreshed stat cache.
A probe from the first start's driver or its origin reconcile can hold that lock at the instant the second start's `git add` runs, which is the reported failure.
Turning off optional locks makes every read-only probe leave the index untouched, so the only `index.lock` holders are lyx's own locked writers.
Bound: git skips only its optional, lock-taking side operations; a read probe no longer writes back a refreshed stat cache, so a later probe may re-hash files a refresh would have cached.

### The smoke test keeps its starts as they are

Decision: the test gains no wait between starts.
Rationale: the race is in production, between any two lyx processes on one weft, and a wait in one test would hide it there while leaving the cause in place.

## Constraints

- `PATTERN-gitexec-checked-call`: git runs through `gitexec`, so its child environment reaches every production git spawn of lyx.
- `PATTERN-comment-line-breaks`: one sentence per Go comment line.
- `PATTERN-test-speed`: the new test row runs in the lowest tier.

## Auto-mode assumptions

- The failing process was a read-only git probe rather than a writer skipping the weft write lock, since every records commit site takes that lock.

## Open risks

- A git spawn outside `gitexec`, such as a test helper's raw `exec.Command`, keeps git's default optional locks.

## Acceptance criteria

- A git child spawned through `gitexec` sees `GIT_OPTIONAL_LOCKS=0` in its environment.
- The existing `gitexec` tests keep passing, including the one that drops `GIT_DIR` and `GIT_WORK_TREE`.
- The `internal/gitexec` package doc states the new environment entry.
