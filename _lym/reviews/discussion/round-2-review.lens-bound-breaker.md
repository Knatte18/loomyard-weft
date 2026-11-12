```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guarantee and claim about existing code that the decision record and its acceptance criteria state.

**Claims attacked**
- D1 "no new branch, worktree or junction": held.
- D2, the fork keeps `_lym/testcover/` from the parent's weft tip and drops nothing else: held (Constraints fact; `forkrecords.go` per round 1).
- D3, two tasks never conflict on a map file: held, since weft branches never merge.
- D3, "the parent's next upkeep pass" carries maps forward: held as worded. D10 makes that pass manual, and Open risks states the window.
- D4, the commit runs under fabric's records write lock with a positive pathspec only: held. `CommitRecordsPaths` takes the weft write lock and runs `StageAndCommit` over the scoped pathspec (`internal/fabricengine/commitweftpaths.go:40-75`).
- D4, a killed call's maps are committed by the next call: held as a claim. A call that refuses before its end commit (a second `--upkeep` during a live pass, or a `prepare` refusal) defers the commit and loses nothing, because the files stay on disk.
- D4, the pair push of the code branch from the verb: **broken** (F1).
- D5, the exemption covers `_lym/testcover/` alone: held.
- D6, "commits nothing" on refusal: held. D11's removal before the refusal deletes untracked bolt files and commits nothing.
- D6, a hub with no owner reads as owned by no other machine: held, per `pattern/PATTERN-owner-machine.md:30`.
- D6, the environment and commit-absent cases degrade to whole-package runs: held. See `testselect.go:244-273`; `commitMiss` returns nil only for a map with no files.
- D7, two readers compare a map's commit with HEAD: held for the audit filter (`testmapaudit.go:38`) and the package guard (`testmappackage.go:82`). `FailedAt` (`coveragemap.go:109-112`) also compares `m.Commit == commit`, but only for a map with a build failure, which is always written.
- D7, "Outside a pass it is the exit record's `planned_head`": held for the dobby. No reader outside a pass exists in a task, where hand calls without `--upkeep` write no exit record (`testmap.go:705`).
- D7 and Open risks, "the stores never auto-gc, and only a pair teardown runs `git gc --auto`": held. See `housekeeping.go:38-50` (gc.auto=0, maintenance.auto=false) and `:98-102`.
- D7, a fetch never prunes, and a gone commit reads `MissCommitUnreadable`: held.
- D8, the indented form is still read and the format stays 2: held.
- D9, "A pass registers in `internal/runproc` … the task's slug in a task", so teardown ends it: **broken** (F2).
- D9, the one-pass refusal and `TestMapRunning` count only same-worktree passes: held. `testMapRunning` lists `runproc.HubOwner` only (`internal/beamcli/dobbydeps.go:619-625`).
- D10, a prime whose layers never set the key reads the new default: held. No `test_map_upkeep` key is in the hub's `_lym/config` or `.lym/config`, so this hub flips.
- D11, "removes an old map only once the prime holds a committed map", so a killed import loses nothing: held. Uncommitted imports stay on disk, and the old file stays until a commit exists.
- D11, the "has no map" import check against a concurrent builder: **broken** (F3).
- D11, the maps are "current at `79f0eb4a`": **broken** in wording only (F4).

**Findings**

F1 — MEDIUM, design, D4 (the push sentence) with D6
- **Summary:** D4's push in a task pair carries the code branch, but no run-scope origin reconcile gates it.
- **Detail:** `PATTERN-owner-machine` makes a loom pair's commit and push wait for a run-scope origin reconcile that reads the entry as active. D6 only checks the owner through `ParentOf`. A hand `lym gate testmap` in a task whose entry origin has ended or paused would still spawn the pair push of its code branch.
- **What limits it:** `PushWarpRebaseFreeAt` skips a task branch that origin lacks or has rewound (`internal/fabricengine/spawn.go:225-228`). Nothing else bounds the push, and the record states no bound.
- **Severity:** the handler should judge whether this is a `PATTERN-owner-machine` violation, which would make it BLOCKING.
- **Fix:** state that the verb's pair push is gated by the run-scope reconcile, or is records-only in a task. Otherwise state the bound.
- **Evidence:** `pattern/PATTERN-owner-machine.md:14-20`; `internal/fabricengine/spawn.go:58-129` (no reconcile).

F2 — MEDIUM, design, D9 (registration and teardown sentences) and its D9 teardown criterion
- **Summary:** only a `--upkeep` call registers in runproc, so a hand call in a task never registers and teardown never ends it.
- **Detail:** `runTestMap` calls `registerUpkeepPass` only when `request.upkeep` is set (`internal/gatecli/testmap.go:252-256`), and `--upkeep` is a hidden flag (`testmap.go:134-137`). D9 says that in a task "the operator or an agent runs it by hand", and that "a pass registers … under the task's slug", so teardown ends it.
- **Counterexample:** the redundancy sweep's `lym gate testmap internal/foo` cannot take `--upkeep` at all (`testmap.go:238-239`), and neither registers nor is ended. Teardown then removes the weft worktree under a live call. That call's `lockPackage` `MkdirAll` (`testmap.go:681`), `WriteMap` and D4's end commit then run against a removed worktree.
- **Fix:** say which calls in a task register (every call, or every call naming no package), and that teardown ends them. Otherwise state the bound for a hand call that teardown does not end.

F3 — LOW, design, D11 (the import condition)
- **Summary:** the import's "package has no map" check is not under the package's build lock, so a concurrent call can overwrite a fresh map with the old one.
- **Interleaving:** call B (a hand prime call) finds no map for package X. Call A, a live pass holding X's build lock (`testmap.go:674-685`), writes X's new map. B then writes the old bolt map over it.
- **Result:** the map goes back to the old commit and stamp. The next plan diffs from that commit, so this costs rework, not a wrong selection.
- **Fix:** state that the import takes each package's build lock, or state this bound.

F4 — NIT, consistency, D11 (rationale) and support log
- **Summary:** the bolt maps record commit `5e41ee2b`, not `79f0eb4a`.
- **Evidence:** the bolt maps carry `commit` and `planned_head` `5e41ee2b06892e7cb9e0ca3090ddae27921cdf8b` (`/home/knatte/Code/loomyard-LYMHUB/_bolt/.lym/testcover/testmap-exit.json`, `…/internal/lock/map.json`). `5e41ee2b` descends from `79f0eb4a` with no Go diff between them, so the import's premise holds.
- **Fix:** name the maps' own commit.

**Tool calls**
1. Grepped gc config, the hub's `test_map_upkeep` layers, and the bolt maps' exit record and one map: the context had none of these, and the D7, Open-risks, D10 and D11 claims rest on them.
2. Read `commitweftpaths.go` and searched for the push spawners: D4 cites the scoped commit, and the context lacked its body.
3. Ran `git` reads on `5e41ee2b` against `79f0eb4a`: needed to test D11's "current at `79f0eb4a`" claim against the maps' recorded commit.
4. Grepped `spawn.go` for the detached push and any reconcile: needed for D4's push and `PATTERN-owner-machine`.
