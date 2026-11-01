# heddle: render lym's instruction text without a trace in warp

LoomYard leaves no trace in a repository's warp and puts no `CLAUDE.md` there, yet its agents still get instruction text of its own.
That text now lives in one database in the Bolt checkout, `heddle.json`, and lym renders it into git-excluded files in each warp worktree, or into the system prompt of each session shuttle launches.

## What changes

- **`lym heddle`** (new `internal/heddlecli` over `internal/heddleengine`): verbs `list`, `get`, `set`, `remove`, `show` and `render`.
  An entry has a `slug`, markdown `text`, an integer `order` and a non-empty list of `targets`.
  Writes save under the Bolt write lock and start the board's detached Bolt sync; heddle pulls and commits nothing in the foreground.
  Every refusal names its way forward.
- **File targets**: `claude-file` renders `CLAUDE.local.md` and `codex-file` renders `AGENTS.override.md`, both at the warp root.
  `CLAUDE.local.md` imports the repository's `AGENTS.md` (root or `.claude`) when the worktree has no `CLAUDE.md`, because a `CLAUDE.local.md` alone would hide it from Claude.
  `AGENTS.override.md` starts with the repository's `AGENTS.md`, because Codex loads one file per directory.
  Each render carries a marker comment; heddle never overwrites or removes an unmarked file and warns instead.
  Files are written atomically through a `.heddle` directory under the warp root, under a per-worktree lock, and `heddle.json` is read under the Bolt write lock for a bounded wait.
- **Prompt targets**: `claude-prompt` entries are appended after lym's own sentences in Claude's append-system-prompt file, through a new seam in `internal/shuttleengine` that `claudeengine.Prepare` reads.
  `codex-prompt` entries are stored and shown; `codex-engine` will read them through the same seam.
- **Fabric**: a new `warp_exclude` config key (Cross-pair, default `CLAUDE.local.md AGENTS.override.md .heddle`) is seeded as anchored lines into the warp repository's `info/exclude`, also through the new `fabricengine.SeedWarpExclude`.
  A new warp seam fires "made" from `Topology.Add`, `AddLocal` and `CloneHub`, and "took in commits, these paths changed" from the warp arm of `Fabric.Pull` and from `ReconcileOrigin`'s code-branch fast-forwards.
  Heddle implements it; a render happens on "made", and on a take-in only for a target whose re-render paths changed.
  A seam failure is a warning naming `lym heddle render`, carried in `Fabric.Pull`'s result and `OriginReport`'s `Summary`; it never fails or undoes the move.
  Fabric names no heddle file.
- **Bolt replay**: `BoardReplay`, `InstallBoardReplay` and the board paths in `replayBolt` are replaced by a registry of Bolt replayers.
  The board registers `board` and is the run ledger; heddle registers `heddle`.
  The ended-set rule, set-aside tags and owner read go through the run ledger, so fabric holds no board file name.
  A set-aside for a non-ledger replayer is tagged `set-aside/replayer/<name>/<short sha>`.
  `ErrReplayerMissing` arises only when a claimed commit, the ended-set read or the owner read finds no run ledger registered.
- **`internal/entrymerge`** (new): the by-entry three-way merge drawn out of `boardengine.FabricReplay`, shared by the board and heddle replayers.
- **Bolt directory name**: the `_board` literal is declared once, in `internal/lymcwd`, and re-exported as `fabricengine.BoltDirName`.
  `fabricengine.BoardDirName` derives from it, `fabricengine.BoltDir` (new) gives the Bolt checkout, and `runproc.HubOwner` derives from the `lymcwd` constant.
  Messages, tests, comments, stencils and skills no longer spell the literal, and a guard enforces the single declaration.
  This lets the later `bolt-dir-rename` task change one value.
- Pattern files: `PATTERN-heddle-instructions` is new, and `PATTERN-durable-vs-ephemeral-state`, `PATTERN-leaf-packages` and the other patterns that named the literal are updated.

## Design decisions

- Files go at the warp root, not the anchor, so a session at any anchor reads them and an anchor change moves nothing.
- `internal/lymcwd` holds the literal instead of `fabricengine`, because `fabricengine` imports `lymcwd` and `runproc`; `runproc` gains an import of `lymcwd` (recorded in `PATTERN-leaf-packages`).
- `lym heddle` resolves the hub through `lymcwd`, not `internal/cliwire`, since heddle has no standalone mode.
- A change to `heddle.json` is never pushed into a live worktree, and a hand edit of `AGENTS.md` in a worktree is not watched.
  A live Claude session picks up a render at its next compaction, and an adopted session reads heddle afresh on resume; both are accepted exceptions to rules not shifting mid-run.
- Out of scope: PATTERN.md in heddle, any hub's heddle content, the `bolt-dir-rename` task, and a worktree pinning the text it started with.

## To check by hand

- `internal/fabricengine/livestate_doc_test.go`: the quoted error text is reworded with the Bolt directory name elided and carries no exemption from the single-declaration guard; confirm no `_board` literal remains there.
- A repository that tracks `AGENTS.override.md` hides the `codex-file` entries from Codex, since heddle skips an unmarked file; the warning names it and no fix exists.
- The `claude-file` render judges only the warp root: an `AGENTS.md` between the root and a session's anchor stays hidden from Claude while a `CLAUDE.local.md` render exists.
- Worktrees that exist before this lands get no files until `lym heddle render` runs in each, or a take-in changes one of a target's re-render paths.
- The `claude-file` behaviour depends on Claude Code's documented load order of `CLAUDE.local.md` and `AGENTS.md`, checked against v2.1.316; confirm against a live session.
