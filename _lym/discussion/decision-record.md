# Decision record — warp-visibility: heddle

## Goal

LoomYard leaves no trace in a repository's warp and puts no `CLAUDE.md` there, yet still gives its agents instruction text of its own.
Lym renders that text from one database in the Bolt checkout, `heddle.json`, into git-excluded files in each worktree it makes, or into the system prompt of each session shuttle launches.
Fabric gains two general seams heddle plugs into and names no heddle file, the board-tailored records replay seam becomes general, and the Bolt directory name lives in one constant.

## Scope

In:

- The `heddle.json` store, its `lym heddle` verbs, and rendering it to file targets and to Claude's appended system prompt.
- A fabric config list of warp paths to exclude, seeded through fabric's exclude seeding.
- A fabric seam for "a warp was made" and "a warp took in remote commits, these paths changed", which heddle implements.
- The reshaped Bolt replay seam, with the board and heddle each registering a replayer, and the shared by-entry merge drawn out of `boardengine`.
- `fabricengine.BoltDirName` (new) and `fabricengine.BoardDirName` as two names, the `_board` literal declared once, every message, test, comment, stencil and skill brought in line.

Out:

- PATTERN.md in any form in heddle; it stays tracked in warp.
- Any heddle content for a given hub: the operator adds entries with the verbs.
- A worktree or run keeping the heddle text it started with: board note `run-pins-its-version`.
- Codex's engine; heddle stores Codex entries, and `codex-engine` reads them through the seam of D4 when it is built.
- The rename of `_board` to `_bolt`: task `bolt-dir-rename`, which lands after this one.
- Board names in fabric outside the records replay, the ended set and the owner read that D9 does not rename, such as the write lock's file name `board.lock`.

## Decisions

### D1 — heddle.json: the store

`heddle.json` sits at the root of the Bolt checkout, beside `board.json`, and is the only source of what heddle renders.
It holds a list of entries; each entry has:

- `slug`, unique, matching `[a-z0-9-]+`;
- `text`, non-empty markdown;
- `order`, an integer; entries render in ascending order, ties broken by slug;
- `targets`, a non-empty list of target names from heddle's target table (D3).

An absent file holds no entry.
The file is a Go-written record (PATTERN-tolerant-records): the file type and the entry type each keep the keys they do not know, as `board.json`'s `storeFile` and `Task` do, so an entry `set` updates, and its neighbours when one is added or removed, keep their unknown keys; every rewrite, a replay's included, goes through those types.
Heddle holds only what is specific to using LoomYard on the repository; lym's own sentences that lym needs to work, the creel trust sentence, the batching sentence and the deny notices, stay in `claudeengine`'s code.

Rationale: the operator's entry names "the files it renders into"; naming targets from a table lets a further target such as `PATTERN.local.md` be one new row, and a numeric order merges field by field in a replay where an array position would not.
`internal/state` alone merges array elements by index and drops unknown keys of an updated or moved element (`internal/state` package doc), so the types keep them themselves.

### D2 — lym heddle verbs and the write path

`internal/heddlecli` (new) is a cobra subtree under the root in `cmd/lym/main.go` (PATTERN-cli-cobra), over `internal/heddleengine` (new).
Verbs:

- `list` — every entry, in render order, without its text.
- `get <slug>` — one entry in full.
- `set <slug>` — create or update an entry from flags for its text (from a file or stdin), order and targets; a new entry needs all three, an update changes only those given.
- `remove <slug>` — delete an entry.
- `show <target>` — print what that target renders to now, writing nothing.
- `render [<worktree>]` — render every file target into the named warp worktree of the current hub, the current one by default, under D3's rules.
  The name is a warp worktree's directory name, the last element of a path `lym fabric list` prints, and resolves through `lymcwd.NamedWorktree`; a name matching no listed worktree, or only one git marks prunable, is refused, so the verb writes files and seeds an exclude file only in an existing warp worktree of its own hub.

