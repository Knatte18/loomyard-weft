# Fast batten start, local-only fabric pairs and one Command producer

## Goal

A batten start reaches Run-Shed in about the time its rows' own actions take: no row waits on GitHub, a row that does nothing costs milliseconds, and Open-Window costs no more than starting the window.
Fabric gains local-only pairs whose add and remove touch no origin.
Batten's flag-gated side-action rows become rows of one generic `Command` producer that runs a configured command.

## Scope

In:

1. Every per-transition status push becomes fabric's detached push: batten's `CommitStatus` seam (`internal/battencli/commitstatus.go`), loom's and darn's seam (`internal/loomcli/wiring.go`), and the records push of `lyx batten cleanup` (`internal/battencli/cleanup.go`).
2. A measurement of a batten start's rows, before and after, kept as a research results document, and the per-transition overhead it finds cut in this task.
3. Local-only fabric pairs: `lyx fabric add --local`, and a remove and a `lyx fabric cleanup` that, for such a pair, run no origin step.
4. One `Command` registry engine replacing the `OpenWindow`, `SeedIDE` and `OpenIDE` engines and `shedadapters`' switched producer, plus the `lyx` verbs the default commands call.

Out:

- Weaver, batch pairs and their scheduler; creating a pair from a parent pair's branch (note `pair-from-parent`).
- `Job-Start`, `Job-Join`, and an async mode or a `halt` failure policy of `Command`: no row in scope needs either, and those notes extend `Command` rather than add engines.
- Moving `deploy` or `publish_verify` into `Command` rows.
- The shed-wide entry gate of note `row-entry-gate`.
- Faster add or remove for ordinary, origin-backed pairs.
- Preflight's weft push (note `preflight-weft-push-sync`): it is a row's own push, not a per-transition status push, and that note settles its race with the board sync.

## Decisions

### D1 — Per-transition status pushes go detached

Each status seam keeps its commit synchronous and its three dispositions (skip-while-mid-merge, commit-hard-errors, push-warns), and replaces its synchronous push with a spawn of fabric's detached push child.
Batten's seam and `lyx batten cleanup` push the prime's records side only; loom's seam pushes the pair's records and code sides, and in the prime the records side only, the rule `PushPairAnchored` follows today.
Fabric gains one `l`-in spawn entry point that applies that prime rule, so no caller names a records path (PATTERN-fabric-vocabulary) and no caller pushes the prime's code branch.
It is also where the slug flags, the child's working and log directories below, and D3's local-mark check enter.
A spawn that fails to start warns, as a failed push warns today.
Each seam's spawn is an injected function, as `battencli`'s `spawnPairPush` is, so no test forks a real child.
`StatusPushLockWait` stays for `commit-records`, and also defaults `Remove`'s teardown-lock wait below.

Rationale: nothing reads a run's status on origin before the next row runs, and the detached child already exists, pushes under the absorbing push lock, retries GitHub's bare `(failed)` refusal with backoff and is gated by `FABRIC_SKIP_GIT`/`FABRIC_SKIP_PUSH`.

Bound: the seam no longer sees a push failure; the detached child logs it at Warn, and the next transition's spawn carries the commits, which the push-warns disposition already accepted.

One child is spawned per committed transition, never per no-op transition.
Today `statuscommit.New` pushes after every nil `Commit`, both seams discard `CommitAnchoredPaths`' committed flag, and only batten's seam has a marker skip, so a clean-path no-op still pushes.
The seams' `Commit` dependency therefore returns whether a commit landed, and the `statuscommit` core spawns the push only when one did.

