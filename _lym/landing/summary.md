# A docs bench's darn writer adds and removes its own bench pairs

A darn run that benchmarks something on the docs profile often needs pairs of its own to measure in.
Until now the sandboxed-role guard refused every mutating fabric verb from such a session on the real hub,
so each add and each removal had to be handed to the orchestrating session and waited on.
This change lets the darn writer make and remove those pairs itself, within fixed bounds,
and makes the run's Worktree-Teardown remove any it left behind.

## What changes

**The guard (`internal/fabriccli/roleguard.go`).**
The fabric `add` and `remove` verbs now pass for a darn writer strand when all of these hold:

- the one argument is a bench pair slug of the writer's own slug: `<slug>-b<N>`, `N` a positive number with no leading zero (`darnengine.IsBenchPairSlug`);
- the verb runs in the writer's own worktree or its records sibling, in the writer's own hub;
- an `add` carries neither `--parent` nor `--adopt-local`, so the pair forks from the writer's task branch, as the hand-off did;
- the run's identity record in that worktree, `_lym/usage/run.json`, names the darn recipe and the docs profile.

The detached push an `add` spawns for the new pair, and any later push of a bench pair, carries the writer's strand name and the pair's paths.
`injectedPathsAreOwn` now accepts a darn writer's bench pair, so those pushes are no longer refused.
A refused `add` or `remove` from a darn writer appends the bench pair way forward to the refusal text.

**Worktree-Teardown (`internal/pairteardown/leftover.go`, `internal/battencli/wire.go`).**
`pairteardown.RemoveBenchPairs` lists the hub's pairs and tears down each whose slug is a bench pair slug of the run.
Each goes the way an orphan pair goes:
tips origin lacks are archived under owner archive tags, uncommitted files committed onto them,
then the pair is removed with force, its branches left on origin and no claim settled.
Unlike the orphan route, a live process of the pair is ended rather than refused, since the run's teardown is the pair's last chance to go.
A pair whose slug a `boardengine` entry or a batten run directory names belongs to another run and is left standing.
The Worktree-Teardown removal half calls it before removing the run's own pair,
so a failure halts the row with the run's pair still standing, and resuming the row retries the removal.

**Smaller pieces.**
`runprofile.DocsProfile` names the docs profile once; `retroengine.TaskProfile` now uses it.
`darnengine.IsWriter` recognizes a darn writer strand, and `IsMergeTaskWriter` builds on it.
`shedrun.ReadRunUsageRecord` reads `run.json` alone through the existing usage parser.

## Design decisions

- **The profile comes from the run's identity record, not the `boardengine` entry.**
  Fabric may learn nothing new of `boardengine`, so the guard reads the record the run's seed writes in the worktree.
  That record already holds the profile the run took, the entry's own or the recipe's default, so no default lookup is needed.
- **The slug form is strict, `<slug>-b<N>`, not any slug that starts with the run's slug.**
  Real slugs share prefixes, such as `fabric-core` and `fabric-core-teardown`,
  and a teardown that removed every pair under the prefix would remove another run's pair.
  The guard does not read the board for a clashing entry; the teardown does, and skips such a pair.
- **No cap on the number of bench pairs.**
  Whether to cap the number was left open; the bench body's own cap on runs already bounds it, so none is added.
- **Bench pairs keep their history.**
  The teardown archives each pair's tips before removing it and leaves its branches on origin, as the orphan sweep does, so nothing the bench committed is lost.

## Tests

- `TestIsBenchPairSlug` pins which slugs read as a bench pair, including a shared-prefix slug and a zero or leading-zero number.
- `TestSandboxedRoleGuard` gains a subtest: add and remove pass from the worktree and from its records sibling, for a numbered writer role too.
  They are refused with `--parent`, with `--adopt-local`, for a slug outside the form, from another worktree,
  with no run record, on another profile, on another recipe, and under a non-darn role.
  It also pins that only a darn writer's refusal names the bench pair way forward,
  and that a bench pair's push passes in the writer's hub but not in another hub or under another role.
- `TestRemoveBenchPairs` drives the teardown over a hub holding two bench pairs and a shared-prefix pair.
  Both bench pairs are archived and removed and the shared-prefix pair stands, also with a live process registered under a bench pair.
  A `boardengine` entry naming a bench slug keeps that pair, and a refused archive push keeps the pair and returns the error.

## Merge danger

**Door:** two-way
**Blast radius:** teardown
