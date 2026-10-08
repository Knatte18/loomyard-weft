# Support log — binary-change-and-rules

## Interview

Autonomous mode: no operator answered; every decision below is a self-pick recorded in the ledger.

1. Read the board entry (three parts plus a stray "Also from that PR-Gate" item) and the board.
   No other board entry plans config reconcile, stencil drift or edit-rule work; `shuttle-state` (done) added the python PreToolUse deny, which is context for Part 3.
2. Explored Part 1's landing zone: `configsync.ReconcileAll` (per-worktree) and `ReconcileHubWideAt` (hub-wide), the manual `lyx config reconcile` verb (per-worktree, no commit), `lyx fabric reconcile` (hub-wide, committed and pushed in `_board`), the root `PersistentPreRunE` in `cmd/lyx` that already runs the best-effort stencil seed on every command, `internal/lock` (flock with bounded wait), and Loom-Preflight's retired-key refusal.
   Found `fabricengine` imports `configsync`, so the hub walk (which needs pair enumeration) cannot live in `configsync`.
3. Found two build identities: `shedverbs` (VCS revision + modified) and `loomcli` (executable path, size, mtime).
   Judged path/size/mtime unsafe as the stamp key: an older process stat-ing its replaced executable reads the new file's identity.
   Judged VCS-only insufficient for dirty dev rebuilds; settled on revision + modified + config-registry fingerprint.
4. Explored Part 2: `classifyPortBackDrift` calls any source/embedded difference on an untouched board copy `source-ahead`, with no notion of order.
   Considered per-file history checks and settled on one ancestry check of the binary's revision against the worktree's HEAD.
5. Explored Part 3: the PATTERN directive reaches implementer, review-fix and designer roles but renders nothing without `PATTERN.md`, and the landing conflict resolver has no PATTERN directive.
   Settled on a dedicated, unconditional edit directive pinned by a stencil scan.
