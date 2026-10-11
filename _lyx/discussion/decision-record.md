## Goal

Close six Webster faults that stop or mislead runs and sit outside the audit work: a Master turn end that discards a running gate shell, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, a recovery prompt that does not name its allowed gates, a Create check that cannot say why it fails, and build-tagged or per-GOOS members that resolve `ambiguous` and so can never pass.
The code under design is base 800a0e892; commit 1e79825ff and everything after it counts as not landed.

## Scope

In scope:

- Shuttle's wait and Master's spawn spec: which background shells hold Master's turn end.
- The gate re-prompt text for the verify gate entry, and Master's stencil where it reads a verify failure.
- Recovery write attribution (`loadRunWrites` and the batch record), the recover-batch exit code, and the recovery prefix stencil.
- `internal/planglyph`: the done-check finding detail, and one shared build-constraint classification used by the done-checks and by the plan-time status policy.
- The docs and stencil claim pins these touch.

Out of scope:

- Changing quarry (an external module, `github.com/Knatte18/quarry` v0.2.1).
- A card that deletes every declaration of a name and creates it again (the `create-already-exists` half of #507): the plan gate resolves the tree before any card runs.
- Reviving a dead batch through `lyx loom goto` (the third ask in #511).
- Fork-side gate waiting and notice noise (board entry `fork-wait-noise`).
- Killing shells from lyx: lyx still stops no background shell.

## Decisions

1. **A running `lyx gate test` shell holds Master's turn end.**
   Decision: Master's spawn declares a list of awaited shell prefixes, `lyx webster recover-batch` and `lyx gate test`, replacing the single `masterAwaitedShellPrefix` constant.
   Shuttle's existing awaited-prefix rule then holds the turn end on such a shell like a fork, so `countsAtOnce` is false and the run does not finish while a gate tier runs.
   Rationale: the wait already never expires a shell, but an unawaited shell with every output file present lets the gated run finish and strand removal kills the shell, which is the discarded integration tier of #505, #509 and #517.
   Awaiting by prefix reuses the mechanism `recover-batch` already uses and adds no new expiry or kill path.
   Bound: the hold ends only on the shell finishing, `master_timeout_min`, or the liveness check; at the deadline `classifyDeadlineExpiry` still finalizes Done when the files exist, with the shell on `Result.EndedShells`.
   A shell whose label does not start with an awaited prefix, such as `cd x && lyx gate test`, is not held; Master's stencil says to start a gate run as a plain `lyx gate test ...` Bash call, with no `cd` and no wrapper.

2. **Every ended-shell record says the shell's result was not read.**
   Decision: `expiredShellWarning`, and with it the friction note line, the `RunResult.Warnings` entry and the summary section line, states that the shell's result was never read by the run.
   Rationale: the record already exists on every outcome; the missing half of "records that its result was discarded" is the explicit clause, so no reader takes a `done` as that shell's verdict.

3. **The verify gate's re-prompt tells Master to spawn the fixer fork, not to fix.**
   Decision: `shuttleengine.GateEntry` gains an optional one-line `RepromptNote`; when non-empty it replaces the generic "read it, fix every finding, and end your turn" tail of `gateRepromptText`, and the fixed `Gate findings recorded at <path> — ` head stays.
   Webster's verify entry sets it to: read the findings, edit and commit nothing yourself, spawn the one fixer fork your prompt describes, and end your turn.
   Rationale: the generic text is what Master reads at the point of failure, and it tells Master to fix every finding, which contradicts the stencil's rule that sits earlier in a long prompt.
   Putting the rule in the message Master acts on makes the right path the easy one.
   Bound: the note changes only the text sent for the entry that sets it; no other gate's re-prompt, budget or verdict changes, and the text must stay single-line (`validateSendText`).
   Master's stencil section "A gate failure: spawn one fixer fork" keeps its rule and names the message as saying the same.

4. **Recovery writes count as the batch's own writes.**
   Decision: each batch record carries the session ids of its recovery strands, appended at `recoverSpawn` from the spawned run's session id and carried across respawns like `ForkTranscripts`.
   `recordedSessions` includes them, so `loadRunWrites` returns a recovery strand's own writes and its forks' writes with the Master and fork writes.
   Rationale: attribution by batch, not by shuttle run id (#522): a session of the run is any session the state records, and the first recovery's session was never recorded, so its 19 files read as "Not written by this run".
   One reader feeds three consumers, so the uncommitted-path grouping, a reset's own-path set and the contract-file evidence all gain the recovery writes.
   Bound: only a successful Write, Edit or NotebookEdit by a recorded session is evidence; a failed write is not; a `state.json` without the field reads as no recovery sessions, which is today's behavior; an unattributable path stays "Not written by this run".
   What it lets through: a second recovery keeps and may commit the first recovery's uncommitted paths after checking them against the card, and a reset's foreign-dirty refusal no longer fires on a recovery's own uncommitted tracked paths.

5. **A terminal recover-batch digest that is not `done` exits non-zero.**
   Decision: `recover-batch` returns the digest fields with `ok: false`, an error message naming the batch and its status, and a non-zero exit for a terminal `stuck` or `dead` digest; a `done` digest and the non-terminal `running` snapshot keep exit 0.
   Rationale: #522 saw exit 0 while the batch ended failed and terminal; Master and the orch read the exit code of a backgrounded call first.
   Bound: only the exit code, the `ok` flag and the added message change; every digest field Master's rungs read (`status`, `dead_reason`, `recovery_retry`, `recoveries`) keeps its name and value, and Master's stencil says the notification of a non-done terminal digest carries a non-zero exit.

6. **The recovery prompt names its always-allowed commands.**
   Decision: `webster-prefix-recovery.md` gains a short section after the sandbox rule: the card gates and the batch gate as rendered (`lyx gate test`, including `--tags integration`) and `lyx loom lint-comments` are always allowed and are not fabric commands; the ban covers mutating `lyx fabric` verbs only; a card step tagged `tmux` is not run and is left to the plan's verify and to Publish.
   Rationale: the recovery in #522 could not tell whether the gates fell under the fabric ban and could only compile-check the tmux step.
   The section lives in the recovery prefix because a fork inherits Master's context and a recovery starts cold.
   A `contracts/stencils/claims_test.go` row pins the section.

7. **The Create done-check says how it looked the target up.**
   Decision: the `create-not-done` detail, and the sibling delete and rename details, name the resolve key looked up, quarry's status for it, the unit status when the member is missing, and each ambiguous candidate's file.
   A test pins that an unexported identifier and a member declared in a `_test.go` file both resolve.
   Rationale: #511 asks to tell a missing symbol from a resolver limit; quarry v0.2.1 resolves unexported identifiers and test-file members, so the fix is the diagnostic, with the pin keeping the resolver honest.
   Bound: finding ids, severities and the pass/block rules do not change in this decision.

8. **A candidate set partitioned by build constraints counts as one declaration.**
   Decision: `internal/planglyph` gains one classifier over an `ambiguous` answer's candidate files, read under the worktree root: the answer is partitioned when its candidates sit in pairwise distinct files whose build constraints can never hold together.
   Constraints come from each file's `//go:build` line and its `_GOOS`, `_GOARCH` filename suffix, evaluated with `go/build/constraint` over every assignment of the tags involved, with the GOOS names mutually exclusive, the GOARCH names mutually exclusive and `unix` derived from GOOS.
   A partitioned answer is treated as resolved, like `multipart`: `DoneChecks` passes a Create target and a Rename new side on it, and the plan-time status policy (`statusFindings`) raises no `glyph-ambiguous` for it, so an Edit or Delete by glyph of a per-GOOS or per-tag member (#500, #507) passes the plan gate.
   A Delete target and a Rename old side on a partitioned answer still block, because declarations remain; `createFindings` still blocks a Create whose name already exists.
   Rationale: quarry v0.2.1 ignores build constraints and cannot be changed here, and the same classification at the plan gate and at record-batch keeps the two from disagreeing; a glyph that stays ambiguous after this is flagged at Plan time as it is today, and so no ambiguity of this class first appears at record-batch.
   Bound: this skips only the ambiguity block, only for Go files whose constraints are provably disjoint; two candidates in one file, overlapping constraints, an unreadable file or an unparsable constraint stay `ambiguous` and block (fail closed).

9. **Docs and pins land in the same commit.**
   `internal/shuttleengine/doc.go` (gate re-prompt note, awaited shells), `internal/websterengine/doc.go` (the background-shell section, recovery write evidence, the recovery prompt section), `internal/webstercli` recover-batch help, `internal/planglyph/doc.go` (done-check detail, partitioned answers), Master's stencil and `contracts/stencils/claims_test.go` change with the code.
   No new PATTERN entry: each change stays inside an existing seam.

## Constraints

- `PATTERN-shuttle-provider-seam`: the re-prompt note and awaited prefixes are provider-neutral; nothing Claude-specific leaves `claudeengine`.
- `PATTERN-completion-signal`: no new negative-answer exit; the hold only keeps the existing paths and `classifyDeadlineExpiry` still consults the output files.
- `PATTERN-no-denied-recovery`: the new recovery text names no denied form and offers no raw module-wide `go` run; the way forward for a tmux step is "left to verify".
- `PATTERN-refusal-way-forward` and `PATTERN-stencil-ownership`: the non-zero recover-batch envelope names the way forward Master's rungs already give; stencils stay on disk, read at call time.
- `PATTERN-sole-parsers` and `PATTERN-leaf-packages`: the classifier lives in `internal/planglyph` and imports the standard library only; no plan or recipe parsing moves.
- `PATTERN-test-economy` and `PATTERN-test-speed`: new cases extend existing table tests at the lowest tier that fits; a package past its row in `cmd/lyx/testdata/test-budget.yaml` has that file listed under `Edit:`.
- `PATTERN-comment-line-breaks` and the markdown rule: one sentence per line in Go comments and `.md` files.
- `PATTERN-documentation-lifecycle`: designs live in the packages' `doc.go`, not in a new design doc.

## Auto-mode assumptions

- No operator answered; every pick below is the producer's own and is logged as an auto-pick in the support log.
- #522 offers two shapes (count the first recovery's files as the batch's, or start clean with the diff as reference); the first was picked.
- "Exit non-zero" in #522 was read as a terminal `stuck` or `dead` recovery digest, because a terminal `failed` already exits non-zero through `batch_failed`.
- "The plan validator flags such a glyph at Plan time" was read as: a partitioned glyph no longer blocks the plan gate, and a still-ambiguous one is flagged there as `glyph-ambiguous` already, so nothing in this class first fails at record-batch.
- The root cause of the two #511 refusals was not reproduced; the base code and quarry v0.2.1 resolve unexported and test-file members, so the diagnostic is the deliverable.
- The `lyx gate test` hold needs Master to start gate runs as a plain command; Master's stencil is changed to say so.

## Open risks

- An unwrapped label is the only held form; a gate shell started through `cd`, `bash -c` or a pipe head is not held and is only recorded.
- The #511 root cause may be a real missing member or a shape the resolver does not cover; the richer detail will tell on the next occurrence.
- A reset's own-path set widens with recovery writes; a recovery that wrote a path outside its card now reads as the run's own, so the card check in the recovery prompt is the guard.
- The build-constraint evaluator covers `go/build/constraint` tags, GOOS, GOARCH and `unix`; `go:build` lines with custom tags assumed independent can classify two files as overlapping when a project never builds them together, which only keeps today's `ambiguous`.
- Create-after-delete in one card (#507, second half) stays unsolved.

## Acceptance criteria

- A Master turn end with every output file present and a `lyx gate test ...` shell outstanding does not finish the run; the run finishes after the shell's completion notification and Master's next turn end.
  A turn end with only an unrelated unawaited shell still finishes Done at once and lists the shell on `EndedShells`.
- At the run deadline with a gate shell outstanding and every file present, the run is Done, the shell is on `EndedShells`, and the warning, friction note and summary section each state that its result was not read.
- A failed verify gate sends `Gate findings recorded at <path> — ` followed by the entry's note; an entry with no note sends today's text unchanged; the sent text is one line.
- Master's stencil carries the plain-command rule for gate runs and the pointer to the gate message; a claims row pins both.
- A second recovery's prompt lists the first recovery's uncommitted paths under "Written by this run"; a path no recorded session wrote stays under "Not written by this run"; a `state.json` without recovery sessions loads and behaves as before.
- `lyx webster reset` over a tree holding only a recovery's uncommitted tracked paths does not refuse them as foreign.
- `recover-batch` exits non-zero with the digest fields intact for a terminal `stuck` and for a terminal `dead`; exit 0 for `done` and for a `running` snapshot.
- The recovery prompt renders the always-allowed section, and a claims row pins it.
- A Create done-check on an unexported identifier and on a `_test.go` member passes once the member exists; a missing member's finding names the key, the status and the unit status; an ambiguous finding names each candidate file.
- An `ambiguous` answer whose candidates sit in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`), and one split by `_linux.go` and `_windows.go`, passes a Create and a Rename new side, blocks a Delete and a Rename old side, and raises no plan-time `glyph-ambiguous` for an Edit target.
- An `ambiguous` answer with two declarations in one file, overlapping constraints, an unreadable candidate file or an unparsable constraint still blocks everywhere it blocks today.

## Notes for the plan writer

- Shuttle: `AwaitedShellPrefixes` and `awaitedLabel` in `internal/shuttleengine/wait.go` and `spec.go`; `gateRepromptText` in `gate.go`; `GateEntry` in `gate.go`.
- Webster: Master's spec in `internal/websterengine/runlevel.go` (the verify entry is built beside it); `expiredShellWarning` in `triage.go`; `recordedSessions` and `loadRunWrites` in `runwrites.go`; `recoverSpawn` and `BatchState` in `recoverbatch.go` and `state.go`; the CLI verb in `internal/webstercli/recoverbatch.go`.
- Stencils: `contracts/stencils/webster/webster-template-master.md`, `webster-prefix-recovery.md`; pins in `contracts/stencils/claims_test.go` and `internal/websterengine/template_test.go`.
- Planglyph: `donecheck.go` (`doneCheckVerdicts`), `resolve.go` (`statusFindings`), `create.go` (`createFindings`); `statusFindings` has no worktree root today, so the classifier's file reads need one threaded in.
- Verify the base state with `git show 800a0e892:<path>`; the working tree is ahead of it.
