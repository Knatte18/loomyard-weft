# Agent names: `<code>:<slug>:<role>` for every strand, with parentage

## Goal

Every strand lyx spawns through reed carries one hierarchical name, `<code>:<slug>:<role>` in a task worktree and `<code>:<role>` in the prime and in a standalone run.
That one name is the reed strand name, the `LYX_STRAND_NAME` env value, the tmux pane title and the Claude session name.
Every worktree run records the full name of the session that spawned it, and every strand in that worktree is told it as `LYX_PARENT`, so the driver's questions and notices reach that session through Claude Code's own SendMessage.
Each repo gets a short code, given at `lyx fabric clone` and recorded on weft:main; no name can be formed without it.

## Scope

In:

- The short code: its grammar, its record on weft:main beside `.lyx-warp`, `lyx fabric clone --code`, and a new `lyx fabric code` verb for existing hubs.
- One new leaf package that owns the name grammar: forming, parsing and validating names and their segments.
- Standalone runs (`lyx webster run` and `lyx burler run` with `--target-dir`): a code derived from the target, told by `standalonegeom`.
- reed: full names in strand state, `-N` numbering for several strands of one role, `LYX_STRAND_NAME` and `LYX_PARENT` injection on launch and resume, the pane-title mirror with `allow-set-title off`, watchdog drift repair for pane titles and session names, and a new `lyx reed list` verb.
- The Claude engine: `--name` on both the launch and the resume line.
- Parentage: a `parent` field in the shed run seed, written by `lyx batten run` and `lyx loom start`, and carried from a batten run into its child's seed.
- The ly-drive skill and the loom driver prompt: escalations also go to `$LYX_PARENT`.
- Docs: `CONSTRAINTS.md`, `docs/overview.md`, and the reed and fabric package docs.
  `manifest/designs/reed-mailbox.md` is left alone: `manifest/` is being retired (bolt task `manifest-retire`), and bolt's `creel` note already records that this grammar supersedes its addressing.

Out:

- creel (the mailbox, `lyx creel` verbs, envelopes, connector domains): deferred as the tier-3 note `creel`.
- `LYX_REED_ID` injection: creel's need, not this task's.
- Producer stencils (discussion, plan, judge and the rest): they stay autonomous file-writers.
  orch-discussion-review's live question mode is a separate task that builds on this one.
- Names for processes reed does not spawn: detached watchers and daemons (`lyx orch watch`, the reed watchdog daemon), and hand-started Claude sessions.
- A machine-wide check that two hubs do not share a short code.
- Any resolver for the conversational short form (`orch`, `landing:driver`) outside reed's own `--name` flags.

## Decisions

### Short code

- **Grammar: `[a-z][a-z0-9]{1,5}` (2–6 characters, no `-`, `:`, `.` or `@`).**
  Rationale: short enough to prefix every name (`ly`, `mh`, `qu`), and free of every separator the name grammar and the later creel grammar split on.
- **Record: a single-line file `.lyx-code` at the board root, committed on weft:main beside `.lyx-warp` and `.lyx-anchor`.**
  fabricengine owns its reader and writer, as it owns `.lyx-warp`'s; lyxcwd does not read it, since cwd resolution never needs it.
  Rationale: the code is per-repo structural data with the same lifecycle as the warp binding, and later clones derive it from weft:main the same way.
- **`lyx fabric clone --code <code>`.**
  A clone that binds a fresh weft refuses without `--code`, naming the flag.
  A clone of a weft that already records a code derives it; a `--code` that disagrees with the record refuses, naming the recorded code; an equal one is a no-op.
  A clone of a bound weft that records no code accepts `--code` and records it; without the flag it succeeds and its envelope carries a warning naming `lyx fabric code`.
  `--reset` keeps the record.
  Rationale: the operator decided the code is given the first time a weft is bound; the bound-but-uncoded branch covers repos bound before this task.
- **`lyx fabric code [<code>]`: with no argument it prints the recorded code; with one it records and commits it onto weft:main through Bolt.**
  It is idempotent when the argument equals the record and refuses when it differs.
  It runs from any worktree of the hub.
  Rationale: existing hubs get their code once through a fabric verb, as decided; refusing a change keeps every name already in use valid.
