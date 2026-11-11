```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review: can each guard, refusal, branch and test row the cards add be reached at the point it guards?

**MEDIUM** — Card 4, `TestName_Set` row `CwdHubIsSweptOnce`: a tier-0 row cannot reach the working-directory-hub branch.
- `hubsToSweep` finds the hub of the working directory with the real `lymcwd.ResolveWorktree(cwd)` and `preflight.BoardLymPresent`. No deps seam stands in for either.
- `internal/lymcwd#resolveCore` starts with `gitWorktreeRoot(cwd)`, which needs a real git worktree (`internal/lymcwd/lymcwd.go:120-123`). `BoardLymPresent` stats `BoltDir(HubPath)/_lym` (`internal/preflight/predicates.go:75-77`).
- A tier-0 test with only `fakeHubs` has no git hub. The row's working directory therefore resolves to no hub, and the branch it names ("once each, cwd hub last") goes unreached.
- Fix: move the row to the tier-1 hubforge test, where hub B already holds the working directory and is also recorded. Or put the hub-of-cwd lookup behind a `deps` seam.

**MEDIUM** — Card 4, `RenameSweepsEveryHub`: by the time this subtest runs, the "owner only origin held before the sweep" state is gone.
- `CollisionWithAnOriginOnlyOwnerIsRefused` runs first on the same hub A. Its step-4 pull already brings the `hubforge.CloneSecond` beams into A's local Bolt.
- The rename subtest therefore never sees an owner that only origin holds.
- The verb's own step-4 pull also runs before `SetOwners` in every run, so even a fresh fixture brings origin's owners local before the write's pull. The `SetOwners` pull-time selection is reached at the verb level only when the step-4 pull failed or origin moved in between.
- The acceptance criterion "an owner only origin held before the sweep included" (D6) is therefore pinned only by card 1's `ReplaceSetSelectsAfterThePull`. The verb-level claim in card 4's subtest list does not hold.
- Fix: give each subtest its own fixture, and say which test pins the after-pull selection.

**LOW** — Card 4, step 4: the `readOwners` error branch ("fails that hub as in step 7 and the check goes on") has no row.
- It is reachable through the `fakeHubs` `readOwners` seam (a malformed `board.json` in production).
- The card leaves open whether step 7 then still sweeps that hub, and no row pins either outcome.

**LOW** — Card 4: "A failed registry read refuses before any write, naming the record's path" has no row.
- The `hubs` deps seam reaches it, but no row of `TestName_Set` or `TestName_SweepsHubs` returns an error from it.

**LOW** — Card 4, `hubsToSweep`: when the working directory is the hub container directory itself (`<hub>`, not a worktree), the hub is not found.
- `gitWorktreeRoot` fails there (`internal/lymcwd/lymcwd.go:120-123`), so the hub is not added, though D5 says "plus the hub of the working directory when it lies in one".
- The way forward ("run the verb from inside it") then fails from that directory.
- Either state in the Intent that the hub counts only from inside a worktree, or resolve the container too.

**LOW** — Card 1: two branches can only be reached through a request no entry point sends, and no test reaches either.
- A slug "both named in `Slugs` and selected is written once": the verb's only caller of `Replace` passes no `Slugs` (card 4 step 7).
- `Prime: true` together with a prime owner that is in `Replace`: the Intent does not say whether the `OwnedElsewhere` lost-check or the replace selection wins.
- Neither can occur from a real entry point. Drop the dedupe sentence, or state it as a fail-loud invariant.

Calls made:
- Read `lymcwd.ResolveWorktree`/`resolveCore`, `fabricengine.BoardDir` and `BoltDir`, and `preflight.BoardLymPresent`. My context had only their names and signatures, not whether they need git or a `_lym` stat, which decides whether the tier-0 row can reach the working-directory-hub branch.