A write follows the board's own write path: it saves `heddle.json` under the Bolt write lock the board's writes and Bolt's pull and replay take, the one `internal/lock` file `fabricengine.BoardWriteLockFile` names, and then starts the board's detached Bolt sync, which commits what is dirty under that lock and pushes.
Heddle pulls and commits nothing in the foreground and starts no sync of its own.
A write never touches a live worktree (D6).
Refusals: an invalid slug, an empty text, an unknown target, a missing field on a new entry, an unknown slug on `get` or `remove`, a `render` argument naming no warp worktree of the hub, and any verb outside a hub, each naming the way forward (the valid values, `lym heddle list`, `lym fabric list`, or running inside a hub's worktree).

Rationale: the operator rules every write goes through `lym heddle` under the lock the board uses; the board's writes save under the lock and leave commit and push to the detached sync (`internal/boardengine/sync.go`), so heddle's writes serialize with Bolt's pull and replay the same way.

### D3 — Targets and file rendering

Heddle's code holds the target table; each row names its delivery and, for a file, its file at the warp worktree's root and the warp paths whose change re-renders it:

- `claude-file` → `CLAUDE.local.md`: when the worktree's root holds no `CLAUDE.md`, there or in its `.claude` directory, one import line for each `AGENTS.md` it holds there or in its `.claude` directory, `@AGENTS.md` and `@.claude/AGENTS.md`, then the row's entries, in order, joined by blank lines; re-rendered on a change to `CLAUDE.local.md`, or to an `AGENTS.md` or a `CLAUDE.md` there or in its `.claude` directory.
- `codex-file` → `AGENTS.override.md`: the repository's own root `AGENTS.md`, when the worktree holds one, then the row's entries in order; re-rendered on a change to `AGENTS.md` or `AGENTS.override.md`.
- `claude-prompt` and `codex-prompt`: the system prompt of a session of that engine (D4).

Each file opens with a one-line HTML comment marking it as heddle's render.
Rendering a file target into a warp worktree:

- It writes each file atomically: it writes a temporary file into a `.heddle` directory at the warp root, renames it onto the target and removes the emptied directory, so a reader sees the old file or the new one whole and the render leaves no file beside its targets.
  The directory sits under the warp root with its targets, so the rename never crosses a filesystem, and fabric's exclude list carries it (D5), so it never shows as untracked in warp while it exists.
  Its temporary files are named `<target>.tmp`; one a crash left is overwritten and removed by the next render.
  When the root already holds a `.heddle` that is a file, or a directory holding anything else, the render writes no file target and warns naming the path.
  Claude Code re-reads the project-root `CLAUDE.md` from disk after each compaction, and a `CLAUDE.local.md` is treated the same way (Claude Code memory docs, code.claude.com/docs/en/memory, sections "Instructions seem lost after `/compact`" and "Import additional files", checked on v2.1.316 on 2026-11-01), so a take-in or hand render reaches a live Claude session at its next compaction.
- A render of a worktree holds a per-worktree lock while it reads its inputs and writes its files: an `internal/lock` file under the pair's never-tracked `.lym` tree, its directory named by `internal/lymdirs` and its path handed to heddle with the warp root.
  Two renders of one worktree therefore serialize, and the last one's text stands whole.
- It reads `heddle.json` holding the Bolt write lock, the file `fabricengine.BoardWriteLockFile` names in the Bolt checkout, which Bolt's pull, its replay and `reconcileRecordsBranch` hold while git replaces the file; it takes that lock only around the read and releases it before it writes, so a render sees the old entries or the new, never an absent file in mid-move.
  Since a Bolt push holds that lock across its fetch and push retries, the render waits for it a bounded time (`lock.AcquireWriteLockWithin`); a wait that runs out fails the render, writing nothing, which a fabric move reports as a seam failure (D6) and a hand `lym heddle render` refuses with running it again as the way forward.
  The lock order is the render lock, then the Bolt write lock, and fabric fires D6's events with no Bolt write lock held, since a second acquire of an `internal/lock` file in one process blocks.
- It never overwrites or removes a file at the target path that lacks the marker, which covers a file the repository tracks: it skips that target and reports a warning naming the file.
- A target with no entry gets no file, and a stale render of heddle's own, marked, is removed; a repository `AGENTS.md` with no `codex-file` entry therefore needs no override.
  Removing a stale marked render is never refused by the exclude-list check below, which guards writes only.
- It refuses to write a target whose file, or the `.heddle` directory, is not on fabric's exclude list (D5), naming the path's return to fabric's `warp_exclude` key as the way forward.
- Before any render writes, the list's lines are in the warp repository's `info/exclude`: fabric seeds them before it fires either event of D6, and `lym heddle render` first has fabric seed them through `fabricengine.SeedWarpExclude` (new), which takes the exclude lock as D5's seeding does.
  Heddle runs no git, and a render never shows as an untracked file in warp, in a worktree made before this lands as in any other.

Files go at the warp root, not at the anchor: Claude Code reads `CLAUDE.md` and `CLAUDE.local.md` from the session's working directory and each parent up to the filesystem root (Claude Code memory docs; `claudeengine`'s `InstructionFiles` names both), and Codex reads one instruction file per directory from the git root down to the working directory, `AGENTS.override.md` before `AGENTS.md` (OpenAI's AGENTS.md guide, checked 2026-10-31, as the board entry records).
A root file therefore reaches a session started at any anchor, and an anchor change moves nothing heddle wrote.

Under its default Project instructions value, `claude-md-or-agents-md`, Claude Code reads `AGENTS.md` only when no `CLAUDE.md`, `CLAUDE.md` in a `.claude` directory, or `CLAUDE.local.md` sits in the working directory or above it, so a `CLAUDE.local.md` alone hides the repository's `AGENTS.md`, at the root or in its `.claude` directory, which Claude otherwise reads at session start; an `@AGENTS.md` or `@.claude/AGENTS.md` import inside a `CLAUDE.local.md` is expanded, and an `AGENTS.md` already loaded is never read twice (Claude Code memory docs, code.claude.com/docs/en/memory, sections AGENTS.md, When Claude Code reads AGENTS.md and Choose which instruction files load; behaviour from v2.1.277, checked on v2.1.316 on 2026-11-01).
A repository with its own root `CLAUDE.md` already keeps its `AGENTS.md` from Claude, so the render adds no import there.

Rationale: the override must carry the team's `AGENTS.md`, since Codex loads only one file in that directory, and `CLAUDE.local.md` must import it, since Claude otherwise skips it; the marker is what lets heddle tell its own render from a file it must not touch without running git on warp (PATTERN-fabric-git).
The import reaches every Claude session in the worktree, where setting `instructionFiles` in shuttle's settings file would reach only sessions launched with shuttle's `--settings`.

### D4 — Prompt delivery through a shuttle seam

`internal/shuttleengine` declares a process-wide seam that answers, for a hub's Bolt checkout and an engine name, the standing passages to put in a launched session's system prompt; the lym root pre-run installs heddle's implementation beside the fabric seams, and a missing seam reads as no passage.
`claudeengine`'s `Prepare` appends the `claude-prompt` entries, in order, after lym's own sentences in the run directory's append-system-prompt file, separated from them and from each other by a blank line.
The one-line rule `claudeengine`'s init check applies stays on lym's own sentences only: the file is passed to Claude Code by `--append-system-prompt-file` and is echoed nowhere.
A resume through the launch's own resume line reuses the file its launch wrote, so its passages stay; a resume that runs `Prepare` again, as an adopted session does, reads heddle afresh like any launch.
That re-read is one of two bounded exceptions to an agent's rules not shifting mid-run: an adopted session's run directory holds no earlier passages, so it takes heddle as it stands at adoption.
The other is a file render reaching a live Claude session at its next compaction (D3).
`codex-prompt` entries are stored and shown, and are read through the same seam by `codex-engine` when it is built, into Codex's `developer_instructions`.
A session launched outside a hub gets no passage.

Rationale: shuttle keeps engine specifics under `internal/shuttleengine/<engine>` (PATTERN-shuttle-provider-seam), so heddle knows no Claude flag and `claudeengine` parses no `heddle.json`.

### D5 — Fabric's warp exclude list

`internal/fabricengine/template.yaml` gains a key, `warp_exclude`, a whitespace-separated list of paths relative to a warp worktree's root, separate from `pathspec` and its junction names; its template default is `CLAUDE.local.md AGENTS.override.md .heddle`.
Fabric seeds an anchored line `/<path>` for every listed path into the warp repository's `.git/info/exclude`, never `.gitignore`, through the same exclude-file mutation the junction seeding uses, idempotently and under the same exclude lock, whenever it creates or re-wires a warp worktree (`Topology.Add`, `AddLocal`, `CloneHub`, `lym fabric reconcile`), before it fires D6's take-in event, and on `fabricengine.SeedWarpExclude` (new) (D3).
The list is never folded into the wired junction names, so health and drift checks never ask for a junction of it.
A listed path's line is never unseeded, since `info/exclude` is one file for every worktree of the warp repository and every lym worktree may write the paths; a line left after a path leaves the list excludes a file nobody writes.
The key is classified Cross-pair (PATTERN-config-layers): its line reaches every worktree of the repository, so the local layer cannot hold it.
Fabric's code names no heddle file.

Rationale: the operator rules the excludes a list of their own in fabric's config, seeded through fabric's existing seeding; the warp root anchors the lines because D3 writes there.

### D6 — Fabric's warp seam

`internal/fabricengine` declares a process-wide seam, installed by the lym root pre-run as the push notifier and the Bolt replayers are, with two events:

- a warp worktree was made: fired by `Topology.Add`, `AddLocal` and `CloneHub` last, once the worktree is wired, D5's lines are seeded and, for `CloneHub`, the Bolt checkout exists;
- a warp worktree took in commits from origin: fired, with the paths that differ between the old and new HEAD, wherever fabric moves a checked-out code branch to origin's commits: the warp arm of `Fabric.Pull`, a fast-forward or a reset, and `ReconcileOrigin`'s fast-forward at its code-branch call sites, never for a records or Bolt branch.

Every lym process, detached or not, runs the same root pre-run, so every such move reaches the seam.
A missing seam is no event, so a test binary needs no install; a pin in `cmd/lym` checks the root pre-run installs it.
A seam failure never fails the fabric verb and never undoes its move: it is logged as a warning naming the worktree and `lym heddle render` as the way forward, and the move's result carries it too, `Fabric.Pull`'s result and `ReconcileOrigin`'s `OriginReport`, which gains a per-branch warning its `Summary` prints.
A process that prints no result only logs it: the stale file stays until the next take-in of one of its re-render paths or a hand `lym heddle render`.
Heddle implements it:

- on "made", it renders every file target into the worktree (D3);
- on "took in commits", it renders again only a file target whose re-render paths (D3) are among the paths; any other path renders nothing.

A change to `heddle.json` is never pushed into a live worktree, and no hand edit of `AGENTS.md` inside a worktree is watched for (operator, 2026-10-31).
A re-render takes `heddle.json` as it stands then (operator, 2026-10-31).
A parent merge or a landing that brings a new `AGENTS.md` into a beam is not a take-in from origin and renders nothing; `lym heddle render` covers it.

Rationale: the operator rules fabric knows nothing of heddle and re-renders only when a sync settles remote commits; "made" rides the same seam so every worktree lym makes gets its files without a call at each of fabric's callers.

### D7 — A general Bolt replay seam

`fabricengine.BoardReplay` (removed), its request and result types, `InstallBoardReplay` (removed) and the board paths `replayBolt` recognises are replaced by a registry of Bolt replayers:

- Each replayer has a name and claims a set of Bolt-root-relative paths, and re-applies, by entry, a local commit's changes to its own paths onto origin's tip, reporting what it applied, lost and refused as `BoardReplay` reports today.
- `replayBolt` walks the local commits as today: a commit touching only claimed paths goes to the replayer of each path it touches, each applying its own files of that commit, then one commit is made of what they kept; a commit touching any unclaimed path is cherry-picked; seed commits are dropped as today.
- One replayer may also be the run ledger: it names the Bolt file that holds the runs and decodes from it each run's slug, whether it has ended and its created stamp, and names the runs a commit of its paths touches.
  Fabric keeps the ended-set rule, the set-aside tags and the owner read, and reads every one of them through the run ledger, never through a file name of its own.
- The ended-run set-aside covers only the run ledger's part of a commit: a commit whose ledger changes all belong to ended runs is set aside whole only when it touches no other replayer's path; otherwise its ledger part is skipped, the original is tagged as today, and the other replayers still apply theirs.
- A set-aside for a loss or refusal of a replayer that is not the run ledger is tagged `set-aside/replayer/<name>/<short sha>`, a shape no run's `set-aside/<slug>/<short sha>` takes.
  `setAsideTag`'s empty-slug fallback no longer names the board: a run ledger set-aside with no run slug takes the run ledger's replayer tag.
- The board registers one replayer, named `board`, for `board.json`, its overview files and its design files, and is the run ledger; heddle registers one, named `heddle`, for `heddle.json`.
- The lym root pre-run registers both, and a test kit registers what its test needs.
- A commit whose claimed paths have no replayer registered fails as `ErrBoardReplayMissing` fails today, naming the registration as the way forward; a read of the ended set and the origin owner read with no run ledger registered fail the same way.

Afterwards the replay, the ended set and the owner read in fabric hold no board file name, board status value or board-named type.

Rationale: the operator rules fabric knows only that a Bolt file has an entry-level replayer, and that both the board and heddle register one; the ended set and the owner read used the old seam's board file name and `done` status too, so leaving them would leave board paths in fabric.
The board's detached sync commits everything dirty in Bolt in one commit (`internal/boardengine/sync.go`), so one commit can carry a heddle change beside an ended run's entry.

### D8 — One shared by-entry merge

The three-way merge `boardengine.FabricReplay` does entry by entry, keyed by slug, field by field against the merge base, origin winning a field both sides changed and the loss reported, is drawn out into a package of its own, `internal/entrymerge` (new), parameterised by the record's key and field table.
The board's replayer keeps around it what is the board's own: handles, the run lock and other write checks as refusals, `prime_owner`, design files and the overview re-render.
Heddle's replayer uses it with the fields `text`, `order` and `targets`, refuses an entry that fails D1's validation, and writes through D1's types.

Rationale: the operator rules the merge is shared, not copied.

### D9 — The Bolt directory name in one place

- The `_board` literal is declared once, as an exported constant in `internal/lymcwd`: `lymcwd` reads the hub-wide anchor marker in the Bolt checkout and imports only the standard library, `internal/dotgit` and `internal/weftname` (PATTERN-cwd-resolution), while `fabricengine` imports both `lymcwd` and `runproc`, so neither `lymcwd` nor `runproc` can take the name from `fabricengine`.
  `runproc` takes it from `lymcwd`, which imports neither `runproc` nor any package that does, so the new edge closes no cycle; PATTERN-leaf-packages's background gains `internal/lymcwd` in `internal/runproc`'s import set, for the one Bolt directory name, and its overview line under `## PATTERN changes` stays as it is.
- `fabricengine.BoltDirName` (new) is that constant re-exported and is the name every other package uses; `fabricengine.BoardDirName` is set from `BoltDirName`, and `runproc.HubOwner` is set from the `lymcwd` constant.
- What lives in Bolt beside the board takes its place from `BoltDirName`, never from `BoardDirName`: `heddle.json`, the `_lym` stencils, specs and config, the `.lym` scratch tree, the Bolt write lock and the hub-reserved name.
  `fabricengine.BoltDir` (new), built from `BoltDirName`, gives the Bolt checkout's path, and fabric's uses of `BoardDir` that mean the Bolt checkout, `BoardWriteLockPath` included, move to it; `BoardDir` is built from `BoardDirName`.
  What is the board's own, `board.json`, its overview and design files, takes it from `BoardDirName`.
- Every message and test takes the name from the constant, bar `TestEnforcement_GeometryLiterals`, whose policed-token list, owner-row key and planted fixture sources spell the token it polices; that test's owner row for `_board` narrows to `internal/lymcwd`.
- A comment names "the Bolt checkout", never the path.
- A stencil or skill names the path through what lym already resolves for it, or as "the Bolt checkout", never as a literal.

Rationale: `bolt-dir-rename` then changes one constant's value.

### Added after Discussion (parent, 2026-11-01): The replay registry fails only when no run ledger is registered

Decision: D7's failure for a claimed path with no replayer is dropped: fabric learns a claim only from a registered replayer, so a path no registered replayer claims is cherry-picked. A claimed commit, the ended-set read and the owner read fail with ErrReplayerMissing when no run ledger is registered.

Rationale: A claim exists only through a registered replayer, so a claimed path without one cannot occur; the only missing registration fabric can detect is the run ledger.

### Added after Discussion (parent, 2026-11-01): lym heddle resolves the hub as the other hub-only CLIs do

Decision: lym heddle is hub-only and resolves the hub through lymcwd (CwdFrom, Resolve, RequireDrivableWorktree), not through internal/cliwire.

Rationale: internal/cliwire serves only the standalone-capable CLIs and may import neither lymcwd nor fabricengine; heddle has no standalone mode.

### Added after Discussion (parent, 2026-11-01): No _board literal stays in livestate_doc_test.go

Decision: The quoted error text in livestate_doc_test.go is reworded with the Bolt directory name elided, and the new guard carries no exemption for that file.

Rationale: D9 allows the literal only in the one declaration and TestEnforcement_GeometryLiterals; an exemption would also be one more spelling for the later Bolt directory rename.

## Constraints

- PATTERN-cwd-resolution: `lymcwd`'s import set is unchanged; heddle's engine is handed the warp root and Bolt paths and imports no `lymcwd`.
- PATTERN-leaf-packages: its background gains `internal/lymcwd` in `internal/runproc`'s import set, for the one Bolt directory name (D9); its overview line under `## PATTERN changes` is unchanged.
- PATTERN-durable-vs-ephemeral-state: the rendered files and their `.heddle` temporary directory are never tracked and sit outside `.lym`, which the entry forbids; see `## PATTERN changes`.
- PATTERN-sparse-hub-base: the new key reaches a hub through the template default; no hub base file gets it written.
- PATTERN-cli-cobra and PATTERN-cliwire-sole-wiring: `lym heddle` is one cobra subtree, wired through `internal/cliwire`.
- PATTERN-spawn-observability and PATTERN-run-process-owner: heddle spawns nothing of its own; the Bolt sync it starts is the board's.

## Auto-mode assumptions

- The verb set of D2 is the task's to shape (operator, 2026-10-31); the set chosen is the smallest that writes, reads, previews and repairs.
- Rendering at the warp root rather than the anchor (D3), on the documented load order of Claude Code and Codex.
- `warp_exclude` as the key's name and Cross-pair as its category (D5).
- D5 seeds warp only, reading the entry's "when it creates a warp and weft" as "when it creates a pair": nothing on the list is written in weft.
- The fabric seam carries creation as well as take-in (D6), so every worktree fabric makes gets its files with no caller-side call.
- A seam failure in fabric is a warning, not a verb failure (D6).
- An adopted resume reads heddle afresh (D4); only a resume through the launch's own line keeps its passages.
- The `claude-file` render keeps the repository's `AGENTS.md` reaching Claude by an import line, not by setting Claude Code's Project instructions value (D3).
- The one `_board` declaration lives in `lymcwd`, re-exported by `fabricengine`, departing from the entry's wording "`lymcwd` drops its own copy", which cannot hold while `fabricengine` imports `lymcwd` (D9).

## Open risks

- A repository that tracks `AGENTS.override.md` hides lym's `codex-file` entries from Codex, since heddle skips a file without its marker; the warning names it, and no fix is designed.
- The `claude-file` render judges only the warp root: an `AGENTS.md` between the root and a session's anchor is hidden from Claude while a `CLAUDE.local.md` render exists, and a root `AGENTS.md` reaches a session whose anchor holds the repository's only `CLAUDE.md`, which the team's own setup keeps it from.
- Worktrees that exist when this lands get no files until `lym heddle render` runs in each, which has fabric seed the exclude lines first, or a sync takes in a change to one of D3's re-render paths, before whose render fabric seeds them.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - detached-seam-failure-reaches-no-one (class consistency, severity MEDIUM, open in rounds 1, 2)
  - heddle-json-read-no-bolt-lock (class design, severity MEDIUM, open in rounds 2)
  - in-place-render-reread-at-compaction (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a store round-trip keeps an unknown top-level key and an unknown entry key, on an entry `set` updates and on its neighbours after an add and a remove; an absent `heddle.json` lists no entry; entries render in `order`, ties by slug.
- D2: `set`, `remove` and their refusals each have a case, every refusal naming its way forward; a write saves under the Bolt write lock and starts the board's detached sync, with no foreground pull or commit; a verb outside a hub refuses; `render` with a name matching no listed warp worktree of the hub, or only a prunable one, refuses, naming `lym fabric list`, and writes and seeds nothing; `show` writes nothing.
- D3: a `claude-file` render joins its entries in order; in a worktree with a root `AGENTS.md` and no `CLAUDE.md`, there or in its `.claude` directory, it opens with the `@AGENTS.md` import, in one whose only `AGENTS.md` sits in its `.claude` directory it opens with the `@.claude/AGENTS.md` import, and with a root `CLAUDE.md` it holds none; a render replaces its file by rename and leaves no `.heddle` directory and no untracked file in warp; a render concurrent with a Bolt fast-forward over a `heddle.json` change renders the old entries or the new, never none; a render whose Bolt write lock stays held past its wait writes nothing and fails naming `lym heddle render`; a root `.heddle` file, or a `.heddle` directory holding another file, yields a warning naming it and no file target written; a `codex-file` render puts the repository's `AGENTS.md` first and writes no file when there is no `codex-file` entry; a file at a target path without the marker is neither overwritten nor removed and yields a warning; a stale marked render is removed when its target has no entry, also when its path is off the exclude list; a write to a target off the exclude list is refused; a `lym heddle render` and a take-in render in a worktree whose `info/exclude` lacks the lines leave the lines seeded before the file is written; two renders of one worktree with different inputs leave one render's whole text.
- D4: a fresh Claude launch's append-system-prompt file holds lym's sentences then the `claude-prompt` passages in order; a resume through the launch's own resume line keeps the file; a launch with no seam installed or outside a hub holds lym's sentences only.
- D5: a made warp worktree's `info/exclude` holds `/CLAUDE.local.md`, `/AGENTS.override.md` and `/.heddle` once each after repeated seeding; weft's exclude holds none of them; a removed pair leaves the lines; the wired junction names hold none of them; `warp_exclude` is classified and closed to the local layer.
- D6: `Topology.Add` and `CloneHub` fire "made" after the exclude lines and the Bolt checkout exist; a pull that fast-forwards warp over a commit changing `AGENTS.md` fires "took in" with that path and re-renders `AGENTS.override.md`, while one changing only other paths re-renders nothing; a records or Bolt fast-forward fires nothing; a `heddle.json` change alone re-renders no worktree; a failing seam leaves the pull's result intact with a warning; a failing seam in a `ReconcileOrigin` code-branch fast-forward logs a warning naming the worktree and `lym heddle render`, puts it in the report's `Summary` and leaves the move intact; the root pre-run installs the seam.
- D7: a commit touching only `heddle.json`, made on both sides, replays by entry with no conflict; a commit touching `board.json` and `heddle.json` is applied by both replayers into one commit; such a commit whose board changes belong only to an ended run keeps its heddle change on the tip and tags the original; a commit touching an unclaimed path is cherry-picked; a claimed path with no replayer fails naming the registration; the ended set and owner read work through the run ledger as before, and with no run ledger registered each fails naming the registration; a heddle set-aside is tagged `set-aside/replayer/heddle/<short sha>`; no `board.json`, overview or design file name, board status `done` or board-named replay type is left in fabric's replay, ended-set or owner-read code.
- D8: the board's existing replay cases pass unchanged over the shared merge; heddle's replay keeps origin's value of a field both sides changed, reports it lost, and keeps an unknown entry key.
- D9: the `_board` literal appears in one Go declaration and in `TestEnforcement_GeometryLiterals`, in its policed-token list, owner-row key and planted fixture sources; `fabricengine.BoardDirName`, `fabricengine.BoltDir` (new) and `runproc.HubOwner` derive from it; stencils, specs and skills hold no `_board` literal; the geometry literal test's owner row names `internal/lymcwd` alone.

## PATTERN changes

- `PATTERN-durable-vs-ephemeral-state` — changes: Adding file: tracked under `_lym`, never-tracked under `.lym`, same subpath, bar heddle's rendered instruction files and their `.heddle` temporary directory at a warp root; only `internal/lymdirs` names either, bar a `tools/` script's variable block.
- `PATTERN-heddle-instructions` — adds: Rendering heddle entries into a worktree file or a session's system prompt: only from `heddle.json` in the Bolt checkout, through `internal/heddleengine` (new); that file written only by `lym heddle` verbs and heddle's Bolt replayer; fabric names no heddle file.
- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.

## Notes for the plan writer

- `fabricengine.ConfigDirs` hands configengine the board dir as its `Board` field; under D9 that value is the Bolt checkout, taken from `fabricengine.BoltDir` (new), whatever the field is called.
- The three-way field table in `boardengine`'s replay is the starting point for D8's package.
