# Decision record — binary-change-and-rules

## Goal

A hub keeps working across a `lyx` binary change without a manual `lyx config reconcile --apply` in the prime and every pair, with the reconciled config committed and no new pair inheriting a retired key, a pair behind main stops asking for a production deploy, and every spawned agent that edits files is told the no-script edit rule.

## Scope

In:

- Part 1: a hub-level binary stamp; the first start verb (`lyx orch start`, `lyx batten run`, `lyx loom start`, `lyx loom resume`) under a changed binary reconciles the hub's config (hub-wide modules, the prime, every pair) once, under a hub lock, before its own work, and commits what it wrote; the recipe's `Preflight` row checks the same stamp as a safety net.
- Part 2 (issue #445): the stencil port-back drift warning stops classifying a worktree that lacks the binary's build commit as `source-ahead`.
- Part 3: one shared edit directive stating the no-script edit rule, rendered into every spawned role stencil whose agent edits files.
- Consolidating the duplicated build-identity readers that Part 1 would otherwise copy a third time.
- `PATTERN.md` entries for the two new invariants (Decisions 6 and 14), the refusal-spec row for the `Preflight` safety net, package docs.

Out:

- The `merriamFixedContext` calibrate step in `tools/tokencount` listed under "Also from that PR-Gate" in the board body: it serves Merriam's token budget, not this task's goal; the orch should file it as its own note.
- Extending the shuttle PreToolUse deny beyond python (sed, perl, node): the board names stencil text as the fix direction, and the python deny is shuttle-state's.
- Changing `lyx config reconcile` (the manual verb): it keeps its per-worktree behavior and does not write the hub stamp.
- Running the check from any other command: ordinary commands (`lyx board`, `lyx config`, hooks, key presses) never reconcile.
- Other `scribe:conversation` rules (tone, reply length, `.scratch/` files): they govern chat or conflict with Go test conventions (`t.TempDir`), and are not edit rules.

## Decisions

### Part 1 — config reconciles itself after a binary change

1. **The stamp key is a build key: the binary's VCS revision, its modified flag, and a fingerprint of the config registry.**
   The fingerprint hashes every registered config module's name, flags (`HubWide`, `SeedOnly`, `OpenMaps`) and embedded template.
   Rationale: the revision makes every prod deploy trigger, as the operator asked; the fingerprint catches a dirty dev rebuild whose revision did not move but whose templates did.
   The executable's path, size and mtime are not used: an older long-lived process stat-ing its replaced executable would read the new file's identity and mistake itself for the new build.
   Bound: a dirty dev rebuild that changes only a `Migrate` hook, with no template change, does not trigger; `lyx config reconcile --apply` remains the manual route for that case.

2. **The stamp is one never-tracked file under the hub's board dir, at the `.lyx` mirror of the hub-wide config dir (`<BoardDir>/.lyx/config/`), with the hub lock file beside it.**
   It records the build key, plus the revision and modified flag in readable form for the log.
   Rationale: the hub-wide config already lives at `<BoardDir>/_lyx/config/` (`PATTERN-hub-wide-config`), and a stamp that is never tracked belongs at the mirrored `.lyx` subpath (`PATTERN-durable-vs-ephemeral-state`); only `internal/lyxdirs` spells the literal (`PATTERN-lyxdirs-single-declarer`).
   An absent stamp counts as stale, so a hub's first start verb under this feature reconciles once.

3. **The check runs only in the start verbs that launch long-lived work: `lyx orch start`, `lyx batten run`, `lyx loom start` (and its root `lyx start` alias) and `lyx loom resume`, each calling Decision 6's entry point before any of its own work, and only when the cwd resolves into a hub.**
   No other command runs it; the root `PersistentPreRunE` of `cmd/lyx` is unchanged.
   A fresh stamp costs one file read and no git.
   Rationale (operator, 2026-10-08): one reconcile at a start verb covers the prime and every pair, and `lyx orch start` reaching it first means the orch runs on the new config; ordinary commands stay unaffected, so a failing reconcile never stops the orch's own `lyx` calls.
   Standalone repos (no hub) are untouched.

4. **`lyx batten run` runs the check before it creates or adopts the run's pair.**
   Rationale: `fabricengine`'s pair add forks the new pair's records branch from the prime's local records branch, so once Decision 7 has committed the prime's reconciled config, the new pair inherits it with no retired key; the async push of that commit does not matter, since the fork reads the local branch.
   A pair created another way (a direct `lyx fabric add`) before any start verb ran under the new binary is still covered: the next start verb's walk reconciles and commits it like any other pair.
   The stencil-seed skip annotation stays as it is.

5. **A stale stamp takes the hub lock with a bounded wait, re-reads the stamp, and reconciles only if it is still stale (double-checked).**
   Order: hub-wide modules at the board dir (`configsync.ReconcileHubWideAt`, seeded from the prime as `lyx fabric reconcile` does), then the fabric config is loaded and the pairs enumerated, then `configsync.ReconcileAll` and Decision 7's commit at the prime's anchor and at every pair's anchor.
   The stamp is written only after every worktree reconciled and committed, then the lock is released.
   Rationale: two commands starting together after a binary change produce exactly one reconcile; the second waits, sees the fresh stamp and proceeds.
   The lock is an OS file lock (`internal/lock`), so a crashed holder never leaves it stale.
   A lock still held when the wait runs out stops the start verb with a way forward to re-run it; the plan picks a wait comfortably above a hub-wide reconcile's duration.

6. **The hub-wide reconcile lives in one new non-CLI package that both callers use; it is told the hub's geometry and derives no path.**
   Rationale: the start verbs and the `Preflight` row must run the same function (the same reason as `PATTERN-gate-self-check-parity`), `internal/configsync` cannot host it because `internal/fabricengine` imports `configsync` and pair enumeration needs `fabricengine`, and a shed producer must not import a `*cli` package.
   New `PATTERN.md` entry, in the same commit: reconciling a hub's config after a binary change happens only through this package's one entry point, which owns the stamp and the hub lock.

7. **Every reconciled file ends committed.**
   Hub-wide files are committed in `_board` under the board write lock and pushed, as `lyx fabric reconcile` does; a push failure is logged and never fatal.
   Each worktree's written config files (the prime's and every pair's) are committed on that worktree's records side through `fabricengine`'s `Fabric.Commit`, as `lyx fabric commit` does, with a file list holding only the config files that reconcile wrote or removed.
   Rationale (operator, 2026-10-08): a manual reconcile's uncommitted `_lyx/config` changes stopped `Preflight`'s worktree-clean check and a Finalize merge, and let a new pair fork the prime's stale committed config.
   Race with a running pair's driver: `Fabric.Commit` takes the pair's combined write lock, the same lock the driver's own records commits take, so the two serialize; the scoped file list never sweeps the driver's in-flight files, and a driver commit that lands between the write and this commit takes the config change with it, after which this commit is a no-op.
   `Fabric.Commit`'s async push follows its existing rule: records only in the prime, both sides in a pair (`PATTERN-push-both-sides`).
   A worktree whose records side refuses the commit because a merge is in progress (`ErrMergeInProgress`, foreign merge state) is not written at all: it is skipped, logged, and the stamp stays stale so the next start verb retries it; the start verb continues, since a merge is transient.

8. **Every applied change is logged through `internal/logger`: one line per changed module per worktree (worktree, module, added, removed, migrated), and one summary line naming the previous and new revision.**
   Nothing goes to stdout, which carries the command's own JSON envelope.

9. **A reconcile or commit that fails stops the start verb with a JSON error envelope, and the stamp stays stale.**
   The message names the worktree and file that failed, the error, and the way forward: fix the named file (or run `lyx config <module>` on it), then re-run the start verb.
   Worktrees that reconciled and committed cleanly before the failure keep their commits, since each per-worktree reconcile is idempotent; the next start verb retries the whole walk.
   A pair that vanishes mid-walk (torn down concurrently) is skipped, not a failure; a pair mid-merge is skipped as Decision 7 states.
   Bound: while the stamp stays stale and failing, only the start verbs stop; every other command, `lyx config` and the orch's own calls included, runs as today.

10. **The loom recipe's `Preflight` row (`internal/preflightshed`) checks the same stamp and calls the same entry point when it is stale.**
    A failure is `Stuck` with Decision 9's way forward, and gets a row in the shed section of `contracts/specs/refusal-spec.md` in the same commit, with a test that reaches it (`PATTERN-refusal-way-forward`).
    Rationale: the operator asked for it as a safety net for a run whose start verb predates the binary change, or that reaches `Preflight` without a start verb (a `goto` re-entry, a driver started before the binary changed).
    The stamp check runs before the row's worktree-clean check, so the committed reconcile leaves the tree clean for it.
    Loom-Preflight's retired-key refusal and its `lyx config reconcile --apply` way forward stay unchanged.

11. **The two existing build-identity readers are consolidated: the VCS reader in `internal/shedverbs` (`runningBuildIdentity`) moves to one shared home that `shedverbs` and the new package both import; `internal/buildinfo` stays import-free.**
    Rationale: Decision 1 needs the same VCS reading, and a third copy is duplication.
    `internal/loomcli`'s path/size/mtime identity serves a different question (is the status strand's launching file stale) and stays where it is.

