# Decision record — webster-audit-recovery

## Goal

A Webster run that halts on its run-exit audit or its verify gate can always be recovered by the agent that receives the refusal, without the operator and without discarding the run's batches.
Four failures from the `board-model` run (2026-10-04) close: the gate waiting forever on a hung background shell, a fork-contract-write finding that `accept-audit` can never accept, garbled way-forward text, and recoveries that name a command the agents are denied.

## Scope

In:

- Shuttle's wait loop bounds how long a turn end waits on a background shell; fork waits are unchanged.
- Every Bash command a lyx-spawned agent runs gets `/dev/null` as its default stdin, closing the hung-stdin class at its source.
- The Webster fork stencils forbid writing `outcome.yaml` and `summary.md`; Master's template says it needs no script or interpreter.
- A fork-contract-write finding becomes checkable: accepted once each contract file is absent or Master rewrote it after the fork's write.
- A new verb, `lyx webster reset`, performs in Go every branch reset a Webster refusal names, through a new gated `fabricengine` primitive.
- Every Webster refusal, `contracts/specs/refusal-spec.md` row and stencil that names a reset or move of HEAD names `lyx webster reset` instead.
- The run-exit audit and pending-findings messages state each finding once, say why an uncheckable path cannot be checked, and give an ordered, unambiguous command list.
- A new PATTERN entry and its scan test: no refusal, stencil or spec names a command the agents are denied.

Out:

- Any `python`/`python3` deny in the agents' settings (operator decision: Loomyard develops Python in other repos).
- Removing the global deny on `git reset --hard` (operator decision: it stays).
- Fabric's manual weft reset (`git -C <weft> reset --hard origin/<branch>`): it appears only in Go comments in `internal/fabricengine/pull.go` and `internal/fabriccli/weft_verbs.go`, in no refusal, stencil or spec, and fabric has no refusal-spec section; see Decision D9.
- The full deny/allow setup for every role (board entry `agent-permission-setup`).
- Killing a hung background process from lyx.

## Decisions

### D1 — Bounded wait on background shells in shuttle

Decision:
`claudeengine` reports, on each `EventWaiting`, the outstanding background tasks as a provider-neutral list on `shuttleengine.Event`: each entry has a kind (`fork` for an Agent/Task subagent, `shell` for a backgrounded Bash or a Monitor), an id and a label (the shell's command, or the Monitor's description).
Both signals feed it: `background_tasks[]` entries in the Stop payload and the transcript's unmatched launches.
Shuttle's wait loop records when it first saw each shell id outstanding.
A waiting turn end whose outstanding tasks are all shells, each either outstanding longer than the bound or already expired earlier in this run, counts as a turn end (an arrival for a gated run, otherwise the usual done/asking classification).
The check runs on every poll tick, not only on new events, because a hung shell produces no further event.
An expired shell id stays expired for the rest of the shuttle run, so a later Stop that still lists it does not restart the wait.
Any outstanding fork keeps the turn waiting, as today.
The bound is a new `shuttle.yaml` key, `background_shell_wait_min`, default 10; a non-positive value is a config error, not "unlimited".
A spawn's `RunSpec` may declare awaited shell command prefixes; a shell whose label starts with one is waited on like a fork, bounded only by the run's own timeout.
Webster's Master spawn declares `lyx webster recover-batch`, which its failure ladder backgrounds and which `recovery_timeout_min` already bounds.
Bound: the expiry lets a turn end through while non-declared shells still run, at the earliest `background_shell_wait_min` after each was first seen; it never lets a turn through while a fork, or a declared shell within the run timeout, is outstanding.

Rationale:
The operator decided the gate waits on a shell only up to a configurable bound while forks are waited on as before.
Master legitimately backgrounds `recover-batch` for up to the recovery timeout; the caller-declared prefix keeps the bound short without shuttle knowing Webster's verbs (PATTERN-shuttle-provider-seam keeps Claude payload shapes in `claudeengine`; the declared prefix is caller data, not provider knowledge).

### D2 — The driver is told which shell was waited out

Decision:
Each expiry is logged through `internal/logger` with the run dir and the shell label, and the shuttle `Result` carries the expired shells' labels.
`lyx webster run` puts one warning per expired shell on its envelope's `warnings` ("turn end counted after background shell `<label>` ran past `background_shell_wait_min`; the shell may still be running in the session"), adds a `## Background shells waited out` section to `summary.md` when the run writes one, and writes a Go-authored friction note, following the `VerifyGateNotes` precedent for flaky verify identities.
Other shuttle callers get the labels on `Result` and log them; they add no envelope field in this task.