- **A spawn on a hub with no recorded code refuses, naming `lyx fabric code <code>`.**
  Rationale: the operator decided no name can be formed without the code; the refusal carries its way forward per the Refusal Way-Forward Invariant.
- **No machine-wide short-code check.**
  Rationale: no machine-wide hub registry exists, and the likeliest clash, two clones of one repo, shares a code by derivation, so a check could only refuse a legitimate setup.

### Name grammar

- **One new leaf package, `internal/agentname`, is the sole former, parser and validator of names.**
  It imports no other internal package.
  Rationale: the board requires one owner; a leaf keeps reed, fabric, shuttle and shed free to import it.
- **Full name: `<code>:<role>` in the prime, `<code>:<slug>:<role>` in a task worktree.**
  Two segments mean prime, three mean task; no segment contains `:`, so the split is unambiguous.
  Rationale: decided with the operator; it supersedes both `process-naming`'s `<slug>:<role>[.<repo>]` and reed-mailbox's `name@slug[.hub]`.
- **Slug segment: the task worktree's name, validated against fabric's slug grammar `[a-z][a-z0-9-]*`.**
  A task worktree whose name fails the grammar refuses a spawn with a way forward.
  Rationale: the slug is always the worktree the process runs in; fabric already keeps slugs unique within a repo.
- **Role segment: `[a-z][a-z0-9-]*`, an open set.**
  The caller's existing role token is the role (`driver`, `orch`, `discussion`, `judge`, `worker`, `loom-status`, `claude`, …).
  agentname declares constants only for the roles other processes address by convention: `orch` and `driver`.
  Rationale: roles grow with every producer, so a closed list in agentname would rot; guessability matters only for the addressed roles.
- **Several of one role: reed numbers them.**
  The first strand of a role in a worktree takes the bare role; each later one takes the lowest free `-N` from 2 upward (`worker`, `worker-2`, `worker-3`), counted against every strand in that worktree's state, live or dormant.
  A caller may instead pass an explicit role with its own suffix (`orch-design`); an explicit name already held refuses rather than renumbers, and `--if-absent` keeps matching on it.
  Rationale: the auto rule needs no caller change for parallel workers and reviewers, while an explicit name is one a process means to address, so silently renumbering it would misroute.
- **A standalone run's code is derived from its target: `s` followed by the first five hex digits of the hash8 that `standalonestate.Derive` computes.**
  `agentname` owns the derivation rule; `standalonegeom.ReedGeometry`, which already receives the hash8, tells reed the code, and its strands take the two-segment form `<code>:<role>`.
  `LYX_PARENT` is never set there, since a standalone run has no seed.
  The `strand_name` retirement applies unchanged: standalone loads the same reed `Config` from its state directory.
  A standalone run has no watchdog and no `lyx reed list` row, so its pane titles and session names are set at launch only.
  A hub never takes a derived code: an uncoded hub still refuses.
  Rationale: the Claude engine's lines reference `$LYX_STRAND_NAME` in every mode, so a standalone strand needs a name; a standalone target has no weft to record a code on, and the hash8 already identifies it, so the derived code never changes the way a repo-name fallback would once a code is set.
- **reed's `--name` flags accept either the role segment, read relative to the current worktree, or a full name.**
  Rationale: people and agents write the short form; reed already scopes `--name` to the current worktree.
- **The `strand_name` template key in `reed.yaml` and its `<ROLE>`, `<ROUND>` and `<SHORT_GUID>` tokens retire.**
  The round and the guid prefix leave the name.
  Rationale: the name is now formed by agentname alone; the round and guid prefix made names unaddressable, and `-N` covers concurrency.

### Mirrors

- **reed forms the full name inside `AddStrand`, under its state lock, and stores it in the strand record as a birth attribute.**
  reed is told the name prefix (code, and slug or prime) and the worktree's parent through its geometry, built by `hubgeom.ReedGeometry` in a hub and `standalonegeom.ReedGeometry` in a standalone run; reed resolves neither.
  Rationale: forming the name where the state lock is held makes `-N` assignment race-free; telling the prefix keeps the Told-Geometry Invariant.
