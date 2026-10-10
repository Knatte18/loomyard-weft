# Decision record: git-robustness

## Goal

Five git defects stop or stall runs; fix each at its source:

- A cached go-git handle reads a stale pack index after a repack and reports `object not found` (#468), because `Status()`, history walks and tree diffs in `internal/gitrepo` bypass `lookupObjectRetrying`.
- The hub's shared object stores start a background auto-gc after nearly every commit, merge or fetch, so repacks land under live steps.
- A git network call (push, fetch, pull, clone, ls-remote) can hang forever on a dead connection after a laptop sleep (#458).
- `lyx batten run <slug> --window` on a slug whose earlier run is `done` reports ok, opens a window and starts nothing (#447).
- `gitrepo.HeadContains` walks HEAD's whole history whenever a stencil seed pass finds a source-ahead stencil.

## Scope

In:

- `internal/gitrepo`: one reindex-and-retry wrapper around every go-git read, whole operation included; a date-bounded `HeadContains` walk.
- `internal/gitexec`: a timeout on every remote subcommand, classified as a transport failure.
- `internal/fabricengine`: `gc.auto=0` on the hub's code and records object stores, and a foreground housekeeping gc after a pair's removal.
- `internal/battencli`: the `--window` path refuses a done slug before opening a window.
- `PATTERN.md` entries for the new invariants, and the packages' `doc.go` files.

Out:

- Recovering from a staged merge-in at Publish, and the dirty-tree refusal naming `lyx fabric merge --abort`: board entry `publish-recovery` owns both.
- Making a done slug reusable by archiving its status and starting fresh, and batten's cleanup verb for finished statuses: board entry `lyx-cleanup` owns removing finished statuses.
- A step-level watchdog and Windows process-tree kill for verify: board entry `verify-watchdog`.
- Retrying a refused weft push: board entry `fabric-pushes`.
- Making any timeout or gc threshold operator-configurable.
- The `push` and `fetch` that `tools/deploy` runs through its own `exec.Command`: they run only under `update-plugins.sh`, in the operator's own terminal, where a hang is seen and interrupted.

## Decisions

### D1. One retrying entry point for every go-git read

Every go-git use in `internal/gitrepo` runs inside one helper that holds the handle lock, runs the caller's whole read (object lookups, `Worktree.Status()`, `Log` walks, tree diffs, ref reads), and on an error whose chain holds `plumbing.ErrObjectNotFound` reruns the whole read once if the pack fingerprint differs from the one the read started under, reindexing first when no other caller already has.
`lookupObjectRetrying` is replaced by this helper rather than kept beside it, and `goGit()` gets this helper as its only production caller.
A read that legitimately expects a missing object (`HeadContains` on an absent sha, `SHAExists`) maps the error after the helper returns, so the retry happens before the "absent" answer is given.
A gitrepo test pins `goGit()`'s single production caller.

Lock discipline, since `r.goGitMu` is a non-reentrant `sync.RWMutex`:

- The helper resolves the handle through `goGit()` before it takes the lock.
- It runs the read under the shared lock, and takes the exclusive lock only around the fingerprint check, `Reindex()` and the fingerprint update, then reruns the read under the shared lock.
- Under the shared lock, before the read, it snapshots `lastPackFingerprint`.
  Under the exclusive lock it computes the current fingerprint: when that equals the snapshot it returns the error, since the pack set has not changed since the read began; otherwise it reindexes only if the current fingerprint also differs from `lastPackFingerprint`, and reruns either way.
  A concurrent caller's reindex in the gap between the two locks therefore still triggers this caller's rerun, instead of making the pack set look unchanged.
- A wrapped read never calls another `Repo` method; a composite method calls the helper once per read, one after another, so each read gets its own single retry.

Rationale: #468 came from `Status()` calls that bypass the per-lookup wrapper; retrying the whole operation covers status, walks and diffs, whose object reads happen deep inside go-git where a per-lookup wrapper cannot reach.
The fingerprint gate stays, so an absent object costs a reindex only when the pack set changed.
Bound: the helper retries at most once per call and only on object-not-found; every other error passes through unchanged.

### D2. Hub object stores never auto-gc; lyx runs gc at pair removal

`internal/fabricengine` sets `gc.auto=0` and `maintenance.auto=false` in the shared config of both the code store and the records store when it creates a pair (Add) and again before each housekeeping run, so an existing hub picks them up at its next pair create or removal.
`maintenance.auto=false` switches off the post-command `git maintenance run --auto` trigger itself, whose repack tasks under a `geometric` or `incremental` `maintenance.strategy` `gc.auto` does not govern.
After a successful pair removal (`Remove`, which both `lyx fabric remove` and batten's Worktree-Teardown reach through `internal/pairteardown`), fabricengine runs housekeeping on both stores: `git gc --auto --quiet` with `gc.auto` set on the command line to git's default threshold, `gc.autoDetach=false` and `gc.worktreePruneExpire=never`, so it runs in the foreground, only when the store needs it, and never prunes a worktree's admin entry, which stays under fabric's destruction gate.
Housekeeping is best-effort: a failure, including another gc holding `gc.pid`, is logged through `internal/logger` and never fails the removal.
It lives in its own fabricengine file, not `destroy.go`: it removes no worktree, branch or file that the destruction gate's containment, ownership and dirtiness checks protect, and what the gc deletes is unreachable objects and expired reflog entries, each past git's own default expiry.
A new PATTERN entry names fabricengine's housekeeping as the only lyx code that runs `gc` or writes `gc.*` or `maintenance.*` config on a hub store.

Rationale: the stores hold more unreferenced loose objects than git's default auto-gc threshold, so almost every git command in any worktree, agents' included, starts a detached repack.
Turning automatic gc and maintenance off in the store's config stops agents' and the operator's git commands from triggering it too, not only lyx's own.
Pair removal is the one point where a run has finished and lyx is already doing slow work.
D2 bounds how often a repack lands under a live step; D1 carries correctness.
Bound: the two keys disable git's automatic gc and maintenance for everyone using the store, the operator's manual git included; lyx's housekeeping is the replacement, and a manual `git gc` still works.
Scheduled maintenance an operator registers with `git maintenance start` stays outside lyx's control.

### D3. A timeout on every git remote command, in `gitexec`'s exec core

`gitexec`'s exec core, which every git call under `internal/` already goes through, treats `push`, `fetch`, `pull`, `clone` and `ls-remote` as remote subcommands (by the existing global-option-skipping subcommand parse) and runs them:

- with `GIT_HTTP_LOW_SPEED_LIMIT=1000` and `GIT_HTTP_LOW_SPEED_TIME=60` in the environment, so a stalled HTTPS transfer fails within about a minute;
- under a 10-minute wall-clock deadline, which on expiry kills git's process group on Unix (git itself on Windows), with `WaitDelay` bounding the wait on pipes an orphaned helper still holds.

A timed-out command surfaces as a `*GitError` whose message names the timeout, and `IsTransportFailure` reports true for it, so each caller's existing transport-failure path (best-effort status pushes log and continue, transient marks at shed boundaries) takes over.
The low-speed abort is a transport failure too: `operation too slow`, curl's text when the limit fires, joins the phrases `IsTransportFailure` matches.
`pull` joins the subcommands `IsTransportFailure` classifies.
The deadline is injectable for tests and is not operator configuration.
The process-group setup verifyrun uses moves to `internal/proc`, so gitexec and verifyrun share one implementation.

Rationale: all git under `internal/` runs through `runCore`, so one site covers every network call lyx makes, best-effort or not, with no per-caller wiring.
The low-speed limit catches the #458 dead-HTTPS case quickly; the wall-clock deadline covers SSH remotes, credential-helper hangs and anything else the limit misses.
Bound: a legitimate remote command that takes longer than 10 minutes, or moves under 1000 B/s for 60 s, now fails as a transport failure; local subcommands are unaffected.
What a timeout then triggers stays bounded: a best-effort push logs and continues, and at a shed boundary the transient mark gets the driver's one re-step and then hands back.
The worst case is one deadline per remote command a step attempt runs, best-effort ones included, times two attempts.

### D4. `batten run --window` refuses a done slug before opening the window

`runInWindow` runs the same read-only status check `battenPreRun` uses before it seeds (decode failure, unrecognized state, `done`), and on a refusal prints the error envelope and exits 1 without opening a window.
Both paths share one check function, so they cannot drift.
The done-slug refusal keeps its current way forward and adds the alternative the operator used for #447: give the new task a different slug.
Seeding a fresh status stays inside the window's own run.

Rationale: the windowless run already refuses a done slug through `doneSlugRefusal`; #447 happened only because `--window` returns before any pre-run hook reads the status, and the refusal then fired unseen inside the window.
Removing a finished status, which reuse of the slug needs, is batten's cleanup verb in board entry `lyx-cleanup`.

### D5. `HeadContains` walks a committer-time-bounded window

`HeadContains` walks HEAD's history newest-committer-time first and stops with `false` once the next commit's committer time is more than 24 hours older than the target commit's.
It stays on go-git inside D1's helper.
Its contract (true for HEAD and its ancestors, false for an absent sha or unborn HEAD, `ErrInvalidSHA` for a malformed sha) is unchanged.

Rationale: a commit that descends from the target almost always has a committer time at or after the target's, so the walk only reads commits made since about a day before the build commit, instead of the whole history when the target is absent.
It keeps `PATTERN-gitrepo-client-boundary`'s split: a local read on go-git.
Bound: a descendant whose committer time is more than 24 hours older than the target's (clock skew on the committing machine) makes `HeadContains` answer false for a commit that is in HEAD.
Its one caller, `fabricengine.StencilSource`, then picks the "behind the build" drift message instead of "source ahead"; both are log lines that never block.

### D6. Docs and PATTERN

- New PATTERN entry for D1: reading through go-git in `internal/gitrepo` goes only through the one retrying helper, the sole caller of `goGit()` (test).
- New PATTERN entry for D2: hub stores carry `gc.auto=0` and `maintenance.auto=false`, and fabricengine's housekeeping at pair removal is the only lyx code that runs `gc` or writes `gc.*` or `maintenance.*` on them; the operator's manual `git gc` and maintenance registered with `git maintenance start` stay outside lyx's control.
  The entry states its carve-out: housekeeping deletes only unreachable objects and expired reflog entries past git's default expiry, never a worktree's admin entry, and is not a destructive primitive under `PATTERN-fabric-destruction-chokepoint`.
- `internal/gitrepo/doc.go`'s packfile section describes the whole-operation retry; `internal/gitexec`'s package doc describes the remote timeout; fabricengine's and battencli's docs gain their behavior.

Rationale: CLAUDE.md requires a new cross-cutting invariant in PATTERN in the same commit, and module docs to follow observable behavior.

## Constraints

- `PATTERN-gitrepo-client-boundary`: D1 and D5 keep local reads on go-git; no gitrepo method joins the pinned CLI-bound set.
- `PATTERN-gitexec-checked-call`: D3 changes the exec core under both `Run` and `RunGit`, not their call sites.
- `PATTERN-fabric-git`: D2's config writes and gc run through fabricengine via gitexec's checked form.
- `PATTERN-fabric-destruction-chokepoint`: D2's housekeeping stays outside `destroy.go`, for the reason D2 states; it must not use any of the destructive guard's banned tokens.
- `PATTERN-pair-teardown`: D2 hooks into fabricengine's removal after the worktree is gone, so the session-end-before-removal order is untouched.
- `PATTERN-spawn-observability`: the housekeeping gc and a timed-out git kill are logged through `internal/logger`.
- `PATTERN-transient-stop`: D3 adds no transient mark; it makes timeouts visible to the existing `IsTransportFailure`-based marks.
- `PATTERN-no-denied-recovery`: no refusal or doc in this task names a denied command.
- `PATTERN-test-speed`, `PATTERN-test-isolation`: tests that spawn git sit in the `integration` tier under `gitkit.HermeticGitEnv()`; D3's deadline is injected, never waited out in real time.
- `PATTERN-hubforge-fixtures`: D2's tests build hubs through hubforge.
- `PATTERN-refusal-way-forward`: batten is not a bound module, but D4's refusal names its way forward anyway.
- Windows: D3's kill on Windows reaches git only, the same limit `verify-watchdog` tracks for verify; `WaitDelay` still bounds the wait.

## Auto-mode assumptions

- The board Goal ("refuses by name") governs D4 over the merged #447 note's proposal to treat a done slug as free; `lyx-cleanup` owns removing finished statuses.
- 10 minutes and 1000 B/s for 60 s are the network bounds; no current remote operation in lyx needs longer.
- 24 hours of clock-skew slack is enough for the machines the operator commits from.
- Running `gc --auto` with git's default threshold at each pair removal is cheap enough on the hub's stores; it does nothing when the store is below the threshold.
- Disabling auto-gc in the hub stores' config is acceptable for the operator's own manual git in those stores.

## Open risks

- With unreferenced loose objects kept until git's two-week expiry, the stores may stay above the gc threshold, so `gc --auto` runs an incremental repack at nearly every pair removal until those objects expire.
- A housekeeping gc at one pair's removal can still repack under another pair's live step; D1 is what makes that safe, so a go-git read that escapes D1's helper would reopen #468.
- A step attempt whose best-effort remote calls each time out waits out one deadline per call before its failing call hands back, up to 10 minutes each on SSH, and the transient re-step doubles that.
- On Windows a timed-out git's `git-remote-https` child may outlive the kill until its own connection fails.
- Operators who run `git gc` by hand get no automatic gc in hub stores after this lands; nothing tells them so except the docs.
- A done slug stays unusable until `lyx-cleanup` removes finished statuses, so the operator's #447 expectation that a done slug runs fresh is not met by this task; the refusal names a different slug as the way forward.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d1-split-lock-retry-race (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card2-headcontains-notfound-swallows-walk-failure (class consistency, severity MEDIUM, open in rounds 2)
  - card5-remove-subtest-vacuous (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- Regression for #468: build a `Repo`'s pack index, commit with the git CLI, run `git repack -d` and `git prune-packed`, then `UntrackedFiles` and `WorktreeChangedFiles` succeed and report the right paths.
- The same stale-pack setup does not break `ChangedFilesSince`, `HeadContains`, `PathRevisions` or a merge-state read: each answers correctly after the repack.
- A read for a genuinely absent object reindexes at most once while the pack set is unchanged.
- Two concurrent reads on one `Repo` over a stale pack index both succeed after the repack, and the handle is reindexed once.
- A test fails when any gitrepo production code other than the retrying helper calls `goGit()`.
- After a pair is created, both hub stores' config holds `gc.auto=0` and `maintenance.auto=false`.
- After a pair is removed from a hub whose stores lack those keys, both stores' config holds them, set before housekeeping runs.
- After a pair is removed in a store above the gc threshold, a foreground gc has run (loose objects packed) before the removal returns, and a gc failure leaves the removal successful with a logged warning.
- A remote git command against a remote that accepts the connection and never answers fails within the injected deadline, as a `*GitError` that `IsTransportFailure` reports true for, and no git process it started is left running on Unix.
- A remote git command runs with the low-speed environment set, and a failed command whose stderr carries curl's `Operation too slow` text is one `IsTransportFailure` reports true for.
- A `pull` that fails with a transport phrase is one `IsTransportFailure` reports true for.
- A local git command is never subject to the deadline or the low-speed environment.
- `lyx batten run <slug> --window` on a slug whose status is `done` exits 1 with the done-slug refusal and opens no window; on a slug with no status, or a resumable one, it opens the window as before.
- `HeadContains` keeps its existing contract table, and on a target absent from HEAD's history it stops at the committer-time bound without reading older commits.
- `PATTERN.md` holds the two new entries, and `go test ./...` and `go test -tags integration ./...` pass.

## Notes for the plan writer

- `lookupObjectRetrying` callers today: `merge.go`, `gitrepo.go`, `blobread.go`; direct `goGit()` callers include `worktree.go`, `PathRevisions` and the ref reads.
- Test fixtures already set `gc.auto=0` and `maintenance.auto=false` in `internal/gitkit` and `internal/hubforge`; D2's tests must set a positive threshold to exercise housekeeping.
- `cmd/lyx/gitrepoboundary_test.go` pins CLI-bound gitrepo methods; D1's single-caller test can sit beside it or in gitrepo.
- `internal/verifyrun/procgroup_unix.go` holds the process-group setup D3 moves to `internal/proc`.
- `runInWindow` in `internal/battencli/cli.go` has an injectable `windowOpener` for D4's test.
- New top-level tests may push a package past `cmd/lyx/testdata/test-budget.yaml`; list that file under `Edit:` when they do.
