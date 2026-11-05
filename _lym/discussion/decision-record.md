## Goal

```markdown
## Seen

The hub NORCE Models wants its LoomYard branches named `hanf/<slug>`, through fabric's `branch_prefix` (currently `""`, template).
The prefix cannot be set safely while any pair is live: the code re-derives the warp branch as `cfg.BranchPrefix + slug` from the current config at every later step instead of reading the branch the pair was created with:

- `internal/fabricengine/remove.go` (`warpBranch := t.cfg.BranchPrefix + slug`, the local branch check, the weft sibling via `RecordsBranchName(t.cfg.BranchPrefix+slug)`).
- `internal/fabricengine/pairparts.go` (`remoteHeadTip` on `BranchPrefix+slug`; slug recovered with `CutPrefix(branch, cfg.BranchPrefix)`).
- `internal/pairteardown/pullrequest.go` (`taskBranch = cfg.BranchPrefix + slug` when finding the PR).
- `internal/fabricengine/remoteteardown.go` (the job carries the current `BranchPrefix`).
- Also `taskbranchdiscard.go`, `cleanup.go`, `cleanupremotewarp.go`, `addparent.go`, and the managed-branch ownership checks in `checkout.go` and `reconcile.go`.

So setting the prefix while beams and tasks run makes their teardown, PR lookup and remove look for `hanf/<slug>` and miss the real `<slug>` branches.
A hub with a long-lived beam never has a moment with no live pair, so the prefix effectively cannot be introduced.

## Chosen fix (operator, 2026-11-05)

The pair's worktree is the source of its branch name, not a new record and not the current config:

- The warp worktree's path follows the slug, and git knows which branch each worktree has checked out (`git worktree list`), so every step that starts while the pair's worktree exists reads the warp branch from it, and the weft branch from the weft worktree the same way.
- Steps that run after the worktree is gone, the detached remote teardown and discard jobs, carry the branch names read when the removal started; the job already has fields for them, so it stops carrying `BranchPrefix` for that purpose.
- The PR lookup in `pairteardown` takes the branch read from the worktree.
- The steps that go the other way, from a branch to its slug (orphan cleanup, remote warp cleanup, pair listing, managed-branch ownership), accept a branch with the current prefix and one without, so pairs created before the prefix changed stay managed.
- `branch_prefix` then only names the branches of pairs created after the change, and can be set at any time.

## Acceptance

- With live pairs created under prefix `""`, setting `branch_prefix: hanf/` leaves their remove, teardown, PR lookup, listing and cleanup working, and new pairs get `hanf/<slug>`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-05): A pair's branch is the one its worktree has checked out, under any prefix

Decision: The narrowing is refused (review F2). pairTaskBranchLocal accepts the task worktree's checked-out branch when it is a branch of the pair, recognised by its records branch existing in the records repository, not only a branch named by the current prefix or the bare slug. So pairs created under any earlier prefix, empty or not, stay recognised after branch_prefix changes again.

Rationale: The operator chose the worktree as the source of a pair's branch (2026-11-05) so that branch_prefix can be set at any time; a fix that only covers a change away from the empty prefix leaves the same fault one prefix change later.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - detached-literal-survives-next-to-constant (class consistency, severity BLOCKING, open in rounds 2)
  - prefix-changed-between-nonempty-values-unrecognised (class decision, severity MEDIUM, open in rounds 2)
  - unresolvable-records-root-skip-untested (class scope, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