### Part 2 — stencil drift behind main

12. **A drifted stencil whose board copy is untouched and whose worktree source differs from the embedded bytes is classified `source-ahead` only when the binary's VCS revision is an ancestor of the worktree's HEAD.**
    When the revision is not in HEAD's history, the worktree is behind the binary's build: the class is `behind`, logged at info level, with a message that the worktree is behind the build that deployed the board copy and that syncing with main resolves it, and no deploy advice.
    When the binary carries no VCS revision, today's classification stands.
    Rationale: a worktree that holds every commit the binary was built from can only differ through its own edit, which is the case a deploy fixes; one that lacks the build commit differs at least partly through main's later changes, and a deploy fixes nothing (issue #445).
    A diverged pair that also edited the stencil itself is classified `behind` too, which is still the right advice: its edit reaches production only after it lands.
    The hand-edited, both and neither classes are unchanged.
    The plan settles how a revision absent from the local object store classifies, with "do not tell the operator to deploy" as the bar.

13. **The ancestry check runs in `internal/fabricengine` (`PATTERN-fabric-git`), is computed by the stencil-seed caller only when a drift comparison needs it, and reaches `internal/stencilstore` as an input; `stencilstore` runs no git.**
    Rationale: the seed pass runs before every command, so a worktree with no drift must pay no git cost.
    The stamp of Part 1 is not used here: it says the binary changed, not whether a given worktree has caught up, and a pair stays behind until it syncs regardless of when the binary changed.

