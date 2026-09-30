# Webster: a parent merge after a batch commit wedges record-batch

## Goal

A Webster run survives a parent merge-in that lands between a fork's commit and its `record-batch` (or `recover-batch`) call.
Today `record-batch` refuses because the report's `head_sha` no longer equals the worktree HEAD, and `begin-batch`/`recover-batch` refuse because an OK report exists, so the run is wedged until the merge commit is removed by hand (run `batten-hub-ready`, batch 11).
The fix makes the head cross-check tolerate merge-only movement, and makes the operator path that causes it (`lyx fabric merge-in`) warn while a Webster run is in flight.

## Scope

In:

- A shared head-reconciliation rule in `internal/websterengine`, used by both `RecordBatch` and `RecoverBatch`'s head cross-check.
- `RecordBatch` records the batch against the report's own `head_sha`, not the post-merge HEAD, for `CardSHAs`, the post-batch delta, and drift's triggering SHA.
- A refusal of `record-batch`/`recover-batch` while a git merge is in progress in the worktree.
- A webster-owned exported predicate answering "is a Webster run in flight at this anchor".
- A non-blocking warning from the `lyx fabric merge-in` CLI verb when that predicate is true.
- A short rule in the `ly-drive` skill (`plugins/ly/skills/ly-drive/SKILL.md`) about parent merges during the Webster phase.
- Docs in the same commit: `websterengine`'s package doc (the record-batch head cross-check), `merge-in`'s `Long` help text.

Out:

- Any change to `begin-batch`'s "report already exists" refusal or to `recover-batch`'s refusal of an OK report — both are correct once `record-batch` can consume the report.
- Any change to integration bisect or `accumulatedCardSHAs` (see Open risks).
- Any warning or guard inside `fabricengine.MergeIn` itself, so landing's own parent merge-in (which runs after Webster finishes, through the engine) is untouched.
- The non-terminal-batch resume wedge noted in the board entry; already fixed in `b16efae35` and `4ab6f6dd6`.
- `manifest/roadmap.md` — this is a bugfix.

## Decisions

### D1 — Merge-only head reconciliation

Decision: the head cross-check accepts the worktree HEAD when HEAD equals the report's `head_sha`, or when walking HEAD's first-parent chain reaches the report's `head_sha` and every commit walked before reaching it is a merge commit (two or more parents).
Any non-merge commit on that walk, or a walk that never reaches the report's `head_sha`, is refused with an error that names both SHAs and states the merge-only rule.
The rule lives in one webster helper beside `headSHA` in `gitwrap.go`, and both `RecordBatch` and `RecoverBatch` call it.
Rationale: a parent merge-in adds exactly merge commits on the task branch's first-parent line whenever HEAD has commits the parent lacks, which is the observed case, including a conflicted merge-in concluded through `lyx fabric merge --continue`;
a fork that committed past its own report still leaves a non-merge commit on the walk and is still refused.
A merge-in that fast-forwards instead (it merges with `--ff`) is refused by the same rule; see Open risks.
The walk is a read-only git query, which the Fabric Git Invariant exempts, matching how `gitwrap.go` already reads HEAD and status.

### D2 — The batch is recorded at its own commit

Decision: once D1 accepts a moved HEAD, `RecordBatch` uses the report's `head_sha` as the batch's head for `CardSHAs`, the `postBatchChecks` delta (`StartSHA`..head), and `DetectDrift`'s triggering SHA.
`DoneChecks` and drift's tree resolution still read the worktree as it stands (the merged tree).
Rationale: `CardSHAs` is the per-card bisect anchor and the delta is the batch's own work; using the merge commit would attribute the parent's changes to the batch's cards (spurious scope-guard notices, bind mismatches, drift).
The tree checks read the merged tree because that is the tree the remaining plan builds on, so a Create target or a repaired ref has to resolve there.
`RecoverBatch` already records `digest.HeadSHA`, so this makes the two verbs agree.

### D3 — Accepted merges are reported, not silent

Decision: when D1 accepts a moved HEAD, the verb adds one warning to its result naming the report's `head_sha`, the current HEAD, and the SHA of each merge commit it walked.
Rationale: the run's record should show that HEAD moved under it; the warning channel (`RecordResult.Warnings`, and `RecoverBatch`'s own warnings) already exists and never fails the batch.

