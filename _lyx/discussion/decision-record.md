# Fast batten start, local-only fabric pairs and one Command producer

## Goal

A batten start reaches Run-Shed in about the time its rows' own actions take: no row waits on GitHub, a row that does nothing costs milliseconds, and Open-Window costs no more than starting the window.
Fabric gains local-only pairs whose add and remove touch no origin.
Batten's flag-gated side-action rows become rows of one generic `Command` producer that runs a configured command.

## Scope

In:

1. Every per-transition status push becomes fabric's detached push: batten's `CommitStatus` seam (`internal/battencli/commitstatus.go`), loom's and darn's seam (`internal/loomcli/wiring.go`), and the records push of `lyx batten cleanup` (`internal/battencli/cleanup.go`).
2. A measurement of a batten start's rows, before and after, kept as a research results document, and the per-transition overhead it finds cut in this task.
3. Local-only fabric pairs: `lyx fabric add --local` and a remove that, for such a pair, runs no origin step.
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
A spawn that fails to start warns, as a failed push warns today.
Each seam's spawn is an injected function, as `battencli`'s `spawnPairPush` is, so no test forks a real child.
`StatusPushLockWait` stays for `commit-records`.

Rationale: nothing reads a run's status on origin before the next row runs, and the detached child already exists, pushes under the absorbing push lock, retries GitHub's bare `(failed)` refusal with backoff and is gated by `FABRIC_SKIP_GIT`/`FABRIC_SKIP_PUSH`.

Bound: the seam no longer sees a push failure; the detached child logs it at Warn, and the next transition's spawn carries the commits, which the push-warns disposition already accepted.
One child is spawned per committed transition, never per no-op transition, since the batten marker skip and the clean-path commit no-op come first.
Live children per worktree are bounded by the absorbing lock: a child that finds the lock held leaves its commits to the holder; if the plan finds the lock does not behave so, the child gets that behaviour in this task.
A detached child honours a hub pause as the synchronous push did, waiting under the run's pause entry.
`fabricengine.Remove` waits for the pair's push lock before its first mutation and holds it through the remote steps, so an in-flight child finishes before teardown and a child started later fails on the missing worktree and warns, never re-pushing a deleted branch.

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
The skip sits where every fabric push passes, so the detached child, `PushAnchored`, `PushPairAnchored`, `lyx fabric push` and `Fabric.Commit`'s spawn all skip a marked branch and report it skipped, and a later commit never publishes a local pair.
`lyx fabric remove` of a marked pair runs no origin step: no archive tag, no remote delete, no ls-remote; `--remote` is accepted and reported as having nothing to do.
Every existing refusal (slug, prime, target-exists, merge-in-progress, dirtiness) still runs first.

Bound on add: `--local` refuses when either branch already exists unmarked, locally or as an `origin` remote-tracking ref, naming a plain `lyx fabric add` as the way forward, so it never adopts and silences an ordinary pair.
Bound on remove: dropping the archive tag drops the only copy of unmerged work.
So remove of a local pair refuses when either branch carries a commit contained in no other local branch of its repo, other than fabric's own provenance commit on the weft, naming `--force` as the way forward; `--force` then answers both dirtiness and unmerged commits.
A fresh pair carries only its base and the provenance commit, so it removes without `--force`.
The mark never applies to a pair created without `--local`, so an ordinary pair's archive and push are unchanged.

Rationale: batch pairs (notes `weaver`, `parallel-batch-parts`) never reach origin, and add's network time was 92% origin probes.

### D4 — One `Command` producer

`shedadapters` gains a `Command` producer, registered as the single `Command` engine in `internal/shedrecipe`, replacing the `OpenWindow`, `SeedIDE` and `OpenIDE` engines, their `Env` seams and `SwitchedProducer`.
A row's recipe `config` names two settings as `module.key`, read through `Env.Setting` when the row is reached (PATTERN-recipe-setting-lookup): the switch (bool) and the command (string), plus an optional `timeout_s` (default 30).
Batten's rows keep their names (resume depends on them) and point at `batten.open_window`/`open_window_command`, `batten.seed_ide`/`seed_ide_command` and `batten.open_ide`/`open_ide_command`.
The switch keys keep `CategoryNone`, so a task's `run_settings` still switches a row per task; the command keys are `CategoryExecutable`, so only the repo default and worktree layers may set a command, the trust class of `deploy`.
`lyx batten run --window` still forces Open-Window's switch on for that run.

Behaviour per call: switch off, or an empty command, is Done without acting, an empty command under `--window` logged at Warn.
Otherwise the placeholders are expanded and the command runs once through the platform shell `verifyrun` uses, in a directory and with a log path the recipe's owner tells (batten: the prime, and a log under the run's scratch directory from `internal/shedrun`).
Its start and its end are logged (PATTERN-spawn-observability), and the row is Done whether it exits 0, fails or times out, a failure logged at Warn with the log path.
A setting that cannot be read is an error naming the key and the way forward: correct it, then `lyx batten run <slug>`.

Placeholders: exactly `{lyx}` (the `lyx` path batten resolved at wiring), `{slug}` (the run's slug) and `{task_dir}` (the task worktree's anchor path); any other brace text passes through literally.
Each value is quoted for the shell that runs the command (POSIX `sh`, and `cmd` on Windows), the quoting kept beside that shell's choice so the two cannot differ.
The recipe's owner tells the producer the facts and the runner; the expansion and the policy live in the producer.