### Part 3 — the no-script edit rule reaches spawned agents

14. **One shared edit-directive stencil states the rule, filled into a new marker in every spawned role stencil whose agent edits files, by one Go renderer.**
    The rule: change files only with Edit or Write, never with a script that rewrites them (not `sed`, `awk`, `perl`, or an inline `python`/`node` heredoc), bulk changes included (one Edit per site, or `replace_all` for an identical string); never use `sed` at all, even to read; search and read with `grep`, `awk` or `cat`.
    The wording stays consistent with the shuttle python deny's steer.
    The directive renders unconditionally, unlike the PATTERN directive, which renders nothing in a repo without `PATTERN.md`.
    Roles that get it: every spawned role that edits files, its own artifacts included — webster's implementer fork, recovery strand and verify-gate fixer, Webster's Master (it writes state files), the Burler fixer, Rework, the landing conflict resolver, Plan-Write and Discussion-Write; review-only and judge roles do not.
    New `PATTERN.md` entry, in the same commit, enforced by a stencil test like the parent-directive one: a file-editing role's opening stencil carries the edit-directive marker, and no other stencil restates the rule (`PATTERN-producer-pointer-rule`).
    Rationale: the rule lives in `scribe:conversation`, a chat skill spawned roles never load; adding the rule to the PATTERN directive would miss consumer repos without `PATTERN.md` and the landing conflict resolver, which has no PATTERN directive; typing `scribe:conversation` into spawned roles would also load chat rules ("only answer, do not edit code") that contradict an implementer's task.

## Constraints

