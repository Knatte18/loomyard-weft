```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism run on one concrete case, checked against the decision's purpose.

**Decisions walked**
- **D1** — loom.yaml `code_max_bounces: 2` and `discussion: sonnet[medium]`, read as `code.max_bounces` and `discussion.model`. Held: the table maps every old key named in the spec tail at `internal/loomengine/config.go:690-731`.
- **D2** — drift `judge_ask_parent` stays flat. Held: drift's spec is untouched.
- **D3** — a budget refusal for perch `code`, rendered by `configengine.KeyName("loom", "code.max_bounces")`. Held.
- **D4, local-file site** — `discussion: sonnet` in a pair's `loom.local.yaml`. `ValidateLocal`'s leaf path `discussion` is a non-mapping, so the new check matches it before the template-shape check (`internal/configengine/layers.go:249-268`). The way forward, `lym config reconcile --apply`, runs `ReconcileAll` with `PruneUnknown` (`internal/configcli/configcli.go:590`). That reaches `reconcileSparseFile`, where D5 moves the key before the prune. Held.
- **D4, worktree-file site** — same command, same path. Held.
- **D4, hub-base site** — `code_max_bounces: 2` left in the hub `loom.yaml`. `lym config reconcile --layer default --apply` from the prime runs `ReconcileHubWideAt` with `PruneUnknown` (`internal/configcli/hubwide.go:192-223`), and D5 moves the key before the prune. Held for the operator or the hub orch. A task session that runs it is refused by `hubWriteRefusal` (`internal/configcli/hubwide.go:51-63`), but the way forward names the prime, so that matches.
- **D4, run_settings site** — Seed-Child gets `{"loom":{"code_max_bounces":2}}`. `PrepareRunSettings` refuses it (`internal/configreg/runsettings.go:43`), `battencli.prepareRunSettings` wraps the refusal in `ErrInvalidRunSettings` (`internal/battencli/runsettings.go:51-53`), and `writeRunSettings` appends its single way forward (`internal/battenshed/seamchild.go:154-155`). One way forward is printed. Held.
- **D4, profile-block site** — reaches the same wrapper through `LayerRunSettings` (`internal/configreg/runsettings.go:115-117`). Editing in `lym config profiles` then goes through `copyStagedToHub`. An edit that fixes every old key passes the before/after check (`internal/configcli/hubwide.go:462-470`). Held.
- **D4, `--set` site** — `setSparse` refuses today with "unknown config key(s)" (`internal/configengine/set.go:210-212`). D4 replaces that text with the new path. Held.
- **D4, recipe-reference site** — `SettingKnown` is `configreg.CheckSettingKey` at recipe build (`internal/battencli/wire.go:1691`, `internal/loomcli/wiring.go:1244`). An old reference is refused there before any run. Held.
- **D5, hub base** — hub base `code_max_bounces: 2`, then the walk's `ReconcileHubWideAt` default branch: the move comes before `pruneSparse`. The write gate `changed` is driven only by `removed` and `dropped` (`internal/configsync/configsync.go:504-507`), so the move has to set it. The D5 criteria ("leaves `code.max_bounces: 2` … a second reconcile changes nothing") pin that. Held.
- **D5, absent hub base** — seeded from the prime: `primeConfig`, then the D5 move, then `sparseSeed` (`internal/configsync/configsync.go:451-492`). This criterion is pinned too. Held.
- **D5, pair file** — the move sets `Applied`, and `reconcileWorktree` commits worktree-layer results only (`internal/hubreconcile/walk.go:346-363`). Held.
- **D6** — shipped `solid.tapestry` block with `loom.code_max_bounces: 3` (operator), template base `2`, new template `code.max_bounces: 2`. Moving file and base first gives file `code.max_bounces: 3` against base `2`, so `followUnit` keeps the operator's value (`internal/yamlengine/follow.go:167-171`). An untouched block takes the template. Held.
- **D6, mixed builds** — an older build's walk on the moved `profiles.yaml` adds flat `code_max_bounces` at its template default to each block (`internal/yamlengine/follow.go:173-181`). The next walk of the new build sees both keys, and new-path-wins keeps the operator's `3`. Held, but no criterion pins it (finding W1).
- **D7** — board entry `{"loom":{"code_max_bounces":2}}`. The carry runs after `reconcileHubWide` returns not skipped. On a failure, `walkHub` returns skipped, so `ensure` writes no stamp (`internal/hubreconcile/walk.go:77-85`). Every start verb's `EnsureOrWarn` retries before any module load (`internal/battencli/arm.go:300-303`, `internal/loomcli/arm.go:219-224`). Run-held entries take `run_settings` writes (`internal/boardengine/runlock.go:211-213`). Held.
- **D8, reinterpretation** — `FlagCodeAutoAccept` ignores the worktrees argument and reads only the hub base (`internal/loomengine/reinterpret.go:27`), which the configsync move has already carried. Held.
- **D8, recipe reference** — `loom.split.reviewer_fan` is cut at its first dot into `loom` and `split.reviewer_fan` (`internal/shedrecipe/entries_settings.go:48`). D8 names the needed `CheckSettingKey`/`SettingLookup` nested read. Held.
- **D9** — local `discussion: {reviewer_fan: ""}`: `coveringCategory` on `discussion.reviewer_fan` finds the Guard entry (`internal/configengine/layers.go:189-202`). Held.

**Findings**

- **W1** — LOW, consistency, `_lym/discussion/decision-record.md:129`
  - Summary: D6 applies D5's new-path-wins rule to `profiles.yaml` blocks, but no acceptance criterion pins a profile block that holds both an old key and its new path.
  - How it arises: after an older build's walk adds flat template defaults beside moved values (`internal/yamlengine/follow.go:173-181`), the next walk must keep the moved value. The only collision criterion (`decision-record.md:216`) covers a loom file, not a profile block.
  - Evidence: `internal/yamlengine/follow.go:167-181`, `_lym/discussion/decision-record.md:216-219`.

Tool calls:
1. grep for `FlagCodeAutoAccept` and `code_auto_accept`: to check whether D8's reinterpretation reads worktree files the walk has not yet moved; the manifest lacked `internal/loomengine/reinterpret.go`.
2. `sed` of `internal/battencli/arm.go`, `internal/loomcli/arm.go` and a grep for `CheckSettingKey`: to check that the D7 retry point runs before any strict load, and where recipe references are checked at build; the manifest listed those Ensure callers by grep only.