- **Every process reed starts, on launch and on resume, gets `LYX_STRAND_NAME=<full name>`, and `LYX_PARENT=<parent full name>` when the worktree has a parent.**
  Rationale: the name and parent are frozen at process start, so a rename is a relaunch, as reed-mailbox already settled.
- **The Claude engine passes `--name` on both its launch and its resume line as a reference to `$LYX_STRAND_NAME`, emitted through `internal/shell`.**
  Rationale: shuttle builds the line before reed forms the name, so the pane shell expands the name reed injected; the resume line must carry it because reed replays that line verbatim.
- **reed sets each strand pane's title to the full name (`select-pane -T`) and turns `allow-set-title` off on that pane.**
  Delivery and lookup never resolve through a pane title or a session name; strand state is the only key.
  Rationale: Claude Code retitles its terminal continuously; the title is a display mirror, as reed-mailbox settled.
- **The per-hub watchdog's tick repairs drift and logs every repair.**
  For pane titles it compares each tracked pane's title with the strand name and rewrites it.
  For session names it goes through a provider seam that reedengine declares and cliwire fills with the Claude engine's implementation, so reed stays provider-blind.
  The Claude implementation reads the session transcript's latest `custom-title` entry and, when it differs from the strand name, types `/rename <full name>` only when the session is idle.
  A missing or unreadable transcript logs that the name could not be read and counts as no drift.
  Rationale: Claude resets the session name after a compact, so setting it at launch is not enough; the operator assigned the repair to the watchdog.
- **`lyx reed list` is the hub-wide name directory.**
  One row per strand across the hub's worktrees: full name, guid, worktree, pane id, the pane's current title, alive or dormant, and a drift flag; it emits the standard JSON envelope.
  Rationale: the board names it the directory, and the drift flag makes it the diagnostic for the mirrors.

### Parentage

- **The shed run seed gains an optional `parent` field holding a full name.**
  The seeding site fills it from the spawning process's `$LYX_STRAND_NAME` at first seeding; an unset or empty env leaves it absent.
  The seed stays write-once: a later start from a different process never rewrites the field and is not refused over it.
  Rationale: the spawner recording itself at create time is the decided mechanism; refusing on a parent mismatch would block an operator resuming a hub-spawned run.
- **`lyx batten run` records its caller in the batten seed, and `Seed-Child` copies that same parent into the child's seed.**
  `lyx loom start` records its caller when it seeds an unseeded run.
  Rationale: when the hub spawns a worktree, the hub is the parent of every process in it, not batten.
- **A worktree's parent is the `parent` of its default run; `hubgeom.ReedGeometry` reads it and tells reed.**
  The default run is run-id `self`, which `shedrun` resolves to the worktree's slug, so its seed is `_lyx/shed/<slug>/seed.json` (`_lyx/shed/self/` only as shedrun's legacy fallback); it is read through `shedrun`, never a joined `self` path.
  The prime has no parent.
  Rationale: one parent per worktree is what the board asks for, and the default run is the run every task worktree is driven by.
- **No parent recorded means the operator, and the worktree's existing hand-back path (stop report, awaiting) is the whole escalation.**
  Rationale: decided on the board.
- **The hub's name comes from `lyx orch start`: the orch strand is named `<code>:orch` by reed like any other strand.**
  Until `lyx orch` can host the hub (bolt note `orch-live-ready`), a hand-started hub session records itself as parent with a per-command env prefix, `LYX_STRAND_NAME=<code>:orch lyx batten run <slug>`, and names its own Claude session `<code>:orch` with `/rename`; this needs no code, since seeding already reads `$LYX_STRAND_NAME`.
  It is not a deliverable of this task: no repo doc carries it, and the hub session applies it from this record.
  A hand-started session gets no watchdog repair, so it re-runs `/rename` after a compact.
  Rationale: env set by reed is the only source lyx reads, and a per-command prefix sets it for exactly the seeding call; without this path no run spawned by today's hub has a parent, and `parent-review` would skip its review on every run.

### Delivery

- **Delivery is Claude Code's SendMessage to the parent's full name.**
  The ly-drive skill and the loom driver prompt are changed so that every escalation, besides its existing stop report, sends one short notice to `$LYX_PARENT` when it is set, naming the run and the report path; a failed send leaves the stop report as the escalation.
  Rationale: SendMessage already addresses sessions by name, and creel later replaces it with the same names.