Every action that is Go code today gets a `lyx` verb, and batten's template spells the default commands with them: `lyx ide seed <slug>` and `lyx ide open <slug>` in `idecli` over `ideengine.SeedTask`/`OpenTask`, and a `reedcli` verb that opens a terminal window attached to a worktree's reed session through `internal/termwindow`.
Each verb that starts a GUI process detaches it and returns once it has started, so the window outlives the row, its timeout and batten.

Bound: a command runs whatever a tracked config layer says, on the operator's machine, with the operator's rights; a board entry cannot set it, because `run_settings` lands in the local layer, which refuses `CategoryExecutable` keys.
A command that hangs is ended at `timeout_s`.
No row's failure halts the run.

## Constraints

- PATTERN-fabric-destruction-chokepoint: the local remove path and its guard live in `internal/fabricengine/destroy.go`.
- PATTERN-gitrepo-client-boundary, PATTERN-gitexec-checked-call: the local mark is read and written through `gitrepo`/`gitexec`, never raw git.
- PATTERN-mutation-record: `Add` and `Remove` record the local-pair path's mutations and skipped steps on their results.
- PATTERN-shed-recipe-registry: `Command` is one map entry; the three engines leave the map.
- PATTERN-told-geometry: `Command` derives no path; its directory and log path are told.
- PATTERN-cli-cobra, PATTERN-cliwire-sole-wiring: the new `ide` and `reed` verbs are cobra subcommands of their modules.
- PATTERN-sandbox-coverage: the new verbs and `fabric add --local` are covered by the sandbox suite or excluded with a reason.
- PATTERN-config-layers: the new keys are batten template keys with categories.
- PATTERN-refusal-way-forward: the `--local` add refusal, the unmerged-remove refusal and `Command`'s setting error each name their way forward, and a test reaches each.
- PATTERN-test-speed, PATTERN-test-economy: the producer and the seams are tested at Tier 1 over stubs; fabric's local pair at the integration tier over `hubforge`.
- Docs in the same commit: `internal/shedadapters`, `internal/fabricengine`, `internal/battencli` and `internal/statuscommit` `doc.go` texts, the batten recipe header comment, `docs/overview.md` if its module table names the switched engines.

## Auto-mode assumptions

- The operator ran nothing in this discussion; every choice is an auto-pick recorded in the support log's question ledger.
- The cause of Open-Window's 8 s is unmeasured; the push lock is per records worktree, so it is not the prime waiting on the child pair's lock, and D2's measurement names the cause.
- `ideengine.OpenTask` and the terminal launcher return once their process has started.

## Open risks

- The live after-numbers for the rows with an action on (Open-Window, Seed-IDE) need a live batten start; when the implementer cannot take one, the orch takes it on the first batten start after landing and adds it to the report.
- Note `row-entry-gate` would later move the switch out of `Command` into a shed-wide entry gate, and `Command`'s switch config then goes.
- More detached pushes reach the shared records repository alongside the board sync's; their `(failed)` refusals retry off screen and show only in the logs.
- Batch pairs also need a pair created from the task branch, which note `pair-from-parent` owns; `--local` composes with it.

## Acceptance criteria

- Batten's, loom's and darn's status seams and `lyx batten cleanup` commit synchronously and spawn the detached push instead of pushing; a no-op transition spawns nothing; in the prime no spawn names the code side; no test forks a real push child.
- A failed spawn warns and the run goes on; a mid-merge skip and a commit hard error behave as before.
- Teardown right after a status transition leaves no branch on origin that remove deleted, and the in-flight push finishes first.
- `docs/research/batten-start-2026-10-18/` holds before and after numbers per row; in the integration-tier drive, a row whose switch is off, and Open-Window over a command that only starts a process, each cost under 100 ms beyond that process.
- `lyx fabric add --local` makes no network call, creates no portal or launcher, and marks the pair's branches; over an existing unmarked branch it refuses with its way forward.
- A commit in a local pair, and an explicit `lyx fabric push` there, publish nothing and report the local branch skipped.
- `lyx fabric remove` of a local pair pushes no archive tag and deletes nothing remote, with or without `--remote`; a fresh local pair removes without `--force`; a pair with a commit no other branch holds refuses without `--force` and is removed with it.
- An ordinary pair's add and remove are unchanged.
- `Command`: switch off is Done with no process; empty command is Done with no process; a command exiting 0, non-zero or past `timeout_s` is Done, the last two warned with the log path; an unreadable setting is an error naming the key and its way forward; a placeholder value with spaces or quotes reaches the command as one argument; `${HOME}` and other brace text pass through literally.
- A task's `run_settings` can switch Open-IDE on and off but cannot set `open_ide_command`, which the local layer refuses.
- `lyx batten run --window` opens the window with `open_window` off.
- A batten run stopped at Open-Window or Seed-IDE under the old binary, its commit marker naming that row, resumes at that row by name under the new one.
- `lyx ide seed <slug>` writes the task worktree's `.vscode/` config and title-bar colour, `lyx ide open <slug>` opens VS Code on it, and the reed window verb opens a terminal attached to its reed session, each returning once started.

## PATTERN changes

- `PATTERN-push-both-sides` — changes: Pushing run records from Go: the pair's code branch goes along, and a per-transition status push goes through fabric's detached push.
- `PATTERN-command-row` — adds: Adding a side-action row to a recipe: a `Command` row over its config keys, never a new engine.

## Notes for the plan writer

- The detached child's bypass handler (`CoalescePushBothAt`) is one of the paths the local-mark skip must cover; the `gitrepo` push primitives are where every path meets.
