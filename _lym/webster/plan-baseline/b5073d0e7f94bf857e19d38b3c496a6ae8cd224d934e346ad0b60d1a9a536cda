# Card 9 — reconcile-absent-pair

**Edit:**
- `internal/fabricengine#ReconcileScope`
- `internal/fabricengine#EntryOutcome`
- `internal/fabricengine#originTargets` -> `func originTargets(l *lymcwd.Location, scope ReconcileScope) (targets []originTarget, pairs []string, warpDir, recordsDir string, absent bool, err error)`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#DropLocal`
- `internal/fabricengine#OriginReport.Summary`
- `internal/fabricengine_test#TestReconcileOrigin_Judge`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine_test#newOriginHub`
- `internal/fabricengine_test#originPair`
- `internal/fabricengine_test#writeStatus`
- `internal/fabricengine_test#publishBoard`
- `internal/fabricengine_test#originBoard`

**Intent:** Decision record D1's run-scope option: a reconcile of a run whose pair worktree is absent judges its entry instead of erroring.

- `ReconcileScope` gains `PairMayBeAbsent bool`, meaningful only with `Run` set; only batten's D1 resume sets it.
- `originTargets`, under it, when the run's task worktree is not a registered worktree of the code repository, returns no error and `absent` true: the slug still joins `pairs`, and the `wanted` filter matches no warp worktree at all, so neither a branch of the absent pair nor the parent's branch is listed, whether or not the records worktree still stands and `ReadOriginFor` finds the origin record.
  Bolt's target is still listed, as every scope lists it, so the owner and ended reads have origin's Bolt tip.
  Without the option it keeps today's `no pair worktree` error; `DropLocal` (reached from `lym adopt`) leaves the option unset and ignores the new `absent` return, so its behaviour is unchanged.
- `ReconcileOrigin` passes the slug on as today: `recordedPairs` keeps it when the prime holds its batten run record, the owner and ended reads run over origin's Bolt with no pair, `reconcileEnded` calls no teardown for the run's own slug, and `classifyEntries` judges the entry with no branches of its own.
  `EntryOutcome` gains `PairAbsent bool` (`pair_absent`, omitempty), set on the run's entry when `originTargets` answered `absent`.
  `OriginReport.Summary` prints the entry with a `pair absent` mark.
- Extend `internal/fabricengine/doc.go`'s reconcile run-scope paragraph with the option.

Test: `TestReconcileOrigin_Judge` gains rows (`tier1`, its existing fixtures), each calling `fabricengine.ReconcileOrigin` directly rather than the package's `reconcile` helper, which passes empty seams and fails on any error:
- A run scope with `PairMayBeAbsent` of a slug whose batten run record stands in the prime and whose task worktree is removed with `git worktree remove`, its records worktree still standing with the origin record naming the prime's branch: no error, one entry for the slug with `PairAbsent` true and state active, and no branch outcome for the slug or for the prime's branch.
- The same with origin's Bolt recording the slug owned by another machine (`publishBoard` and `originBoard`, with an `OwnerOf` seam reading it): the entry reads `owned_elsewhere`, `PairAbsent` true.
- The same with origin's Bolt recording the slug's entry done (`publishBoard`): the entry reads ended, `PairAbsent` true, and a recording `Teardown` seam is never called.
- The same scope without the option: an error naming `no pair worktree`.

**ImpactSummary:** Adds one scope field and one entry field read by no existing caller; re-signs the private `originTargets`, whose two callers `ReconcileOrigin` and `DropLocal` are on this card, and every other run-scope caller (fabriccli remove, Finalize, loomcli owner check, battencli) leaves the option unset.

**Commit:** `9: A run-scope reconcile may judge a run whose pair is absent`
