# Card 10 — inert_paths: an inert-only diff runs no tier, and a whole gate carries over it

**Create:**
- `plan:internal/gatecli#allInertSince` -> `func allInertSince(ctx context.Context, cfg gateslot.Config, root, base string) bool`

**Edit:**
- `internal/gatecli#computeScope`
- `internal/gatecli#greenPlan`
- `internal/gatecli#greenRun.subtract`
- `internal/gatecli#execution.runGreenAware`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gatecli/doc.go`

**Uses:**
- `internal/gatecli#diffBaseGroups`
- `internal/gatecli#greenRecord.newestWholeAncestor`
- `internal/gatecli#scopesConfigWith`
- `plan:internal/impactset#ChangedPaths`
- `plan:internal/gateslot#Config.AllInert`

**Intent:**
Apply a hub's `inert_paths` at narrowed and whole gates alike (Decision 5); with the empty list every path below is skipped, so every scoper behaves as today.

`allInertSince` reads `impactset.ChangedPaths` from `base` to the working tree, untracked files included, and reports `cfg.AllInert` of them; a read error is logged at Warn and reports false, so the scope stays as it would be.

- Narrowed gates: under the `go` and `none` scopers, a computed or `--base` scope takes each listed tier's base through `diffBaseGroups`, as the `go` scoper does today, and a base group whose diff `allInertSince` holds inert gets the empty share, so no listed tier's command runs but a `NoCarry` tier's, which card 9 runs whole.
  Under `go` the inert check runs before `impactset.DeriveTargets`, which it then skips for that group, and the guard step stays planned over a module scan of `tier0`'s dir, as a Markdown-only empty scope plans it today.
  Under `none` a group that is not inert-only stays whole, and a tier with no base stays whole; `none` reads the diff for this check only, and its scope, inert-only or not, plans no guard step, so `planTierRuns`' `--guards` refusal is never met by a call that did not pass `--guards`.
  Under `dotnet` the derivation drops inert paths itself (card 6).
  An untracked non-inert file keeps the scope as the scoper derives it, and an empty diff is not inert-only.
- Whole gates: in `greenRun.subtract`, a whole invocation, over the tier's `all_targets` in its `dir`, of a tier that is not `NoCarry`, under a non-empty `inert_paths`, that the record holds no entry for on the current tree, is carried when `greenRecord.newestWholeAncestor` finds a whole entry under the tier's current fingerprint, empty args and the same module directory whose diff to the working tree `allInertSince` holds inert.
  The new `greenPlan.inertCarried` lists those module keys, and `runGreenAware` writes a whole entry for the current tree for each through `recordGreen`, so a later gate carries it directly.
  `subtract` runs only on a clean tree, since `openGreen` reads no record for a dirty one, so the inert carry obeys `PATTERN-gate-slot`'s clean-tree rule, and the inert list is in the fingerprint (card 9), so a changed list voids the entries it would carry from.
- `doc.go`'s sentence that every form is whole under the `none` scoper is rewritten for the inert-only empty scope, and the doc gains the inert rules, their Bound (a test reading an inert file is skipped when only that file changes) and the carry.

Tests:
`TestGateTest_Scopes` gains rows with `inert_paths: ["*.md"]` set through `scopesConfigWith`.
The computed rows sit before the rows that commit to the pair, beside the Markdown-only row, so the merge base the pair diffs from holds no non-inert change:
a change to `docs/note.md` alone runs no listed tier's command under each of `go`, `none` and `dotnet`, the `go` row still running its guard step and the `none` row planning none, and a `carry: false` tier listed beside them running whole;
and an untracked non-inert file beside the inert change keeps the `go` scope as it derives it.
After the committing rows, at `--all` on a clean committed tree, a tier whole-green on an ancestor is carried and a whole entry is written for the current tree after a committed `docs/note.md` change, while it runs after a committed `lib/lib.go` change, on a dirty tree, for a `carry: false` tier and after `inert_paths` changes.
The existing Markdown-only and `none` rows, with no `inert_paths`, hold today's behaviour.

**ImpactSummary:** `computeScope` and the green carry gain branches entered only when `inert_paths` is non-empty; with the template's empty list no call changes.