- `PATTERN-cwd-resolution` and `PATTERN-told-geometry`: the start verbs and `preflightshed` resolve geometry; the new reconcile package is told the hub, board dir, prime anchor and pair anchors and does not import `internal/lyxcwd`.
- `PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`: the stamp and lock paths are built through `internal/lyxdirs` helpers at the `.lyx` mirror of the hub-wide config dir.
- `PATTERN-hub-wide-config`: hub-wide modules are reconciled only at `<BoardDir>/_lyx/config/`; per-worktree copies are retired or reported divergent exactly as `configsync.ReconcileAll` already does.
- `PATTERN-config-strictness`: no new config loader; the reconcile uses `configsync` and the fabric config's existing strict load.
- `PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`, `PATTERN-gitexec-checked-call`: the hub-wide commit and push go through `fabricengine` (`Bolt`), each worktree's records commit through `Fabric.Commit`; the Part 2 ancestry check is a `fabricengine` local read.
- `PATTERN-push-both-sides`, `PATTERN-mutation-record`: per-worktree commits use `Fabric.Commit` unchanged, inheriting its push rule and mutation record; no new push path.
- `PATTERN-never-force-add`: the commit's file list holds tracked `_lyx/config` paths only.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: the `Preflight` stuck row names its way forward, gets its spec row and a reaching test; no way forward names a denied command.
- `PATTERN-cli-cobra`: a start verb's reconcile failure is a JSON error envelope.
- `PATTERN-spawn-observability`: no new OS process is started; the ancestry check is in-process.
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`, `PATTERN-parent-directive`: the edit directive is read through `internal/stencilstore` from the told stencils dir and is the rule's only statement.
- `PATTERN-role-skills-typed`: no stencil asks an agent to load a skill; Decision 14 adds text, not a skill.
- `PATTERN-leaf-packages`: `internal/buildinfo` keeps its closed (empty) import set.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: a hub-wide walk test uses `internal/hubforge`, in the lowest tier its substrate needs.
- `PATTERN-test-economy`: extend the existing stencilstore drift tests and the role-stencil scans rather than adding parallel suites.
- Docs in the same commit: the new package's `doc.go`, `internal/configsync`'s and `internal/stencilstore`'s docs where behavior changes, `docs/overview.md` if its module table lists the new package, and `PATTERN.md` for Decisions 6 and 14.
- `PATTERN-comment-line-breaks`: semantic line breaks in Go comments.

## Auto-mode assumptions

- Auto-pick: the build key combines revision, modified flag and a config-registry fingerprint (Decision 1); the operator said "the binary that last reconciled", and the key still triggers on every prod binary change.
- Operator call (relayed by the parent review, 2026-10-08): the check runs only in the start verbs and `Preflight` (Decision 3); reconciled config ends committed (Decision 7); a new pair never inherits retired keys (Decision 4).
- Auto-pick: a worktree mid-merge is skipped and retried at the next start verb rather than stopping it (Decision 7).
- Auto-pick: Part 2 does not use the Part 1 stamp (Decision 13), which settles the board's open question.
- Auto-pick: the `merriamFixedContext` calibrate step is out of scope (Scope).
- Auto-pick: the edit directive reaches document writers (Plan-Write, Discussion-Write) as well as code editors (Decision 14).
- Assumption: deployed `lyx` binaries carry a VCS stamp, since `tools/deploy` runs `go build` inside the repo; an unstamped binary contributes an empty revision and relies on the fingerprint alone.

## Open risks

- Downgrade flip-flop: running an older binary in the hub after a newer one reconciles back to the older templates, so a key the older template lacks is removed with its value, and the newer binary later re-adds it with its template default.
  This follows from "the config matches the binary that runs"; the operator's real hub normally sees only the prod binary.
  The same applies to a long-lived process of an older build reaching the `Preflight` safety net after a newer build stamped the hub.
- The first start verb after a binary change is slower, by one read (and possibly a write and records commit) of every worktree's config plus one `_board` commit and push.
- While a reconcile keeps failing, no run can start or resume in the hub (Decision 9); the operator repairs the named file with `lyx config` or an editor.
- A records commit into a running pair lands between its driver's own transition commits; it carries only config files, but it shows up in that pair's records history as a lyx-authored commit.
- A pair that stays mid-merge keeps the stamp stale, so every start verb repeats the hub walk until that merge ends.

## Acceptance criteria

- A hub whose stamp names a different build key: each of `lyx orch start`, `lyx batten run`, `lyx loom start` and `lyx loom resume`, run first, reconciles the hub-wide modules, the prime and every pair before its own work, logs each migrated module, and writes the stamp; the next start verb does not reconcile again.
- Any other `lyx` command under a changed binary performs no reconcile and leaves the stamp untouched.
- A retired key (as `master_base` was) in a pair's `batcher.yaml` is removed by that first start verb without any manual `lyx config reconcile --apply`, and Loom-Preflight then passes.
- After the reconcile no worktree (prime or pair) has uncommitted `_lyx/config` changes: `Preflight`'s worktree-clean check and a Finalize merge both pass.
- A pair created by `lyx batten run` after a binary change carries no retired key in any `_lyx/config` file, and its Loom-Preflight passes without a manual reconcile.
- A pair created by `lyx fabric add` before any start verb ran under the new binary is reconciled and committed by the next start verb's walk.
- A reconcile commit into a pair whose driver commits concurrently serializes on the pair's records lock and loses neither side's change.
- A pair mid-merge is skipped without being written, the start verb proceeds, and the stamp stays stale until a later start verb reconciles that pair.
- An absent stamp is treated as stale.
- Two start verbs run together under a new binary produce exactly one reconcile; the second waits on the lock and proceeds on the fresh stamp.
- A pair's unparseable config file stops the start verb with a JSON error naming that file and the way forward; the stamp stays stale; ordinary commands, `lyx config` included, still run; after the file is fixed, the next start verb completes the reconcile and stamps.
- A pair removed while the walk runs is skipped without failing the reconcile.
- Hub-wide config changes are committed in `_board`; a push failure is logged and does not stop the start verb.
- The `Preflight` row on a stale stamp runs the same reconcile before its worktree-clean check; on a reconcile failure it is `Stuck` with the way forward, and that refusal has its spec row.
- A dirty dev rebuild with a changed config template triggers the reconcile; a rebuild with no config change and the same revision does not.
- Stencil drift: a worktree whose HEAD lacks the binary's revision and whose source differs from the embedded bytes logs class `behind` at info level with no deploy advice; a worktree whose HEAD contains the revision and whose source differs logs `source-ahead` with deploy advice as today; a binary with no revision keeps today's classification; a worktree with no drift runs no git.
- Every file-editing role's rendered prompt contains the edit directive's rule, including in a repo with no `PATTERN.md`; a stencil scan fails when a file-editing role's opening stencil lacks the marker or another stencil restates the rule.
- `shedverbs` step records still carry the same `vcs_revision`/`vcs_modified` fields after the build-identity reader moves.

## Notes for the plan writer

- Precedents to read: `internal/fabriccli/fabric.go` `runReconcile` (hub-wide reconcile seeded from the prime, `Bolt.CommitWritten`, push), `internal/configcli/configcli.go` `runReconcile` (per-worktree reconcile and result reporting), `internal/fabriccli/fabric.go` `runPairs` (pair enumeration through `Topology.Status`).
- The fabric config must be loaded after the hub-wide reconcile, since enumerating pairs reads it.
- Per-worktree commit: `internal/fabricengine/commit.go` `Fabric.Commit` (combined write lock, merge-in-progress refusals, async push); pair seeding: `internal/fabricengine/add.go` (forks from the parent's local records branch).
- Start verbs: `internal/orchcli/start.go`, `internal/loomcli/start.go`, `internal/loomcli/resume.go`, and the batten run verb in `internal/battencli`.
- `internal/stencilstore/reconcile.go` `classifyPortBackDrift` and `warnPortBackDrift` hold the drift classes; `internal/shedverbs/buildidentity.go` holds the VCS reader to move.
- Directive precedents: `internal/parentdirective`, `internal/pattern`, `contracts/stencils/parentdirective_test.go`.
- The python deny's steer text is in `internal/shuttleengine/claudeengine/settings.go`.