6. Wrote the decision record and this log.
7. Parent review round 1 relayed three operator calls (start verbs only, committed config, no inherited retired keys).
   Read `Fabric.Commit` (combined write lock, merge refusals, async push) and pair add (forks from the parent's local records branch) to settle the commit race and the seed order.

## Rejected alternatives

- **Stamp key = executable path, size and mtime** (as `loomcli`'s status sidecar uses): an old long-lived process resolves its executable path to the replaced file and reads the new build's identity; it also triggers on a copy that changes nothing.
- **Stamp key = VCS revision and modified flag only**: a dirty dev rebuild with changed templates keeps the same revision and would not trigger.
- **Stamp key = config fingerprint only**: misses a prod deploy that changes only a `Migrate` hook, and drifts from the operator's "a changed binary" trigger.
- **Hash the executable's bytes**: reads a large binary on every command.
- **Run a dry-run reconcile on every command instead of a stamp**: reads every worktree's config files on every command.
- **Host the hub walk in `configsync`**: import cycle with `fabricengine`, which pair enumeration needs.
- **Host it in `configcli`**: `preflightshed` would import a `*cli` package.
- **Run the check in the root pre-run of every command, with an opt-out annotation** (round 0's Decision 3/4): replaced by the operator's start-verbs-only call; it made every command stop while a reconcile failed.
- **Leave per-worktree config uncommitted for each worktree's own records commit** (round 0's Decision 7): replaced by the operator's committed-config call; it stopped `Preflight` and a Finalize merge on 2026-10-08 and let a new pair fork stale config.
- **Stop the start verb when a pair is mid-merge**: one pair's transient Finalize would block every run start in the hub.
- **Best-effort reconcile that warns and continues on failure**: the operator asked that a failed reconcile stop the start verb rather than run on a stale config.
- **Use the Part 1 stamp to silence Part 2's warning**: the stamp tells that the binary changed, not whether a worktree caught up; a pair stays behind until it syncs.
- **Per-file history check for Part 2** (is the source blob a past version of the file on the binary's history): a history walk per drifted stencil on every command, for no better advice than the ancestry check gives.
- **Put the no-script rule in the PATTERN directive stencils**: missing in repos without `PATTERN.md` and for the landing conflict resolver; mixes a tool rule into the PATTERN text.
- **Type `scribe:conversation` into spawned roles**: brings chat rules ("only answer, do not edit code", reply language) that contradict an implementer's task, and depends on a user plugin lyx does not own.
- **Restate the rule in each role stencil**: duplicated text drifts; violates `PATTERN-producer-pointer-rule`.
- **Extend the shuttle deny to sed/perl/node**: outside the board's fix direction and shuttle-state's area; text first, as asked.
- **Include the `merriamFixedContext` calibrate step**: unrelated to surviving a binary change or delivering edit rules.

## Review rounds

### Parent review — round 1

- F1 (BLOCKING, check in start verbs only): fixed. Decision 3 now runs the check only from `lyx orch start`, `lyx batten run`, `lyx loom start`/`lyx start` and `lyx loom resume`, plus `Preflight`; the root pre-run hook and the annotation's reconcile half are dropped, the stencil-seed annotation stays; Decisions 5, 6, 9 and 10 and the acceptance criteria restated for start verbs.
- F2 (BLOCKING, reconciled config ends committed): fixed. Decision 7 commits each worktree's written config on its records side through `Fabric.Commit` with a config-only file list; the race with a running driver is closed by the pair's combined write lock both take; a mid-merge pair is skipped and retried; criteria added for no uncommitted `_lyx/config`, `Preflight`'s clean check and a Finalize merge; the uncommitted-prime open risk removed.
- F3 (BLOCKING, seeded pair inherits no retired key): fixed. Decision 4 orders `lyx batten run`'s check before pair creation, and states that pair add forks from the prime's local records branch, so the committed reconcile is inherited regardless of push timing; a pair made by `lyx fabric add` before any start verb is covered by the next walk; criteria added for both.

## Question ledger

- Q1 (resolved, auto-pick): What identifies "the binary" in the stamp? — Revision + modified + config-registry fingerprint (Decision 1).
- Q2 (resolved, auto-pick): Where does the stamp live? — `<BoardDir>/.lyx/config/`, mirroring the hub-wide config dir (Decision 2).
- Q3 (resolved, auto-pick): Concurrency of two commands after a binary change? — Hub flock with bounded wait and a double-checked stamp (Decision 5); the board's open item.
- Q4 (resolved, auto-pick): Does the stamp also cover `stencil-drift-behind-main`? — No; Part 2 uses an ancestry check (Decision 13); the board's open item.
- Q5 (resolved, operator call via parent review round 1): Commit per-worktree config writes? — Yes, through `Fabric.Commit` under the pair's records lock; hub-wide files in `_board` (Decision 7). Supersedes the round-0 auto-pick of leaving them uncommitted.
- Q6 (resolved, operator call via parent review round 1): Which commands run the check? — Only the four start verbs and `Preflight` (Decision 3). Supersedes the round-0 every-command pre-run with an opt-out annotation.
- Q12 (resolved, auto-pick): A pair mid-merge during the walk? — Skipped unwritten, stamp left stale, start verb continues (Decision 7).
- Q13 (resolved, operator call via parent review round 1): Can a new pair inherit retired keys? — No; `lyx batten run` reconciles and commits the prime before creating the pair, which forks the local records branch (Decision 4).
- Q7 (resolved, auto-pick): Where does the shared reconcile live? — A new non-CLI package told its geometry (Decision 6), with a PATTERN entry.
- Q8 (resolved, auto-pick): Which roles get the edit directive? — Every spawned role that edits files, document writers included (Decision 14).
- Q9 (resolved, auto-pick): Is the `merriamFixedContext` calibrate step in scope? — No; the orch should file it as a note.
- Q10 (open, for the plan): How does a binary revision absent from the worktree's object store classify in Part 2? — Bar: never advise a deploy (Decision 12).
- Q11 (open, for the plan): The hub-lock wait bound (Decision 5).