Rationale:
The operator asked that the driver get a message naming the shell; the envelope's warnings are what a shed step trace and ly-drive read.
The friction note gives reflection the evidence the 75-minute hang left only in `events.jsonl`.

### D3 — Default stdin of every agent Bash call is `/dev/null`

Decision:
Shuttle's existing PreToolUse hook for Bash (`internal/shuttleengine/claudeengine/settings.go` and its hook handler) returns an updated tool input whose command is prefixed so the shell's default stdin is `/dev/null` (`exec </dev/null;` or an equivalent the Plan producer verifies against Claude Code's hook contract), for every lyx-spawned agent.
A heredoc, pipe or explicit redirect in the command still supplies stdin to the command it is attached to, so no legitimate use changes.
The prefix is applied once: a command that already carries it is left unchanged.
Bound: the hook only rewrites the default stdin; it denies nothing and changes no other part of the command.

Rationale:
The hung shell was `python3 -` reading the Bash tool's open stdin because its heredoc was attached to the next command.
The operator wants the source closed without denying Python; closing the default stdin closes the whole class (any interpreter, `cat`, `read`) while every intended stdin use keeps working.
The Master transcript from `board-model` ran on another machine and is not reachable here, so why Master reached for a script at all cannot be traced further than its command line; D4 covers the stencil side.

### D4 — Stencils: forks never write the contract files; Master needs no script

Decision:
`webster-body-verify-fix.md` and the batch fork prompt (`webster-prefix-fork.md` or `webster-body-implementer.md`, whichever holds the fork's rules) name `outcome.yaml` and `summary.md` by their rendered paths as Merriam's own contract files the fork never writes, creates or deletes, with the reason (a fork writing either forges the run's terminal judgment and halts the run at its exit audit).
`webster-template-master.md` states that Master reads with Read and Grep, writes its two contract files with Write, and never runs a script, an interpreter or a heredoc to read or write a file.
The existing stencil property tests gain assertions for both.

Rationale:
The fixer fork inherits Master's context, including Master's "your final action is writing outcome and summary", and the fixer stencil only says "never touch `_lyx`"; naming the two files closes the slip at its source, as the hub recommended and the operator accepted.

### D5 — A fork-contract-write finding is checkable

Decision:
One evidence function in `internal/websterengine` decides a contract-file path's checkability: the path is cleared when the file is absent, or when Master's own session transcript records a Write or Edit of that path later than the last fork write the finding names.
Every site that classifies a suspect path uses it: `AcceptPendingAudit`, `recover-batch`'s findings check, `pendingPathsWayForward` and `--fresh`'s pending-drop check.
A contract-file path that is not yet cleared gets its own way forward: delete the contract file(s), then re-run the verb (`rm <path>`, not a denied command); at run exit Master has normally already rewritten them, so `lyx webster accept-audit` clears the finding directly.
The run-exit audit still halts on a fork-contract-write finding; this task changes what clears it, not whether it halts.
Bound: only the two contract-file paths of this run use this rule; every other path under `_lyx`, `.lyx` or the scratch directory stays unverifiable, and the rule clears a finding on evidence (absence, or Master's later write), never on an acknowledgement, keeping refusal-spec's "a correctness halt clears only on evidence" line true.

Rationale:
The operator accepted the hub's recommendation: accept once Master has rewritten the files in a later turn or they are absent.
Today `checkSuspectPaths` puts every weft path in `unverifiable`, so the only way forward was a fresh run that discarded every batch.

### D6 — `lyx webster reset` performs every reset a Webster refusal names

Decision:
A new verb, `lyx webster reset --to start|pre-fix`, resets the task worktree's branch, HEAD, index and working tree to a commit the run recorded:

- `start`: the run's start commit; when the batches' recorded starts share no single oldest commit, the octopus merge-base of those recorded starts (the case `--fresh` today names `git merge-base --octopus` for).
- `pre-fix`: the verify gate's pre-fix head, which the gate now persists in `state.json` when it records it (it lives only in the gate closure's memory today); the closure's per-shuttle-run attempt semantics stay as they are.

Webster refuses before calling fabric when: the run lock is held (transient; way forward names `lyx webster status`); HEAD is detached; the checked-out branch is the parent branch or is not the task's pair branch; the target is not recorded (no start commit, no pre-fix head); the target commit is missing from the repository (the existing missing-commit way forward); the target is not an ancestor of HEAD; a git merge is in progress (`refuseMidMerge`).
The destructive step runs through a new exported method in `internal/fabricengine/destroy.go`, beside `Fabric.ResetHard`, with containment the hub, ownership a registered linked pair worktree whose branch is pair-owned and not the parent branch, dirtiness "tracked changes except a caller-supplied path set", and force always false.
Webster supplies, as that set, the tracked paths the run's own transcripts (Master's and every fork's, as the audit already parses them) record a write to; any other dirty tracked path refuses, naming each path.
Untracked files are left alone, as `git reset --hard` leaves them.
The verb accumulates a `*Mutations` record and its envelope embeds `MutationRecord` (PATTERN-mutation-record); the weft is never touched.
The verb changes no Webster state besides the reset itself; `run --fresh` or a plain `run` does the rest, as the way-forward text orders them.
Every new refusal gets a refusal-spec row and a reaching test in the same commit (PATTERN-refusal-way-forward).
Bound: the verb can discard only commits above a run-recorded commit on the task's own branch, plus uncommitted tracked changes to paths the run itself wrote; it cannot move any other branch, reset to an arbitrary SHA, touch the parent branch, the weft or untracked files, or override a refusal (no `--force`).

Rationale:
The operator decided the global deny on `git reset --hard` stays and that lyx performs every reset in Go, guarded to the run's own task branch and a recorded commit, refusing on the parent's branch and on uncommitted changes that are not the run's own.
PATTERN-fabric-git and PATTERN-fabric-destruction-chokepoint require a warp-mutating, destructive git step to live in `fabricengine/destroy.go`; the existing `Fabric.ResetHard` refuses on any tracked dirt and accepts the prime, so it cannot express the operator's guard as is.
A verb of its own keeps the destructive step explicit and serves the `pre-fix` route, which keeps every batch and needs no `--fresh`.

### D7 — Every Webster way forward that resets names the verb

Decision:
Every Webster refusal message, every refusal-spec row and every stencil sentence that says "reset the branch … with git", "move HEAD back … with git" or names `git merge-base --octopus` as a reset target names `lyx webster reset --to start` or `--to pre-fix` instead.
This covers at least the rows `rebaseline card set changed`, `rebaseline overview changed`, `plan fingerprint mismatch`, `plan changed before the verb's own rewrite`, `restore-plan: copy missing`, the audit rows, `verify gate: fixer commit rejected`, `run-exit audit correctness`, `pending audit findings`, `recover-batch: findings recovery cannot check`, `accept-audit: finding not checkable` and the three `--fresh` rows, and the matching Go sites (`auditledger.go`, `runlevel.go`, `rebaseline.go`, `recoverbatch.go`, `planbaseline.go`, `beginbatch.go`, `gitwrap.go`, `internal/webstercli/rebaseline.go`, `doc.go`).
"Restore the named paths to the last batch head with git" stays: `git checkout <head> -- <path>` and a plain file delete are not denied.

Rationale:
The operator's rule: no refusal, stencil or spec may name a command the agents are denied.

### D8 — Way-forward text: complete, each fact once, ordered commands

Decision:
The run-exit audit's stuck reason and the pending-findings refusal name each finding once with its path and drop the separate "suspect paths" list.
An uncheckable path carries its reason (for example "under `_lyx`, outside the task worktree's tracked tree", "ignored by git", "no plan copy recorded"), replacing the bare "cannot be checked" and the truncated "since nothing the run recorded can check <paths>" clause; `AcceptPendingAudit` uses the same reasons.
The way forward is a numbered, ordered command list in one clause (`way forward: 1) lyx webster reset --to start; 2) lyx webster run --fresh; …`), where each step is a distinct action and no step restates another.
The Plan producer settles the exact sequence for a shed-driven run (whether re-stepping the Webster row follows `run --fresh` or replaces it) from the code, and the spec row quotes the resulting text.

Rationale:
Issues #362 and #365: the clause read as truncated, the paths printed twice, "cannot be checked" gave no reason, and the `run --fresh` / `run` pair read as either one step or two.

### D9 — The reset verb is Webster's only

Decision:
No fabric verb is added.
Fabric's weft reset remedy stays an operator note in Go comments; no fabric refusal, stencil or spec names it.

Rationale:
The operator left the scope to this discussion, to be decided from what each module's refusals need.
Webster's refusals name resets; fabric's do not, since `weft_pulled: false` is a success envelope and the reset appears only in comments.
A weft reset to `origin/<branch>` discards local weft commits, a different guard from "a commit the run recorded", and fabric resets are the subject of board entry `fabric-reset-to-sha`.

### D10 — New PATTERN entry: no recovery names a denied command

Decision:
`PATTERN.md` gains `PATTERN-no-denied-recovery` under "CLI", beside `PATTERN-refusal-way-forward`: "Naming a way forward or a step in a refusal, stencil or spec: never a command the agents' settings deny (`git reset --hard`, `git push --force`/`-f`, `rm -rf`); lyx performs that step itself. (test)", with a `pattern/` background file.
Its test scans `contracts/specs/`, `contracts/stencils/`, `plugins/ly/skills/` and the Go string literals of the modules with a refusal-spec section (webster, shed, loom) for those command forms, with the denied set as one closed list in the test.
It lands in the same commit as D6 and D7.

Rationale:
CLAUDE.md requires a new cross-cutting invariant to go into PATTERN with its code; the operator's rule is cross-module and would regress silently without a scan.
The closed list mirrors today's global deny list; `agent-permission-setup` owns the real permission set and updates the list when it changes.

### D11 — Docs land with the code

Decision:
`internal/websterengine/doc.go` documents the reset verb, the persisted pre-fix head, the contract-file evidence rule and the expired-shell warning; `internal/shuttleengine/doc.go` (and `claudeengine`'s file comments) document the shell bound, the declared prefixes and the stdin rewrite; `internal/fabricengine/doc.go` documents the new destroy primitive; `shuttle.yaml`'s template documents `background_shell_wait_min`; `docs/overview.md` changes only if its module table names Webster's verbs.

Rationale:
CLAUDE.md's docs-in-the-same-commit rule.

## Constraints

- PATTERN-fabric-git and PATTERN-fabric-destruction-chokepoint: the reset runs only through a new primitive in `internal/fabricengine/destroy.go`, checking containment, ownership, dirtiness, force in that order; Webster runs no mutating git itself.
- PATTERN-mutation-record: the reset verb's result embeds `MutationRecord` under the fixed envelope key set.
- PATTERN-refusal-way-forward: every new or reworded Webster refusal has its refusal-spec row and reaching test in the same commit.
- PATTERN-friction-capture: the expired-shell friction note is Go-authored.
- PATTERN-shuttle-provider-seam: Stop-payload and transcript shapes, and the hook's updated-input shape, stay in `internal/shuttleengine/claudeengine`; `shuttleengine` sees only the neutral task list.
- PATTERN-completion-signal: the expiry path in `pollEventsTick` still consults `allOutputFilesExist` before any negative answer.
- PATTERN-config-strictness: `background_shell_wait_min` loads through shuttle's existing config path.
- PATTERN-cli-cobra and PATTERN-cliwire-sole-wiring: `lyx webster reset` mounts in Webster's cobra subtree and resolves geometry through `cliwire`.
- PATTERN-sandbox-coverage: the sandbox suite exercises `lyx webster reset` or the module's existing exclusion covers it.
- PATTERN-test-tier-purity, PATTERN-hermetic-git-tests, PATTERN-hubforge-fixtures: git-backed reset tests are `integration`-tagged with hubforge fixtures.
- PATTERN-sole-parsers: summary.md's new section goes through `summaryparser` if that parser owns the file's shape.
- PATTERN-stencil-ownership: stencil changes are file edits under `contracts/stencils/`, read at call time.
- The global deny on `git reset --hard` stays; no agent settings deny Python.
- Markdown in every touched `.md` uses semantic line breaks; Go comments too (PATTERN-comment-line-breaks).

## Auto-mode assumptions

- A1: The bound's default is 10 minutes; nothing in Master's loop legitimately runs a non-declared background shell, so any value well under the 75-minute hang serves.
- A2: The awaited-prefix carve-out is preferred over one long bound above `recovery_timeout_min`, which would keep a hang at an hour.
- A3: Monitor counts as a shell, not a fork; Master's template never starts one.
- A4: Closing default stdin through the PreToolUse hook is the tooling fix at the source; Claude Code's PreToolUse hook supports returning an updated tool input in the mode lyx agents run in (verified at Plan time; see Open risks).
- A5: The run-exit audit keeps halting on a fork-contract-write finding; the operator decision named `accept-audit` as the clearing step, not a downgrade at the audit.
- A6: The reset verb takes named targets (`start`, `pre-fix`) and no raw SHA, since every Webster refusal needs one of those two.
- A7: The pre-fix head is persisted in `state.json`; it is HEAD at the gate's first failed evaluation, normally the last batch head.
- A8: The deny set the PATTERN test pins is today's global deny list from `~/.claude/settings.json`.
- A9: Fabric is out of the reset verb's scope (D9).

## Open risks

- R1: Claude Code may not honour an updated Bash input from a PreToolUse hook in bypass mode, or may run the hook after its own permission step; if the Plan producer finds it unsupported, D3 falls back to the hook denying a Bash command whose interpreter reads stdin with nothing attached, which is narrower and needs shell parsing.
- R2: Stop payloads for a backgrounded Bash may lack the command text; the label then falls back to the shell id and the transcript's launch input.
- R3: A legitimate long non-declared background shell in some other shuttle caller is cut at the bound; the run then classifies as it would at a plain turn end, and the warning names the shell.
- R4: The "run's own paths" set comes from transcripts, which see Write/Edit/NotebookEdit writes only; a tracked change the run made through Bash is not in the set, so the reset refuses on it and names the path (safe direction).
- R5: Timestamps for "Master wrote later than the fork" depend on transcript entry timestamps across Master's and the fork's transcripts; clock skew is nil on one machine, but a moved run could compare across machines.

## Acceptance criteria

- A waiting turn end whose only outstanding task is a non-declared shell becomes an arrival after `background_shell_wait_min`, with no further event arriving (unit test on the wait loop with a fake clock).
- A waiting turn end with a fork outstanding stays waiting past the bound; a declared `lyx webster recover-batch` shell stays waiting past the bound.
- An expired shell listed again in a later Stop does not restart the wait.
- The expiry still runs the gate for a gated run, and still reports Done from output files per PATTERN-completion-signal.
- `claudeengine` parses a Stop payload with a `background_tasks[]` shell entry (the `board-model` shape) and a transcript-only backgrounded Bash into the neutral task list with kind and label.
- `lyx webster run` surfaces an expired shell as an envelope warning, a summary section and a friction note.
- The PreToolUse hook output for a Bash call carries the stdin-closing prefix; a command with a heredoc into `python3 -` still receives its heredoc; applying the hook twice does not double the prefix.
- The fixer and batch fork stencils name both contract files as forbidden; the Master template forbids scripts and interpreters for file reads and writes; stencil tests assert both.
- `accept-audit` accepts a fork-contract-write finding when both files are absent; accepts it when Master's transcript wrote the file after the fork; refuses, naming the delete way forward and the reason, when the fork's write is the last one and the file exists.
- `recover-batch` and `--fresh` apply the same contract-file rule.
- `lyx webster reset --to start` resets a task worktree whose branch has batch commits on top of the start commit and dirty tracked paths the run wrote; the envelope embeds a mutation record with `worktree_reset`.
- `lyx webster reset` refuses, each with a refusal-spec row and reaching test: on the parent branch, on detached HEAD, on a non-pair branch, with the run lock held, with no recorded target, with a missing target commit, with a target not an ancestor of HEAD, mid-merge, and with a dirty tracked path the run did not write.
- `--to start` resolves the octopus merge-base when recorded starts diverge; `--to pre-fix` resets to the persisted pre-fix head after a rejected fixer commit.
- The run-exit audit stuck reason for the `board-model` findings names each finding once, gives a reason per uncheckable path, and ends in a numbered command list; a golden-text test pins it, and the refusal-spec rows quote it.
- The `PATTERN-no-denied-recovery` scan passes over specs, stencils, the ly skills and webster/shed/loom string literals, and fails on a planted `git reset --hard` (test of the scan itself).
- `go test ./...` and the integration tier pass with `CGO_ENABLED=1`.

## Notes for the plan writer

- `hasOutstandingBackgroundWork` (`claudeengine/background.go`) returns a bool today; it becomes the producer of the task list, and its doc comment's "only subagent entries observed" claim is outdated by `board-model`'s shell entry.
- `pollEventsTick` (`shuttleengine/wait.go`) is where the waiting branch lives; the expiry needs state on `Run` across ticks.
- `Fabric.ResetHard` and `resetHardTo` in `destroy.go` are the model for the new primitive; `ownedPairWarpBranch` and `dirtyScopeTracked` are the nearest existing ownership and dirtiness pieces.
- The pre-fix head and attempt count live in the verify-gate closure (`websterengine/verifygate.go`); `fixCommitRejection` in `gitwrap.go` builds the "move HEAD back" text.
- `pendingPathsWayForward` and the stuck-reason builder sit in `websterengine/runlevel.go`; `AcceptPendingAudit` in `auditledger.go`; `checkSuspectPaths` in `suspect.go`.
- `plugins/ly/skills/ly-drive/SKILL.md` reads refusal-spec for escalations; check whether it names any reset.