### D4 — Refuse while a merge is in progress

Decision: `record-batch` and `recover-batch` refuse, before the head cross-check, when the worktree has a git merge in progress, with an error naming both routes to conclude it: `lyx fabric merge --continue` / `--abort` in a hub, and plain `git merge --continue` / `--abort` for a standalone run, which has no fabric.
The probe asks git whether `MERGE_HEAD` resolves in the repo at `WorktreeRoot`, the warp checkout, through `gitrepo` like the rest of `gitwrap.go`; it never stats a `.git/MERGE_HEAD` file, which a linked worktree keeps in its per-worktree git dir.
The weft is not probed: merge-in never merges it.
Rationale: a conflicted merge-in leaves conflict-marked files that `DoneChecks` and drift would read as the batch's result; refusing leaves the batch non-terminal and retryable once the merge concludes, which D1 then accepts.

### D5 — Webster owns the "run in flight" predicate

Decision: `websterengine` exports one predicate taking the anchor root, true when `state.json` exists under `Dir(anchorRoot)` and `outcome.yaml` is absent or records an outcome other than `done`.
Rationale: the Cwd Resolution Invariant forbids any other package from constructing webster's paths, so the reader belongs to webster; a `paused` or `stuck` outcome is a run that will resume, so it counts as in flight, while `done` means the next parent merge is landing's, not a mid-run one.

### D6 — `lyx fabric merge-in` warns, never refuses

Decision: the `merge-in` RunE in `internal/fabriccli` calls the D5 predicate with `AnchorPath()` of the `lyxcwd.Location` the verb already resolved — the value `hubgeom.WebsterGeometry` hands webster as `AnchorRoot` — so it reads the run in the worktree being merged into.
When the predicate is true, the verb adds a warning to the ok envelope of a merge that moved HEAD and to the conflict envelope, and logs a `logger.Warn`; an already-up-to-date result and a hard failure carry no warning.
The warning goes under a `warnings` string array, the key webster's own CLI envelopes use.
The merge proceeds regardless, and a predicate error degrades to no warning plus a `logger.Warn`, never a failure.
The warning says Webster is mid-run, that `record-batch` tolerates the merge, and that integration bisect over earlier card SHAs will run on pre-merge trees.
Rationale: a parent merge mid-run is sometimes needed (picking up a deployed fix was the trigger), and D1 makes it survivable, so blocking would only move the wedge; the CLI verb is the operator/hub path, while landing merges through `fabricengine` directly after Webster is done and never reaches this code.
`fabriccli` already depends on `websterengine` transitively, so the import creates no cycle.

### D7 — `ly-drive` gets one rule

Decision: `plugins/ly/skills/ly-drive/SKILL.md` gains one short rule: prefer not to merge the parent into a task worktree while its Webster phase is in flight, and when a parent fix is needed mid-run, `lyx fabric merge-in` is survivable and will warn.
Rationale: the hub repairing a stopped run is the actor that did this; the skill is where that actor's repair guidance lives.

## Constraints

- **Fabric Git Invariant:** the D1 walk and D4 probe are read-only git queries; no git mutation is added anywhere.
- **Cwd Resolution / Told-Geometry:** the D5 predicate takes an anchor root it is handed and builds paths only through webster's own `Dir`/`OutcomePath` helpers; `websterengine` still imports no `lyxcwd`.
- **Hermetic Git Test Environment Invariant** for every new git-backed test.
- **hubforge Fabric-Fixture Invariant** for the `fabriccli` merge-in test fixture.
- **CLI/Cobra Invariant:** `merge-in` keeps its `Short`; its `Long` text documents the warning.
- **Documentation Lifecycle / task-completion rule:** docs named in Scope land in the same commit; no new cross-cutting invariant, so `CONSTRAINTS.md` is unchanged.
- Existing `record-batch` behaviour when HEAD equals `head_sha` is byte-for-byte unchanged, including result warnings.

## Auto-mode assumptions

