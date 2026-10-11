## Goal

Close the Webster faults outside the audit that stop or mislead runs, as they stand at bench base `800a0e892`:

1. A Master run can finish `done` while a background `lyx gate test` shell is still running, and strand removal kills that shell unread (#505, #509, #517).
2. Master fixes a verify-gate failure itself instead of spawning the one fixer fork.
3. A second recovery reads the first recovery's uncommitted files as foreign, and `recover-batch` exits 0 on a terminal stuck or dead recovery (#522).
4. The recovery prompt does not say which gates the strand may run.
5. A `create-not-done` refusal does not say how its target was looked up (#511).
6. A member declared once per mutually exclusive build-constraint set or per GOOS resolves `ambiguous`, so its Create, Edit or Delete can never pass (#500, #507).

## Scope

In:

- `internal/shuttleengine`: the awaited-shell match and a per-entry gate re-prompt.
- `internal/websterengine`: Master's awaited prefixes, the gate-shell clause of the background-shell warning, the verify entry's re-prompt, recovery session attribution, and the recovery-terminal refusal.
- `internal/webstercli`: the `recover-batch` exit code.
- `internal/planglyph`: the constraint-split predicate, its normalization of resolve answers, and the done-check detail renderer.
- Stencils: `webster-template-master.md` (ladder rungs), `webster-body-implementer.md` (sandbox rule), `loom-template-plan.md` (per-GOOS Create shape).
- Specs and docs: `contracts/specs/refusal-spec.md`, `contracts/specs/webster-spec.md` and `contracts/specs/loom-plan-spec.md` where they quote changed text, plus `doc.go` of `shuttleengine`, `websterengine` and `planglyph`.

Out:

- Changing quarry (`github.com/Knatte18/quarry`, an external module that by design does not evaluate build constraints).
- A Create that reuses a name the same card deletes (#507's second half): with Edit and Delete working on per-GOOS glyphs, that detour is no longer forced.
- Reviving a dead batch through `lyx loom goto` (#511's third point).
- How forks run gates in-turn, quick checks in the foreground, and `cd` noise: board entry `fork-wait-noise` owns those.
- Renaming a constraint-split member: `renameDeclSource` stays found-only, and `lyx quarry expand` is unchanged.
- Moving gate child-lifetime code into `internal/proc` (board entry `proc-child-lifetime`).

## Decisions

### D1 — A running gate shell holds Master's turn end

Decision: Master's spawn declares a second awaited shell prefix, `lyx gate test`, beside `lyx webster recover-batch`.
A gated Master turn end with its output files present no longer finishes Done at once while a gate shell is outstanding: the shell is awaited like a fork, whether Master or a returned fork started it.
Bound: the hold ends at the shell's completion notification, the run's Timeout (`MasterTimeoutMin`) or the liveness check, and `lyx gate test` bounds its own wait (`limits.CLIWait`).
Only shells whose command has a segment starting with `lyx gate test` are held; every other shell keeps today's rule.

Rationale: at base, `background_shell_wait_min` already expires nothing, and the remaining fault is `countsAtOnce` reading a gate shell as unawaited.
The awaited prefix is caller data on `shuttleengine.Spec`, so the change stays within `PATTERN-shuttle-provider-seam`, and holding means the gate result is read instead of discarded.

### D2 — An awaited prefix matches the start of any command segment

Decision: shuttle's awaited-label match changes from a whole-label prefix to "some command segment starts with the prefix".
The label is split on `&&`, `||`, `;`, `|` and newlines, each segment is trimmed, and leading `NAME=value` words are skipped before the prefix compare.
This is a splitter, not a shell parser: it handles no quoting, subshells or `time`.
Bound: a separator inside quotes can only add a segment, so the splitter can cause a false hold, never a false release, and a false hold ends at the run's Timeout.
The match applies to Master's prefixes only, the sole caller of `Spec.AwaitedShellPrefixes`.

Rationale: the label is the raw Bash command, so `cd <pair> && lyx gate test …` and `cd <pair> && lyx webster recover-batch NN` slipped past a whole-label prefix.
The splitter is self-contained in `shuttleengine` because `PATTERN-shell-mechanics-seam` governs building pane commands, not reading labels, and `fabricengine`'s word splitter is not importable there.

### D3 — A gate shell ended with the run says its result was never read

Decision: `expiredShellWarning` (websterengine) gains a branch for a shell matching the gate prefix: the gate result was never read, so the run's outcome does not mean that tier passed.
That one function already feeds the run warning, the `webster-background-shell` friction note and summary.md's background-shell section, so all three carry the clause.

Rationale: after D1 a gate shell still ends unread when the run dies or times out, and the outcome alone can still read `done` (`finishedShellOutcome`).

### D4 — The verify gate's re-prompt sends Master to the fixer fork

Decision: `shuttleengine.GateEntry` gains an optional re-prompt: a function of the findings path returning one line.
A nil function keeps `gateRepromptText` unchanged for every other gated producer.
Webster's `verify` entry supplies a line that keeps the `Gate findings recorded at <path>` opening, then says not to fix the findings yourself, to spawn the one verify fixer fork on the verify-fix prompt file, and to end the turn.
The line passes the existing `validateSendText` one-line check, and both the first send and the unsent-retry path use it, since both go through the one call site in `wait.go`.
The findings file (`renderVerifyGateFindings`) is unchanged, because the fixer fork reads it, and its "commit or remove these paths" is addressed to the fixer.

Rationale: at base the line Master reads at a failure says "read it, fix every finding", which contradicts its stencil rule.
No verb can spawn an in-session fork, since forks are Agent-tool calls, so the fix is the text at the moment of reading.
`PATTERN-stencil-ownership` is not breached: the re-prompt is a Go-authored line shuttle types mid-run, like `gateRepromptText` today, and not a prompt `stencilstore` owns.

### D5 — Recovery strands' writes count as the run's own

Decision: each recovery spawn records its shuttle session id on the batch record, in a list carried across respawn and re-begin as `Recoveries` is.
`reset --to batch-start` clears the list, and `reset --to start` archives it with the record.
`RunWrites` gains a third slice, `Recoveries`, filled by auditing those sessions.
The own-path readers `writtenWorktreePaths` (recovery prompt classing) and `ownTrackedPaths` (reset) concatenate it with `Master` and `Forks`.
`contractevidence` treats it like `Forks`, never as Master's own writes.
A recovery session whose transcript is missing or unreadable is logged at Warn and contributes no paths, and the recovery spawn does not fail on it.
Bound: only sessions webster spawned and recorded for this run are attributed, and an unattributable path stays under "Not written by this run".

Rationale: at base `loadRunWrites` audits only Master sessions, and a recovery record has no `SessionID`, so the first recovery's 19 files read as foreign.
Starting the second recovery from a clean tree would discard that work through a destructive step.
Reset counting recovery-written paths as its own is correct, because reset stops every recorded recovery strand before it acts.

### D6 — `recover-batch` exits non-zero on a terminal stuck or dead recovery

Decision: a terminal `stuck` or `dead` digest returns through `output.ErrFields`, exiting non-zero, and keeps every field the ladder reads: the digest fields (`status`, `dead_reason`, …), `recoveries`, `recovery_retry` and `warnings`.
Its message names the way forward.
With `recovery_retry: true`, that is `lyx webster recover-batch NN` once more.
Otherwise the run ends stuck naming the batch, and the operator's route is `1) lyx webster reset --to batch-start --batch NN; 2) lyx webster recover-batch NN`, or `1) lyx webster reset --to start; 2) lyx webster run`.
A terminal `done` stays on `output.Ok`.
Master's ladder rungs for stuck and dead change from "completes with" to "refuses with" and keep their actions.
The refusal gets its row in `contracts/specs/refusal-spec.md` (`PATTERN-refusal-way-forward`), and the existing webster friction capture for a halting refusal applies (`PATTERN-friction-capture`).

Rationale: the batch is failed and terminal, and exit 0 told every caller other than Master's JSON reading that recovery succeeded.

### D7 — The shared implementer body names the always-allowed gates

Decision: the sandbox rule in `webster-body-implementer.md`, which forks and recovery strands both read, gains two sentences.
The first says the card and batch gate steps, `lyx gate test` and `lyx loom lint-comments`, are not fabric verbs and are always allowed.
The second says `lyx gate test --tags tmux <packages>` is permitted, but the plan verify compiles the tmux tier and Publish runs it, so a strand never needs to run it and at most compile-checks it with `go vet -tags tmux <packages>`.
The existing sentence "a tmux-tier run goes through `lyx gate test`" is reworded so it no longer reads as a duty.

Rationale: the recovery strand could not tell whether its gates fell under the fabric ban.
Nothing at base refuses `lyx gate test --tags tmux` (the only deny is the raw-go pattern), so "never needs to" is accurate.
Forks read the same ban, so the body is the one place.

### D8 — Done-check details say how the target was looked up

Decision: one renderer in `planglyph` writes the detail of `create-not-done`, `delete-not-done` and `rename-not-done`.
It names the resolve key, that it was resolved through quarry against the post-batch worktree, and the answer.
For `not_found` it says whether the unit exists with the member missing, or the unit is missing (`ResolveResult.Unit`).
For `ambiguous` it lists the candidates with their files, and why they were not read as one member: their build constraints overlap, or a candidate's constraints could not be read.
The detail uses `, ` and never `; `, because `failCardNotDone` splits reasons on `; `.
The record answers the entry's question about unexported and `_test.go` members: quarry v0.2.1 resolves both, and the #511 targets failed on per-GOOS or build-tagged ambiguity, which D9 fixes.

Rationale: an operator must be able to tell a missing member from a resolver answer lyx declines, and the lookup the detail names is reproducible with `lyx quarry resolve`.

### D9 — A constraint-split ambiguous answer reads as one multipart member

Decision: one predicate in `planglyph` decides "constraint-split".
An `ambiguous` answer qualifies when all of these hold:

- every candidate shares one glyph ID;
- each candidate sits in a distinct file;
- no build configuration selects two of those files.

Configurations are judged with stdlib `go/build` `Context.MatchFile`, over one-hot GOOS and GOARCH values (those the candidates' file names and `//go:build` lines mention, plus a sentinel "other" value) crossed with the subsets of the remaining tags mentioned.
Filename `_GOOS`/`_GOARCH` suffixes and `unix` follow from `go/build`'s own matching.
The predicate reads only each candidate file's header and runs no `go` subprocess.
It fails closed: an unreadable file, a candidate no configuration selects (such as `//go:build ignore`), or a `_test.go` file paired with a non-test file is not split.
`resolveTargets`' callers in planglyph get a normalized answer: a constraint-split result becomes `multipart` with its candidates copied into `Symbols`.
So every planglyph reader agrees by construction:

- `statusFindings` raises no `glyph-ambiguous`;
- `createFindings` reports `create-already-exists`, correctly, since the member exists;
- `doneCheckVerdicts` treats it as resolved for Create and Rename-new, and as still existing for Delete and Rename-old;
- `deleteorder`'s `answerSymbols` sees its symbols;
- the drift revalidation and the scope guard treat every candidate file as the member's.

`planglyph.Resolve`, which feeds `lyx quarry resolve`, keeps returning quarry's answer verbatim, as that verb documents.
Bound: only answers meeting the three conditions above are relabeled, and a genuinely ambiguous answer stays blocking at Plan time and at record-batch.

Rationale: the in-repo route is a lyx-side predicate over `ResolveResult.Candidates[].File`, since quarry will not evaluate build constraints.
Normalizing once, inside the package's one resolve boundary, means no reader can be missed.
Free booleans would read `linux` and `windows` as co-true and never find them disjoint, so the GOOS and GOARCH values are modeled one-hot.

### D10 — Plan time: one shared verdict, no new rule

Decision: Plan time adds no new validator rule.
For a member that exists at Plan time (Edit, Delete, Uses), D9 gives Plan and record-batch the same verdict.
A per-GOOS Create cannot be judged from quarry at Plan time.
Its only legal shape is already enforced: one Create declaration in one card covering every file, since `handle-collision` blocks two cards claiming the same handle.
`BindHandles` already binds a per-GOOS Create, since quarry's delta reports both occurrences as `Created` under one ID.
`loom-template-plan.md` gains one line saying a per-GOOS or build-tagged member is one glyph, created by one declaration in one card whose files are all its declarations.

Rationale: the entry's "flag at Plan time" is met for every glyph Plan time can see.
For a Create, the existing `handle-collision` rule and the plan stencil line are the real Plan-time signal.

## Constraints

- `PATTERN-shuttle-provider-seam`: D1 and D2 change caller data and provider-neutral matching only; Claude payload shapes stay in `claudeengine`.
- `PATTERN-completion-signal`: D1 and D2 must keep the output-files-first order in `expiredTurnEnd` and `countsAtOnce`.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: the stencil edits (D6, D7, D10) are read at call time through `stencilstore`, and D4's line is Go-authored re-prompt text, as stated in D4.
- `PATTERN-refusal-way-forward`: D6's refusal and D8's reworded details carry a way forward, their rows in `contracts/specs/refusal-spec.md`, and a test reaching each.
- `PATTERN-friction-capture`: a halting webster refusal still writes its Go-authored friction note.
- `PATTERN-fabric-destruction-chokepoint`: nothing here deletes uncommitted work. D5 adds attribution only, and reset keeps its existing route.
- `PATTERN-glyph-conversion-chokepoint`, `PATTERN-sole-parsers` and `PATTERN-ref-shape-registry`: D9's predicate reads candidate file paths and build lines only, and never converts glyphs or classifies plan refs.
- `PATTERN-leaf-packages`: neither `planglyph` nor `shuttleengine` is a leaf package, and `go/build` is stdlib.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-testkit`: extend existing tables (`donecheck_test.go`'s status table, `waiting_test.go`'s awaited cases, `runlevel_test.go`, `recoverbatch_test.go`, `contracts/stencils/claims_test.go`) before adding top-level tests.
  A package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Docs in the same commit: `shuttleengine/doc.go` (awaited match, per-entry re-prompt), `websterengine/doc.go` (awaited prefixes, gate-shell clause, recovery attribution, recover-batch exit), `planglyph/doc.go` (constraint-split, done-check details), and every spec that quotes changed text.
- `contracts/stencils/claims_test.go` pins the `Gate findings recorded at …` anchor and the ladder wording; the pins move with the text.

## Auto-mode assumptions

- The session ran autonomously with one advisor, `ly:twb-seat-lead-1:discussion-advisor-1`, and every decision rests on its answers plus code read at `800a0e892`.
- #511's failures were per-GOOS ambiguity, not a resolver gap. The code and today's probes support this, but the failing runs' traces were not read.
- The recovery strand's pane cwd is the worktree root, so `AuditForks` finds its transcript under the same project directory as Master's. A test pins it (Acceptance criteria).
- An unreadable recovery transcript degrades to "unattributable" instead of refusing the spawn.
- A `//go:build ignore` candidate is not split.
- `lyx quarry resolve` stays verbatim.

## Open risks

- The `lyx gate test` prefix holds a Master turn end for as long as a gate run lasts. A wedged gate run pins Master to its Timeout rather than finishing early, which is the intended side of the trade.
- D2's splitter misses `time lyx gate test` and `( … )` forms, and those shells keep today's unawaited rule.
- A per-GOOS Create whose one variant already existed before the batch reads as Modified, not Created, in quarry's delta, so `BindHandles` would not bind it. D10's stencil line steers plans away from that shape, but nothing enforces it.
- D5's attribution depends on Claude transcripts being on the same machine, like every existing write-evidence reader.

## Acceptance criteria

- A gated Master turn end with its output files present and a `lyx gate test …` shell outstanding keeps waiting, and the run finishes only after the shell's notification starts and ends another turn. The same holds for `cd <dir> && lyx gate test …` and `FOO=1 lyx gate test …`.
- A shell matching no awaited prefix keeps today's behavior, and `lyx webster recover-batch` stays awaited through a `cd … &&` segment.
- A gate shell outstanding when a Master run dies or times out produces a warning, friction note and summary text saying its gate result was never read.
- A failing verify entry's re-prompt starts `Gate findings recorded at <path>`, says not to fix the findings and to spawn the verify fixer fork, and is one line. Another producer's gate entry still sends the generic text.
- A second recovery of a batch whose first recovery wrote uncommitted files lists those files under "Written by this run". A path no recorded session wrote still lists under "Not written by this run".
- A recovery session with no readable transcript does not fail the next recovery spawn and contributes no paths.
- `reset --to batch-start` clears the batch's recovery session list, and reset's own-path set includes recovery-written tracked paths.
- A recovery write never counts as Master's own write in the contract-file evidence.
- `recover-batch` on a terminal stuck or dead recovery exits non-zero with `status`, `dead_reason`, `recoveries`, `recovery_retry` and `warnings` in the envelope and a message naming the way forward. A terminal done still exits 0.
- The refusal-spec row for that refusal exists, and a test reaches it.
- `create-not-done` for a not_found target names the resolve key, the lookup and whether the unit or the member is missing. For an ambiguous target it lists candidate files and why they are not one member, and no detail contains `; `.
- A member declared in `x_linux.go` and `x_windows.go`, or under `//go:build tmux || llm` and `//go:build !tmux && !llm`, passes `create-not-done` after its Create, passes `glyph-ambiguous` at Plan time for an Edit, and fails `delete-not-done` while either declaration remains.
- Two declarations with overlapping constraints, a `_test.go` and non-test pair, an unreadable candidate and a `//go:build ignore` candidate all stay ambiguous and blocking.
- `lyx quarry resolve` on a per-GOOS member still answers `ambiguous` verbatim.
- The implementer body names `lyx gate test` and `lyx loom lint-comments` as always allowed and the tmux tier as left to the plan verify and Publish. The plan stencil carries the per-GOOS Create line.

## Notes for the plan writer

- Item 1 code: `shuttleengine/wait.go` (`awaitedLabel`, `countsAtOnce`, `expiredTurnEnd`), `shuttleengine/spec.go` (`AwaitedShellPrefixes`), `websterengine/runlevel.go` (`masterAwaitedShellPrefix`), `websterengine/triage.go` (`expiredShellWarning`).
- Item 2: `gateRepromptText` has one call site in `wait.go`, where the failing entry is in scope.
- Item 3: `websterengine/runwrites.go` (`loadRunWrites`, `recordedSessions`, `writtenWorktreePaths`), `reset.go` (`ownTrackedPaths`), `contractevidence.go`, `recoverbatch.go` (`recoverSpawn` holds the shuttle run state with its session id), `state.go`, and `webstercli/recoverbatch.go` for the exit. `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` mentions recover-batch, so check it asserts no exit 0 for stuck or dead.
- Items 5 and 6: `planglyph/repo.go` (`resolveTargets`, `Resolve`), `donecheck.go`, `create.go`, `resolve.go` (`candidateList`), `deleteorder.go` (`answerSymbols`), `handle.go`. The done-check wording is quoted in `webster-template-master.md`, `loom-plan-spec.md` and `planglyph/doc.go`, and `recoverbatch_test.go` pins "still resolves found".