Today fabric's records push lock (`fabric.push.lock` under a records worktree's lock directory) is taken blocking: the detached child's `coalescePush` takes it with `lock.AcquireWriteLock`, and the synchronous `PushAnchored` and `PushPairAnchored` wait on it through `acquirePushLock`, so a child that finds the lock held queues behind the holder, however long it hangs.
This task changes the acquisition of that lock alone and bounds the live pushers per records worktree to its holder; `Bolt.Sync`'s blocking coalescing over `board.push.lock` is unchanged:
a child writes a pending flag, one per side it owes (records, code), beside the push lock, then try-locks, and a child that finds the lock held exits, leaving its commits to the holder.
Every holder of the push lock, detached or synchronous, clears the flags of the sides it pushes before each push step and, after its last no-progress step, releases the lock and re-checks every flag.
When a flag of a side it pushes is set, it try-locks again and loops; when only a flag of a side it does not push is set, such as the code side after `commit-records`' records-only push, it spawns the detached push of that side.
A commit precedes its flag, so the last holder to release always sees a flag set after its last push, and no committed transition is left unpushed.

The spawn entry point passes the child two hidden flags, one per role.
The pause slug names the pause entry the child waits under: the pair's own slug for a push from the pair, and the caller's gate slug for a push from the prime, empty when it has none.
The child's gate install honours it, so batten's and cleanup's pushes, run from the prime, wait under the pair's pause entry as loom's do from the pair.
The teardown slug names the pair whose teardown lock the child takes, and is set only when the worktrees it pushes are that pair's own, a push from the pair; a push of the prime's records side, such as batten's seam's and `lyx batten cleanup`'s, gets none, since `Remove` deletes nothing it touches.
Bound: only fabric's spawn entry point sets either flag, and every respawn below, a holder's spawn of a side it does not push and a failed `Remove`'s spawn of the pair's push, goes through it with the slugs of the push it serves.
A child run by hand with a wrong or empty pause slug waits under that slug's pause entry instead of the pair's, and one with a wrong or empty teardown slug takes no teardown lock of the pair, so `Remove` does not wait for it, and since its worktree-existence check below holds no lock against `Remove`, it can race a removal and recreate its push lock's directory at a removed path.

The spawn entry point starts every child with its working directory in the hub's `_board` worktree, outside both of the pair's worktrees.
The working directory carried the pair identity and the hub; the flags now carry the pair identity, and `_board` still resolves to the hub, so root's pre-run, which arms the logger's sink and installs the network gate from the working directory before any fabric flag is read, resolves the hub as before.
The gate install reads the pause slug flag from the child's arguments in that pre-run, since today it takes the slug from `hubgeom.PauseGeometry` over the working directory, which `_board` no longer ties to the pair.
The sink's cwd-anchored resolution then writes the child's logs to `_board/.lyx/logs` (`fabricengine.HubLogsDir`), so they are not archived with the pair's logs at teardown.

