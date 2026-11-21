# One-repository sandbox trial 3: second clone and namespace hand check

Adds `docs/research/single-repo/sandbox-3-reclone-2026-11-21/report.md`, the third and last sandbox trial of the one-repository hub layout (57015a52).
It also sums up all three trials.
The change is the report alone; no code changes.

## What the report finds

- A plain `lym fabric clone <url>` of `Knatte18/lym-sandbox-onerepo` into a second hub directory detected the one-repository layout.
  It configured the records store's `refs/lym/` refspecs with no flag given,
  and read back the board, the prime records branch and trial 2's five archive tags with their run records.
- `greet-reclone` ran from the second hub to landing as `1c74466`, with no halt and no pull request.
  Teardown deleted both of its branches from origin and pushed `refs/lym/tags/archive/greet-reclone/e421ed44a33f`.
  GitHub's branch list shows `main` alone and its tag list is empty.
- `lym fabric pull` in the first hub brought its code store to `1c74466` and its records store level with origin.
  The board stayed stale until `lym board sync`, because the pull moves the pair and not the board.
  The sync also dropped a duplicate stencil-seed commit, made when both hubs' dobbies re-executed onto a new lym binary.
- The hand check grepped `internal/fabric*` and `internal/gitrepo` at 57015a52 for `refs/heads/` and `refs/tags/`, 82 non-test lines.
  It lists every site that sends a ref name to origin or reads one back.
  None builds an origin ref from a bare prefix on a records-store path.
  One observation is outside the check's scope:
  a discarded task branch's archive tag goes to origin's plain `refs/tags/`, from the code store, in both layouts.

No lym bug was found in this trial, so no board note was filed.
Across the three trials the one bug remains B2, `rules-read-plan-gated-403`, from trial 2.

## Design decisions

- The orch ran the clone, the task filing and release, the dobby start and the first hub's pull,
  because a task session is refused each of them, as trials 1 and 2 found.
- The task session ran `lym board sync` in the first hub to see whether the diverged board settles; it did, with no conflict.

## Merge danger

**Door:** two-way
**Blast radius:** docs