## Constraints

- Cwd Resolution Invariant and Told-Geometry Invariant: reed and shuttle are told the name prefix and parent; only `hubgeom` reads the code record and the seed to tell them, and only `standalonegeom` tells the derived standalone code.
- Standalonestate Leaf Invariant: `internal/standalonestate` does not change; the code is derived from the hash8 it already returns.
- Fabric Git Invariant and Fabric Write-Side Containment: `.lyx-code` is committed onto weft:main through Bolt, as `.lyx-warp` is.
- hubforge Fabric-Fixture Invariant: test hubs get their code through fabric clone, so hubforge fixtures pass a code.
- Refusal Way-Forward Invariant: every new refusal (missing code, disagreeing code, bad grammar, an explicit name already held) names its remedy.
- CLI / Cobra Invariant: `lyx fabric code` and `lyx reed list` carry `Short` and help-tree coverage.
- Shell Mechanics Seam: the env-var reference in the Claude command lines and the env injection go through `internal/shell`, for both bash and pwsh.
- `strand_name` retirement: the key leaves reed's `Config` and both `reed.yaml` templates; a `reed.yaml` that still carries it keeps loading with the key ignored, as reed already ignores its retired display keys in state, and `lyx config reconcile` removes it.
- tmux rewrites `:` and `.` in session names, so a full name is never used as a tmux session or socket name; it goes only into pane titles, env and state.
- reed's existing `Strand.Parent` field is a layout guid; the naming parent is a different value and must not reuse it.
- Shed Run-Directory Invariant: `internal/shedrun` stays the sole reader and writer of `seed.json`, including the new field.
- Documentation Lifecycle: `CONSTRAINTS.md` gains an Agent Name Invariant in the same commit.

## Auto-mode assumptions

- The short-code grammar `[a-z][a-z0-9]{1,5}` was picked without the operator; the examples `ly`, `mh` and `qu` fit it.
- The record's file name `.lyx-code` was picked without the operator.
- Auto `-N` numbering plus explicit suffixes, with bare for the first instance, was picked over either one alone.
- The role segment is an open, grammar-checked set rather than the fixed vocabulary in the older `process-naming` note.
- The parent for a whole worktree is read from its default run's seed (run-id `self`, resolved to the slug).
- Only the ly-drive skill and the loom driver prompt send to `$LYX_PARENT`; producer stencils are untouched.
- The session-name repair lives behind a provider seam filled by cliwire, not inside reedengine.
- A spawn on an uncoded hub refuses rather than falling back to the repo name; after deploy each existing hub needs one `lyx fabric code` before it can spawn.
- The current hub session becomes a recorded parent through the env-prefix path in Parentage until it moves under `lyx orch start`.
- Standalone naming was picked without the operator, who never addressed standalone mode: a code derived from the hash8 (`s` plus five hex digits) rather than a refusal.

## Open risks

