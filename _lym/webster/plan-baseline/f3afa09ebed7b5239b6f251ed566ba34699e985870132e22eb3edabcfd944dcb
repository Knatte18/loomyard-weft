---
format: 5
approved: true
language: go
---

# Plan: warp-visibility — heddle

LoomYard puts no trace and no `CLAUDE.md` in a repository's warp, yet gives its agents instruction text of its own: heddle stores that text in `heddle.json` in the Bolt checkout and renders it into git-excluded files at each worktree's root and into each Claude session's appended system prompt.
Fabric gains an exclude list and a worktree seam heddle plugs into, the board-tailored Bolt replay seam becomes a registry of replayers sharing one by-entry merge, and the Bolt directory name is declared once.
The cards run in order: PATTERN first, the Bolt name (D9), fabric's exclude list and seam (D5, D6), the shared merge and replay registry (D8, D7), then heddle's store, rendering, seam, replayer, prompt seam, CLI and the root pre-run installs (D1–D4, D6, D7).

## Card Index

1 — pattern-entries — prescribe the three PATTERN lines and their background files
2 — bolt-dir-name — declare the Bolt directory name once in lymcwd and derive fabric's and runproc's names from it
3 — fabric-bolt-dir-sweep — move fabric's Bolt uses, messages and comments onto the Bolt names
4 — fabric-test-bolt-dir-sweep — make fabric's tests take the name from the constant
5 — module-bolt-dir-sweep — bring the other packages' messages and comments in line
6 — module-test-bolt-dir-sweep — make the other packages' tests take the name from the constant
7 — module-bolt-dir-uses — move other packages' Bolt-checkout paths from BoardDir onto BoltDir
8 — stencil-skill-bolt-dir-sweep — clear stencils and skills of the literal and guard the one declaration
9 — fabric-exclude-list — add the warp_exclude key and seed it whenever fabric wires a warp worktree
10 — fabric-worktree-seam — add fabric's made and took-in seam and fire it from add, clone, pull, origin reconcile and the parent-pair fast-forward
11 — entrymerge-package — draw the three-way by-entry merge into its own package
12 — board-replay-entrymerge — run the board replay over the shared merge
13 — bolt-replayer-registry — replace the board replay seam with a registry of Bolt replayers and a run ledger
14 — heddle-store — add heddle.json's store, entries and target table
15 — heddle-render — render file targets atomically and read prompt passages
16 — heddle-worktree-seam — implement fabric's worktree seam in heddle
17 — heddle-bolt-replayer — register heddle's by-entry Bolt replayer
18 — shuttle-prompt-seam — append the claude-prompt passages through a shuttle seam
19 — heddle-cli — add the lym heddle verbs
20 — root-prerun-installs — install heddle's seams in the root pre-run

## Shared Decisions

### Decision: no fabric side token in heddle's reach

- **Decision:** every name heddle, `cmd/lym` or `claudeengine` spells carries no side token: the seam is `WorktreeSeam`/`WorktreeEvent`/`InstallWorktreeSeam`/`EventFor`/`NotifyWorktreeMade`, the refusal text naming the `warp_exclude` key comes from `fabricengine.ExcludeWayForward`, and `lym heddle render` reaches `fabricengine.SeedWarpExclude` through `EventFor` rather than calling it.
- **Rationale:** PATTERN-fabric-vocabulary bars a side token in any identifier, string or comment outside its owner set, owner exports included, and `TestEnforcement_FabricVocabulary` enforces it; `SeedWarpExclude` keeps the record's name inside fabric.
- **Applies to:** cards 9, 10, 14–20

### Decision: the clone's "made" fires from `CloneAndWire`

- **Decision:** `CloneHub` returns before the prime is wired, since it hands config and wiring to `fabriccli.CloneAndWire` to avoid the `configsync` import cycle; the clone's "made" event fires from `CloneAndWire` right after its `WireJunctionsWith`, through `fabricengine.NotifyWorktreeMade`, the same function `Topology.add` calls.
- **Rationale:** D6 asks for "made" once the worktree is wired, its exclude lines seeded and the Bolt checkout present; that point lies in `CloneAndWire`, and every clone (the CLI and `hubforge`) goes through it.
- **Applies to:** card 10

### Decision: a seam failure never fails a verb

- **Decision:** a worktree-seam failure is a Warn log naming the worktree and `lym heddle render`, and a warning field on `PullResult` and `BranchOutcome` (printed by `OriginReport.Summary`); a passage-reader failure is a Warn log naming `lym heddle list` and a launch with lym's own sentences.
- **Rationale:** D6 rules a seam failure a warning; a malformed `heddle.json` must not stop every spawn of the hub.
- **Applies to:** cards 10, 18

### Decision: the Bolt replay needs its run ledger

- **Decision:** with no run ledger registered, a `replayBolt` that meets a claimed commit, the ended-set read and the owner read fail with `ErrReplayerMissing` naming the registration, as the old seam failed when uninstalled; a path no registered replayer claims is cherry-picked.
  This departs from D7's acceptance wording "a claimed path with no replayer fails naming the registration": fabric learns a path is claimed only from a registered replayer, so the failure it can raise is the one for a claimed commit or a ledger read with no run ledger registered.
- **Rationale:** D7 keeps the ended-set rule in fabric, reading it only through the run ledger, so a replay without one cannot set aside an ended run's commits; recognising `heddle.json` as claimed while heddle is unregistered would need fabric to name a heddle file, which D7's "fabric knows only that a Bolt file has an entry-level replayer" and PATTERN-heddle-instructions' "fabric names no heddle file" forbid.
- **Applies to:** cards 13, 17

## PATTERN changes

- `PATTERN-durable-vs-ephemeral-state` — changes: Adding file: tracked under `_lym`, never-tracked under `.lym`, same subpath, bar heddle's rendered instruction files and their `.heddle` temporary directory at a warp root; only `internal/lymdirs` names either, bar a `tools/` script's variable block.
- `PATTERN-heddle-instructions` — adds: Rendering heddle entries into a worktree file or a session's system prompt: only from `heddle.json` in the Bolt checkout, through `internal/heddleengine` (new); that file written only by `lym heddle` verbs and heddle's Bolt replayer; fabric names no heddle file.
- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.
