```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

## Decisions walked

- D1: a hub base `loom.yaml` holding `code_max_bounces: 2` and `discussion: sonnet[medium]`. D5's move gives `code.max_bounces` and `discussion.model`. **Purpose met.**
- D2: drift's `judge_ask_parent` is not touched. **Purpose met.**
- D3: a budget refusal through `reviewbudget.Route`, with the key built from the path constant and the module name. **Purpose met.**
- D4: an old name at each refusal site, followed through the way forward the refusal names. **Purpose missed** (F1).
- D5: three cases.
  - A local file holding `code_max_bounces: 2`: the move runs, then the prune. **Purpose met.**
  - A hub base file that already exists: **purpose met.**
  - A hub base file that is absent and gets seeded from the prime: **purpose missed** (F4).
- D6: two cases.
  - A profile the operator added: **purpose met.**
  - A shipped profile whose `loom.code_max_bounces` the operator edited: **purpose missed** (F2).
- D7: two cases.
  - An entry whose `run_settings` hold `{"loom":{"code_max_bounces":2}}` on a walk that succeeds: **purpose met.**
  - The same entry on a walk whose board write fails: **purpose missed** (F3).
- D8: two cases.
  - `lym beam merge-parent` builds `SoftKey` and `HardKey` for `code`, and the nesting helper must merge both into one `code` map. The criterion pins both keys. **Purpose met.**
  - `FlagCodeAutoAccept` runs after `ReconcileHubWideAt` has moved the file (`internal/hubreconcile/walk.go` inside `reconcileHubWide`). **Purpose met.**
- D9: the categories are renamed one for one. **Purpose met.**

## Findings

**F1 — BLOCKING, design.** Location: `decision-record.md` D4, the way-forward paragraph.
D4's single way forward, "`lym config reconcile`, which moves it", reaches no fix at four of its six refusal sites:
- **Hub base file:** plain `lym config reconcile` runs `ReconcileAll` over the worktree's own and local files only. The hub base needs `--layer default` (`internal/configcli/configcli.go` `runReconcile`).
- **Profile block:** the default-layer reconcile runs `ReconcileHubWideAt` alone, with no reinterpretations (`internal/configcli/hubwide.go`, around lines 177–230). After D6's marker is written, nothing moves `profiles.yaml` again.
- **Task `run_settings`:** no reconcile touches board entries.
- **`--set` and recipe references:** there is nothing to move. The fix is the new name.

This breaks `PATTERN-refusal-way-forward`.

**F2 — BLOCKING, design.** Location: D6 together with D5's collision rule.
Case: in a shipped profile, the operator edited `loom.code_max_bounces` from 1 to 3. `FollowTemplate` runs before D6 (`internal/yamlengine/follow.go:41-60`, called from `ReconcileHubWideAt`):
1. `code_max_bounces: 3` differs from the base, so it stays as the operator's value.
2. `code` is absent from both file and base, so it takes the template's `{max_bounces: 1, …}`.
3. D6's move then finds both the old key and the new path. D7 says D6 uses the shared helper, and that helper applies D5's rule that the new path wins.
4. The operator's 3 is replaced by the template's 1.

D6's criterion covers only a profile the operator added, so this case has no test.

**F3 — MEDIUM, design.** Location: D7's bound and its failure criterion.
A failed board write is only logged. `walkHub` still returns `skipped=false`, so `writeStamp` runs (`internal/hubreconcile/walk.go`, `ensure` and `walkHub`). "The next walk" therefore comes only with the next build, or when a repo default is missing.

Meanwhile Seed-Child refuses the entry. Its way forward is `lym config reconcile` (from D4), which never carries board entries. The entry stays stuck, and the refusal names no reachable fix such as `lym board upsert` with the new name.

D7 also never says whether its board write runs when `reconcileHubWide` comes back skipped because the board could not be pulled.

**F4 — LOW, design.** Location: D5, the first paragraph.
When the hub base `loom.yaml` is absent, `ReconcileHubWideAt` seeds it from the prime's worktree copy through `sparseSeed`. That copy still holds old keys, because the prime's own reconcile runs after the hub-wide pass (`internal/configsync/configsync.go`: `ReconcileHubWideAt`'s `fileAbsent` branch and `sparseSeed`; `internal/hubreconcile/walk.go` `walkHub` order). `sparseSeed` prunes keys the template lacks, so the tuned values are silently left out of the seeded base.

D5 lists the files its move covers (hub base, worktree, local) but does not say the move also applies to the prime copy read as a seed.

## Tool calls

1. `internal/beamcli/mergeparent.go` lines 70–95 and a grep of `internal/yamlengine/follow.go`. I needed the shape of the merge-parent `run_settings` for D8's case, and follow's unit rule beyond the doc comment in context.