- Two hubs on one machine with the same code (two clones of one repo) produce identical session names; SendMessage then needs Claude's own `[ref]` disambiguation, and lyx does not detect it.
- `allow-set-title` must exist in the tmux build reed drives on both Linux and Windows; an older tmux rejects the option.
- The `custom-title` transcript entry and the compact-resets-the-name behaviour are Claude Code internals; if the entry's format changes, every read finds no entry, so the watchdog retypes `/rename` on each idle tick and logs each repair.
- The watchdog typing `/rename` and the orch watcher typing `/clear` or a resume prompt both gate on the same idle probe, but are separate processes; a narrow window lets one land in a pane the other just made busy.
- Whether `/clear` resets the session name is unverified; the watchdog repair covers it either way.
- A parent session that no longer exists (hub stopped, renamed by a relaunch) makes SendMessage fail; the stop report stays the escalation.
- Rollout: once this lands and is deployed, every spawn on an uncoded hub refuses, including spawns by runs already in flight in other worktrees.
  The deploy step records the code first (`lyx fabric code ly` in loomyard's hub, right after `update-plugins.sh`); a run that hits the refusal in between lands as blocked and resumes after the code is recorded.

## Acceptance criteria

- `agentname` unit tests: formatting and parsing of two- and three-segment names; rejection of a bad code, slug and role, of an empty segment and of a fourth segment; `-N` assignment picks the lowest free number from 2, counts dormant strands, and leaves the first instance bare.
- `lyx fabric clone` integration: a fresh bind without `--code` refuses naming the flag; with `--code` the code is committed on weft:main as `.lyx-code`; a second clone derives it; a disagreeing `--code` refuses naming the recorded code; a bound but uncoded weft takes `--code`, and without it warns; `--reset` keeps the record.
- `lyx fabric code` integration: prints the recorded code; records and commits a new one; is a no-op on an equal code; refuses a different one.
- A strand spawn on an uncoded hub refuses naming `lyx fabric code`; a spawn in a task worktree whose name fails the slug grammar refuses naming its remedy.
- reed integration against real tmux: a strand added in a task worktree has full name `<code>:<slug>:<role>` in state and pane title; in the prime it is `<code>:<role>`; the pane has `allow-set-title off`; the launched process sees `LYX_STRAND_NAME`, and `LYX_PARENT` when the worktree's default-run seed (`_lyx/shed/<slug>/seed.json`) records a parent; after a reed server rebirth the resumed process sees the same values.
- Standalone: `lyx webster run --target-dir` on a plain checkout with no `.lyx-code` spawns its strands named `<code>:<role>`, with the code derived from the target's hash8, `LYX_STRAND_NAME` set and `LYX_PARENT` unset; the derivation's `agentname` unit test checks the result fits the code grammar.
- Two strands of one role in a worktree are `<role>` and `<role>-2`; an explicit name already held refuses.
- `reed remove --name` and `--if-absent` resolve both the role segment and the full name.
- Watchdog: a pane title changed by hand is restored on the next tick and the repair is logged; the session-name seam is driven by a fake provider, restoring only when the fake reports idle, and logging each repair.
- Claude engine: launch and resume lines carry `--name` referencing `LYX_STRAND_NAME`, correctly quoted for both bash and pwsh.
- Claude session-name provider: given a transcript whose latest `custom-title` differs from the name, it reports drift; given no `custom-title` entry, it reports drift; given a matching one, it reports none; given a missing or unreadable transcript, it logs that the name could not be read and reports none.
- reed config: a `reed.yaml` that still carries `strand_name` loads with the key ignored.
- Seed: `batten run` under `LYX_STRAND_NAME` records it as `parent`; the child seed carries the same parent; `loom start` records its caller on first seeding only; an unset env records no parent; re-seeding from another process leaves the parent unchanged.
- `lyx reed list` emits one row per strand hub-wide with name, guid, worktree, pane id, current title, liveness and a drift flag.
- Help-tree tests cover `lyx fabric code` and `lyx reed list`.
- The ly-drive skill and the loom driver prompt instruct a SendMessage notice to `$LYX_PARENT` on escalation and fall back to the stop report alone when it is unset or the send fails.
- `CONSTRAINTS.md` carries the Agent Name Invariant; `docs/overview.md` lists `internal/agentname` and the new fabric and reed verbs; the reed and fabric package docs describe the full name, its mirrors, `.lyx-code` and the new verbs; nothing under `manifest/` changes.

## Notes for the plan writer

- Build order: short code first (record, clone flag, `fabric code` verb), then `agentname`, then reed naming and env, then mirrors and watchdog, then parentage, then the driver-side stencils.
- reed's existing strand-name formation is `resolveStrandName` and `FormatStrandName` in `internal/reedengine`; `hubgeom.ReedGeometry` and `standalonegeom.ReedGeometry` are the two tellers to extend.
- Spawn sites that pass `Role` or `NameOverride` today are the callers of `shuttleengine.Spec` and `reedengine.AddSpec`; `loomengine.LoomDriverStrandName`, `orchcli`'s `orchStrandName` and loomcli's `statusStrandDisplayName` become role segments.
- `.lyx-warp`'s reader and writer in `internal/fabricengine/warpbinding.go` are the pattern for `.lyx-code`.
- The orch idle probe in `internal/orchengine` is prior art for the Claude idle check the session-name repair needs.
- `manifest/roadmap.md` has no entry for this item, so it does not move.