- "loom/fabric warn" in the board entry is satisfied by the `fabric merge-in` CLI warning plus the `ly-drive` rule; no loom driver or `lyx loom status` change.
- Any commit with two or more parents counts as a merge for D1; the second parent's origin (the recorded parent branch or not) is not checked.
- Mid-run detection reads only webster's own durable files, not loom's shed status.

## Open risks

- Integration bisect walks `CardSHAs` across the merge: a card recorded before the merge is verified on a tree without the parent's changes, so a regression the merge introduced (or fixed) can be attributed to the wrong card. D6's warning names this; the bisect itself is unchanged.
- A fork that runs `git merge` after writing its report is accepted by D1, and nothing else stops it: the fork audit allows repo-native git. D3's warning, which names every walked merge SHA, is the only trace.
- A merge-in fast-forwards when the report's `head_sha` is an ancestor of the parent's tip — the task's warp has no commits the parent lacks and the batch's fork committed nothing. HEAD then moves by the parent's ordinary commits, D1 refuses, and the run wedges as today; recovery is resetting the warp to `head_sha` before `record-batch`.
- A symbol the parent renamed or deleted falls in no batch's delta (this batch's ends at `head_sha`, the next starts at the merge commit), so drift never auto-repairs a remaining-plan reference to it; the next `begin-batch`'s dispatch validation reports it as a blocking finding instead.
- A Webster run started standalone (no fabric) never sees D6's warning, since `lyx fabric` is hub-only; D1 still protects it.

## Acceptance criteria

- `record-batch` with HEAD == report `head_sha` behaves exactly as before (existing tests pass unchanged).
- `record-batch` after one parent merge commit on top of the fork's commit succeeds: the batch goes terminal, `CardSHAs` holds the report's `head_sha`, and the result carries one warning naming both SHAs and the merge SHA.
- Same with two successive merge commits, and the one warning names both merge SHAs.
- A fast-forward from the report's `head_sha` onto parent commits that are not merges is refused like any non-merge movement.
- A file added only on the merged-in parent side does not appear in the batch's delta: no scope-guard notice, no bind mismatch, no drift finding caused by it.
- A non-merge commit after the report's `head_sha` (with or without merges around it) is refused, and the error states the merge-only rule and names both SHAs; the batch stays non-terminal.
- A report `head_sha` that is not on HEAD's first-parent chain is refused the same way.
- With a merge in progress (`MERGE_HEAD` present), `record-batch` refuses with the `merge --continue`/`--abort` pointer and leaves the batch non-terminal; after the merge concludes, `record-batch` succeeds.
- `recover-batch`'s head cross-check follows the same D1 and D4 rules (accepted merge → terminal with the report's head recorded; non-merge commit → refused).
- End-to-end wedge scenario in `websterengine` tests: begin-batch, fork commit + OK report, parent merge, record-batch succeeds, next begin-batch succeeds with `StartSHA` at the merge commit.
- The D5 predicate: no `state.json` → false; `state.json` and no `outcome.yaml` → true; outcome `paused` → true; `stuck` → true; `done` → false; unreadable/malformed `outcome.yaml` → error.
- `lyx fabric merge-in` in a hub fixture with an in-flight Webster state: the merge completes and the envelope carries the warning; a conflicting merge-in's conflict envelope carries it too; without Webster state, or with outcome `done`, no warning; conflict and failure exit behaviour unchanged.
- `lyx fabric merge-in` with a malformed `outcome.yaml`: the merge completes and the envelope carries no warning.
- `record-batch`'s merge-in-progress probe fires in a linked git worktree.
- `ly-drive` SKILL.md contains the D7 rule; `merge-in`'s `Long` help documents the warning; `websterengine`'s package doc describes the merge-only head rule.

## Notes for the plan writer

- Wedge sites: `RecordBatch`'s head cross-check in `internal/websterengine/recordbatch.go` and the matching block in `recoverbatch.go` (search "Cross-check report's head_sha").
- `postBatchChecks` already takes `HeadSHA` as an input; D2 is a change to what `RecordBatch` passes, not to `postBatchChecks`.
- The merge-in verb is in `internal/fabriccli/merge_verbs.go`, envelope handling in `setMergeExit`.
- Outcome parsing is `parseOutcome` in `outcome.go`; `OutcomePath` is in `runlevel.go`.