`fabricengine.Remove`'s exclusion against children lives in a per-slug teardown lock under `fabricengine.HubScratchDir`, outside both of the pair's worktrees.
A child given a teardown slug takes that lock shared for its whole run and exits with a Warn when it finds it held exclusive.
Every child, teardown slug or none, checks that its worktrees exist before it creates the lock directory or touches anything, and exits with a Warn when they are gone; a child given a teardown slug checks while holding the teardown lock, so it leaves nothing at a removed path.
`Remove` takes the lock exclusive after every existing refusal (`RemoveRefusal`'s probe, which takes no lock) and before its first mutation, waiting at most a wait that defaults to `StatusPushLockWait` and that a test shortens, and releases it after its last step, past the worktree teardown.
A `Remove` that fails after taking the lock and leaves the pair spawns the pair's push after releasing it, so a child that exited on the held lock loses nothing.
Neither the lock file nor any child's working directory or log file sits in a tree `Remove` deletes, so no child pins or holds a handle in one on Windows, no late child's log line recreates a removed path, and no unlinked lock file stops excluding on Linux.
When the wait runs out, `Remove` refuses before any mutation, saying a push of the pair is still running or waiting under a pause, and names its way forward: re-run `lyx fabric remove <slug>` once the push finishes or the pause is resumed, or end a stuck child, whose pid its spawn logged.
`pairteardown` gets that refusal from `Remove` after its own `RemoveRefusal` probe passed, and batten's Worktree-Teardown row stands Stuck on it with that reason, its way forward `lyx batten run <slug>` once the push finishes.

### D2 — Measure, then cut what the measurement shows

The task measures a batten start from Worktree-Create's entry to Run-Shed's entry, per row and inside each row (fabric add and its probes, status write, marker, merge probe, commit, push spawn, config reads, the row's action), before and after D1 and D4.
The numbers come from the logger's trace timestamps and the run's status history, with no new timing infrastructure.
Rows a test can drive without tmux are measured by an integration-tier drive over `hubforge`; the report names the rows it measured live and those it did not.
Results go to `docs/research/batten-start-2026-10-18/` as a measurement report.
A cost the measurement finds in batten's per-transition path (seam, marker, config reads, producer wrapping), beyond the commit and the row's own action, is removed in this task.

Rationale: the board entry's cause is read from the code, not measured, and the scope asks for a row-by-row account.

### D3 — Local-only fabric pairs

`lyx fabric add --local` creates a pair on the path `SkipGit`/`SkipPush` takes today: no origin probe, no archived-leftover replacement, and its caller spawns no detached push.
A local pair gets no `_portals` link and no `_launchers` entry, so a hub workspace never lists it.
Add marks both branches local in their repos' git config (a per-branch key fabric owns), so the mark lives and dies with the branch.
The skip sits where every fabric push passes, so a later commit never publishes a local pair.
The spawn entry point checks the mark first and spawns no child for a local pair, recording the skip on its caller's result, so `Fabric.Commit` and the seams report it where they report a spawn today.
`PushAnchored`, `PushPairAnchored` and `lyx fabric push` skip a marked branch and report it skipped on their results.
The detached child, which has no caller to report to, skips a marked branch through the same check and logs it.
`archiveWeftTip`'s tag push, a raw `gitexec` push outside the `gitrepo` push primitives, and `deleteRemoteBranch`, the destructive gate's remote delete, check the mark too, and both `lyx fabric remove` and `lyx fabric cleanup` write to origin only through them.
`lyx fabric remove` of a local pair runs no origin step: no archive tag, no remote delete, no ls-remote; `--remote` is accepted and reported as having nothing to do.
Every existing refusal (slug, prime, target-exists, merge-in-progress, dirtiness) still runs first, and `--force` still answers dirtiness only.
`lyx fabric cleanup --apply` of a local pair's orphan weft branch pushes no archive tag and deletes nothing remote; the branch itself goes by the records rule below.

A pair counts as local when either of its surviving branches carries the mark, so a pair one of whose branches was deleted by hand still runs no origin step.
A pair with no surviving mark is removed as an ordinary pair.

Bound on add: `--local` refuses when either branch already exists unmarked, locally or as an `origin` remote-tracking ref, naming a plain `lyx fabric add` as the way forward, so it never adopts and silences an ordinary pair.
A plain `lyx fabric add` that adopts a marked branch, such as a local pair's warp branch its remove kept, clears the mark and records that on its result, so the adopted pair is ordinary.

Bound on remove: a local pair has no archive, and its weft is never merged (`internal/fabricengine/doc.go`), so its records exist only on its weft branch.
A local pair's records are disposable: its remove commits no pending records, archives nothing, and deletes the weft worktree and branch with every record on them, recording the dropped weft tip's sha on its result.
The weft worktree's removal gate (`dirtyScopeAll` in `removeWeftWorktree`) takes a records-disposable kind on this path, so pending records are discarded without `--force`; the task worktree's dirtiness refusal still needs `--force` as today.
A caller that needs a local pair's records, such as a parent run reading a batch pair's results, reads them before the remove.
The warp branch keeps today's gate: `deleteWarpBranch` deletes it only when its work is on another ref or landed on the recorded parent, and otherwise keeps it with `CodeBranchKeptReason` while the remove succeeds, so a local pair's unlanded code is never lost and `--force` never answers that gate.
So no new refusal guards a local remove, and no flag answers one.
The mark is written only by `lyx fabric add --local`; a key set by hand on an ordinary pair's branch makes that pair local, so its remove archives nothing and drops its pending records, and its pushes publish nothing.
`lyx fabric remove` reports on its result that it took the local path and which branch carried the mark, so a hand-set mark shows.

Rationale: batch pairs (notes `weaver`, `parallel-batch-parts`) never reach origin, and add's network time was 92% origin probes.

### D4 — One `Command` producer

`shedadapters` gains a `Command` producer, registered as the single `Command` engine in `internal/shedrecipe`, replacing the `OpenWindow`, `SeedIDE` and `OpenIDE` engines, their `Env` seams and `SwitchedProducer`.
A row's recipe `config` names two settings as `module.key`, read when the row is reached (PATTERN-recipe-setting-lookup): the switch (bool) through `Env.Setting`, and the command (string) through a second lookup the recipe's owner tells for executable keys, plus an optional `timeout_s` (default 30).
That bound is the producer's contract, not one recipe's: the executable-key lookup resolves over the template and the board default layer only, so no layer a pair can write reaches any recipe's command.
A `Command` row whose owner tells no executable-key lookup refuses when it is reached, before running anything, with an error naming the row and its way forward: wire an executable-key lookup over the template and the board default layer in the recipe's owner, then re-run the run on the rebuilt `lyx` (for batten, `lyx batten run <slug>`).
Batten's rows keep their names (resume depends on them) and point at `batten.open_window`/`open_window_command`, `batten.seed_ide`/`seed_ide_command` and `batten.open_ide`/`open_ide_command`.
The switch keys keep `CategoryNone`, so a task's `run_settings` still switches a row per task; the command keys are `CategoryExecutable`.
Batten wires no `Env.Setting` today, and it is wired in the prime before the task worktree exists, so its lookups resolve each key when the row is reached, as `switchedActions`' flag read does today.
A switch key resolves through `Env.Setting` over the task worktree's config stack (`fabricengine.ConfigDirs` of the task location), whose local layer holds the task's `run_settings`.
Batten's executable-key lookup resolves over `configengine.Dirs` with only `Board` set, the layers `deploy` and darn's `verify`, both hub-wide, are read from; the task worktree's tracked and local `batten.yaml` never reach it.
An operator sets a command with `lyx config set --layer default` from the prime, since a plain `lyx config set` writes a per-worktree module's worktree layer.
When the task worktree's stack resolves a command key to a value other than the one batten runs, the row warns that the value is ignored, naming that way to set it.
`lyx batten run --window` still forces Open-Window's switch on for that run: batten tells the producer the rows whose switch the run forces, and the producer reads no switch for a forced row.

Behaviour per call: switch off, or an empty command, is Done without acting, an empty command on a forced row logged at Warn.
Otherwise the placeholders are expanded and the command runs once through the platform shell `verifyrun` uses, in a directory and with a log path the recipe's owner tells (batten: the prime, and a log under the run's scratch directory from `internal/shedrun`).
Its start and its end are logged (PATTERN-spawn-observability), and the row is Done whether it exits 0, fails or times out, a failure logged at Warn with the log path.
A setting that cannot be read is an error naming the key and the way forward: correct it, then `lyx batten run <slug>`.

Placeholders: exactly `{lyx}` (the `lyx` path batten resolved at wiring), `{slug}` (the run's slug) and `{task_dir}` (the task worktree's anchor path); any other brace text passes through literally.
Each value is quoted for the shell that runs the command (POSIX `sh`, and `cmd` on Windows), the quoting kept beside that shell's choice so the two cannot differ.
`cmd` expands `%NAME%` inside double quotes, where no quoting stops it, so on Windows a value holding `%` runs no process: the row is Done, warned with the placeholder and its way forward, a path without `%` for that placeholder's source (the hub for `{task_dir}`, the Go bin dir `lyx` is built into for `{lyx}`).
The recipe's owner tells the producer the facts and the runner; the expansion and the policy live in the producer.

Every action that is Go code today gets a `lyx` verb, and batten's template spells the default commands with them: `lyx ide seed <slug>` and `lyx ide open <slug>` in `idecli` over `ideengine.SeedTask`/`OpenTask`, and a `reedcli` verb that opens a terminal window attached to a worktree's reed session through `internal/termwindow`.
Each verb that starts a GUI process detaches it and returns once it has started, so the window outlives the row, its timeout and batten.

Bound: a command runs whatever the template or the board default layer says, on the operator's machine, with the operator's rights.
No task branch, origin branch an add adopts, or board entry can set it: the first two write only the task worktree's tracked layer and the last only its local layer, and no recipe's executable-key lookup reads either; the local layer also refuses `CategoryExecutable` keys.
A command that hangs is ended at `timeout_s`.
No command's outcome (a non-zero exit, a timeout, a Windows percent refusal) halts the run; an unreadable setting or a missing executable-key lookup stops it at the row with the error above.

## Constraints

- PATTERN-fabric-destruction-chokepoint: the records-disposable dirtiness kind and `deleteRemoteBranch`'s local-mark check live in `internal/fabricengine/destroy.go`; `Remove`'s local-path step order and `archiveWeftTip`'s mark check stay with their current callers.
- PATTERN-gitrepo-client-boundary, PATTERN-gitexec-checked-call: the local mark is read and written through `gitrepo`/`gitexec`, never raw git.
- PATTERN-mutation-record: `Add`, `Remove` and `Cleanup` record the local-pair path's mutations and skipped steps on their results.
- PATTERN-shed-recipe-registry: `Command` is one map entry; the three engines leave the map.
- PATTERN-told-geometry: `Command` derives no path; its directory and log path are told.
- PATTERN-cli-cobra, PATTERN-cliwire-sole-wiring: the new `ide` and `reed` verbs are cobra subcommands of their modules.
- PATTERN-sandbox-coverage: the new verbs and `fabric add --local` are covered by the sandbox suite or excluded with a reason.
- PATTERN-config-layers: the new keys are batten template keys with categories.
- PATTERN-refusal-way-forward: the `--local` add refusal, `Remove`'s refusal on a running push, `Command`'s setting error, its refusal over a missing executable-key lookup and its Windows `%` warning each name their way forward, and a test reaches each.
- PATTERN-test-speed, PATTERN-test-economy: the producer and the seams are tested at Tier 1 over stubs; fabric's local pair at the integration tier over `hubforge`.
- Docs in the same commit: `internal/shedadapters`, `internal/fabricengine`, `internal/battencli` and `internal/statuscommit` `doc.go` texts, the batten recipe header comment, `docs/overview.md` if its module table names the switched engines.

## Auto-mode assumptions

- The operator ran nothing in this discussion; every choice is an auto-pick recorded in the support log's question ledger.
- The cause of Open-Window's 8 s is unmeasured; the push lock is per records worktree, so it is not the prime waiting on the child pair's lock, and D2's measurement names the cause.
- `ideengine.OpenTask` and the terminal launcher return once their process has started.

## Open risks

- The live after-numbers for the rows with an action on (Open-Window, Seed-IDE) need a live batten start; when the implementer cannot take one, the orch takes it on the first batten start after landing and adds it to the report.
- More detached pushes reach the shared records repository alongside the board sync's; their `(failed)` refusals retry off screen and show only in the children's logs under `fabricengine.HubLogsDir`, beside the reed server's runtime log.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - child-cwd-inside-task-worktree (class design, severity MEDIUM, open in rounds 2)
  - command-trust-bound-not-in-command-producer (class design, severity MEDIUM, open in rounds 2)
  - prime-push-holds-pair-teardown-lock (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-4-review.md; fixer report: _lyx/reviews/plan/round-4-fixer-report.md.
  - pending-flag-protocol-no-bound (class design, severity MEDIUM, open in rounds 1, 2, 3, 4)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - pattern-recipe-setting-lookup-missing-told (class design, severity BLOCKING, open in rounds 1)
  - respawned-code-flag-lacks-weft-path (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- Batten's, loom's and darn's status seams and `lyx batten cleanup` commit synchronously and spawn the detached push instead of pushing; a no-op transition, including a clean-path commit that lands nothing in loom's or darn's seam, spawns nothing; in the prime no spawn names the code side; no test forks a real push child.
- A failed spawn warns and the run goes on; a mid-merge skip and a commit hard error behave as before.
- A child that finds the push lock held exits, and a commit made while the holder pushes is pushed before the last holder exits.
- A child serving a pair waits under that pair's pause entry, whether spawned from the pair or from the prime.
- Teardown right after a status transition, on Linux and on Windows: the pair's in-flight push (loom's or darn's) finishes first, `Remove` releases its teardown lock only after the worktree teardown, no branch remove deleted is left on origin, and a child spawned after `Remove` leaves nothing at the removed paths, so a re-add of the slug succeeds; batten's status push spawned at the transition into Worktree-Teardown does not delay or refuse that row's `Remove`.
- A push still holding the teardown lock past `Remove`'s wait, shortened in the test, makes `Remove` refuse before any mutation, naming its way forward, and batten's Worktree-Teardown row stands Stuck on it.
- A commit on the code side made while `commit-records` holds the push lock is pushed once it releases.
- A plain `lyx fabric add` over a local pair's kept warp branch clears its mark.
- A setting of a command key in an ignored layer warns at the row.
- `docs/research/batten-start-2026-10-18/` holds before and after numbers per row; in the integration-tier drive, a row whose switch is off, and Open-Window over a command that only starts a process, each cost under 100 ms beyond the status commit and that process.
- `lyx fabric add --local` makes no network call, creates no portal or launcher, and marks the pair's branches; over an existing unmarked branch it refuses with its way forward.
- A commit in a local pair spawns no child and reports the local branch skipped on its result; an explicit `lyx fabric push` there publishes nothing and reports the local branch skipped.
- `lyx fabric remove` of a local pair pushes no archive tag and deletes nothing remote, with or without `--remote`, and removes without `--force` a fresh pair, one that ran a shed run and one with pending records, dropping its weft and records and recording the dropped weft tip, the local path and the branch that carried the mark; a warp branch with unlanded work is kept with its reason.
- A local pair with one branch deleted by hand still removes with no origin step.
- `lyx fabric cleanup --apply` over a local pair's orphan weft branch pushes nothing.
- An ordinary pair's add and remove are unchanged.
- `Command`: switch off is Done with no process; empty command is Done with no process; a command exiting 0, non-zero or past `timeout_s` is Done, the last two warned with the log path; an unreadable setting is an error naming the key and its way forward; a placeholder value with spaces, quotes or `%` reaches the command as one literal argument under `sh`, and a value with `%` on Windows runs no process and warns with its way forward; `${HOME}` and other brace text pass through literally.
- A task's `run_settings` can switch Open-IDE on and off but cannot set `open_ide_command`, which the local layer refuses.
- A command key in the task worktree's tracked `batten.yaml` is not run; the board default layer's command is.
- A `Command` row whose recipe's owner tells no executable-key lookup refuses before running anything, naming its way forward.
- `lyx batten run --window` opens the window with `open_window` off.
- A batten run stopped at Open-Window or Seed-IDE under the old binary, its commit marker naming that row, resumes at that row by name under the new one.
- `lyx ide seed <slug>` writes the task worktree's `.vscode/` config and title-bar colour, `lyx ide open <slug>` opens VS Code on it, and the reed window verb opens a terminal attached to its reed session, each returning once started.

## PATTERN changes

- `PATTERN-push-both-sides` — changes: Pushing run records from Go: the pair's code branch goes along, and a per-transition status push goes through fabric's detached push.
- `PATTERN-recipe-setting-lookup` — changes: Giving a recipe row a config value: a plain one is `module.key` via `Env.Setting`, an executable one `module.key` via the owner's told executable-key lookup over the template and board default layer, a review segment's resolved one a `SegmentSettings` field; never its own `Env` field.
- `PATTERN-command-row` — adds: Adding a side-action row to a recipe: a `Command` row over its config keys, its command read from the template and board default layer only, never a new engine.

## Notes for the plan writer

- The detached child's bypass handler (`CoalescePushBothAt`) is one of the paths the local-mark skip must cover; the `gitrepo` push primitives are where every push but D3's two named origin writes meets.
