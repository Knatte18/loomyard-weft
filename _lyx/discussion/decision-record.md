# Decision record: webster-replan-state

## Goal

Webster stays consistent across replans, its batch and fabric checks judge only what they should, implementer forks stay sandboxed, the shell wait leaves nothing unrecorded, and the round gate re-runs whole tests.
The defects from the board entry (#466, #465, #476, #484, #494, #493, #496, #497, #492, #498, plus the overview-frame log, the batch-check call and the verify-chain split) are fixed in one sweep, each with its test and its doc in the same commit.

## Scope

In:

- A `lyx loom goto` back to a row before Webster, after Webster recorded state, never hard-fails the next Webster step on a plan fingerprint mismatch (#466).
- A Plan-Review round over a plan webster has begun tells the reviewer and fixer which cards are frozen, refuses an edit to a done card at the plan gate, and lands follow-up cards through webster (#465).
- A rebaseline over an empty overview frame logs the cause and its refusal names it.
- `commitsNamedBy` classifies every candidate in one `git cat-file --batch-check` call.
- `verify-nested-module` and `verify-module-wide` split a chain at `||` and `|` too.
- `record-batch`'s done-checks are pinned to the batch's own cards and its refusal names batch and card consistently (#484).
- An implementer fork or recovery strand cannot drive the real hub's fabric: the stencil states the sandbox rule, `lyx fabric`'s mutating verbs refuse from such a strand, and `lyx fabric add --help` says it pushes (#494).
- The fabric-reference audit treats a read-only command as policy, and recover-batch accepts a failed batch whose pathless findings are all read-only (#493).
- One background-shell wait rule for transcript-reported and payload-reported shells, and a record of every shell a run ends over (#496).
- The round gate's failing-test re-run names the top-level test only (#497).
- `TestLoomResumeReconcilesTheHubConfigBeforeArming` is deterministic (#492).
- A dead batch whose recovery left commits gets one more recovery from Master, bounded (#498).
- A card that prescribes a verbatim `PATTERN.md` entry line over `MaxEntryLineChars` fails the plan gate (#476).

Out:

- Any Go-side route from a blocked row to a fixer (publish-recovery decided the hub runs the goto).
- Killing an individual background shell inside a provider session; lyx has no pid for it.
- A local-only `lyx fabric add` flag or a scratch-hub remote switch: refusing the verb from the sandboxed roles covers the seen failure.
- Making quarry honour build constraints (board note `quarry-build-tag-ambiguous`).
- The overview size cap at Publish (board note `pattern-overview-headroom`) and the per-attempt verify log (board note `verify-log-overwritten`).
- Bounce-budget ordering (board note `bounce-budget-after-approved`).

## Decisions

### D1. The Webster row rebaselines a changed plan on entry instead of failing

Decision: `websterengine.RunOptions` gains a mode the loom's Webster producer sets and `lyx webster run` never sets: on a fingerprint mismatch at entry, Run calls `Rebaseline` with `Cards` set to the number of every changed card file (`changedPlanFiles`), through the same deps `lyx webster rebaseline` builds.
An accepted rebaseline saves the restamped state with `SaveState` before the batch loop starts, so a crash after it re-enters on a matching fingerprint and never re-runs the rebaseline.
A failed save is an auto-rebaseline error like the rebaseline's own: it blocks the row as Stuck with the transient re-step text below, and Run never continues on the unsaved restamp.
It is logged at Warn and joins the run's entry warnings: a done outcome carries it on `RunResult.Warnings` and in the run summary, and a stuck outcome's reason opens with it, so the step's Stuck reason or the summary its Done points at names it.
Any mismatch `lyx webster run` itself meets keeps today's `ErrFingerprintMismatch` text.
`Rebaseline` takes the step its refusals name from its deps, as Run takes `reentryStep`: `lyx webster run` by default, the row's re-step under the loom.
Every clause of its refusals that names `lyx webster rebaseline` is keyed the same way: the follow-up-card landing, the changed-but-not-named card set and each transient retry; on the auto path each says to fix the plan, a done card's edit moved to a follow-up card, and re-step the Webster row, never to run `lyx webster rebaseline` by hand.
Run wraps every error the auto-rebaseline and its save return in one new sentinel, `ErrAutoRebaseline`, and the loom's producer maps an error matching it, the refusal sentinels and a transient read or restamp failure alike, to `Stuck` with the error text as the reason, never to a returned error, so the run blocks resumable instead of failing with `producer-hard-failure`; a transient failure's text says to re-step the row.
The auto-rebaseline syncs nothing to the fabric itself: the restamped state reaches it with the run's next sync, a bracket verb's or the run end's, as a saved state does after `fabric sync failed`.
A nil plan or state is a programmer error Run never passes, and stays a returned error.
Bound: the auto-rebaseline accepts exactly what an operator naming every changed card with `--card` gets: a done batch's card edit, a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside its Card Index are still refused, each blocking the row as Stuck with the rebaseline's own refusal text; a card of an in-flight batch, which exists at row entry only after a crash, is accepted as an amendment, recorded unrendered and re-run by the next recovery.
Of these refusals only the done-card edit is caught earlier, at the plan gate (D2); the other three are left to the Webster row.
Rationale: the Plan loop edits the plan in place after a goto to Plan-Bouncer, Plan-Burler or Batchifier, and only a goto to Plan-Write rotates the plan and archives webster's record (PATTERN-plan-generation); every other replan reaches Webster with a record the operator wants kept, and the advertised way forward was always a rebaseline.

### D2. A Plan-Review round over a begun webster run knows what is frozen

Decision, three parts:

- The Plan-Review rubric gains an optional `webster_record` marker, filled by Go in the rubric reader the way `publish_failure` is, and so reaching the Plan-Burler fixer through the rubric its explore step already carries: `none` with no webster record, else the begun batches, the done cards, and the rule that a done card is frozen and a finding against it lands as a follow-up card after the last begun batch with its Card Index line in `00-overview.md`.
- The plan gate (`loomshed.ValidatePlan`, already told webster's done cards) gains a blocking `done-card-edited` finding: a done card whose file hash differs from the hash webster recorded at begin, computed in `websterengine` beside `DoneCards`, naming the card and the follow-up landing.
  Bound: the check reads webster's record and card bytes, never parses the plan; with no record or no recorded hashes it reports nothing.
  It gates the done-card edit only; a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside the Card Index pass the gate and block at the Webster row (D1).
- The follow-up cards reach webster through D1's rebaseline at the Webster row.

Rationale: the fix round is winnable once the fixer knows the write surface, and the gate turns a done-card edit, the refusal a Plan-Review fix most often meets, into a finding at the round instead of a refusal at Webster.
The fix-round dispatch itself is not skipped: findings against unbegun cards stay fixable.

### D3. The overview-frame fallback is logged and named

Decision: `recordedOverviewFrame` returns the cause of an empty frame beside it, for each of the three causes: no overview hash recorded, the copy absent, or a copy without a parseable Card Index.
`Rebaseline` logs the cause at Info at its own call, with the baseline copy's path where there is one, so a rebaseline logs it once; the helper logs nothing, and the fingerprint-mismatch way forward, its other caller, keeps its own no-frame text unchanged.
`overviewRefusal`'s no-frame text names the cause and the path.
The refusal-spec row `rebaseline overview changed` carries the new wording.

### D4. One batch-check call per prefix

Decision: `commitsNamedBy` feeds every object name `rev-parse --disambiguate` lists to one `git cat-file --batch-check` run and keeps the lines typed `commit`; a `missing` line is an error.
The call goes through `gitexec`'s checked form with stdin, added there if it has none (PATTERN-gitexec-checked-call).

### D5. One chain splitter for the verify checks

Decision: `planparser` gets one chain splitter that both `crossModuleCommands` and `moduleWideCommands` use, splitting at `&&`, `||`, `;`, `|` and newline, with `||` never read as two pipes, a separator inside a quoted span read as text, and a backslash-newline continuation joined into one line before the split.
In `crossModuleCommands` a `cd` moves the run directory only when its segment follows `&&`, `;` or a newline; a `cd` in a pipeline stage or after `||` ends the scan, as an unresolvable `cd` does today.
Rationale: both checks had the same gap; one splitter keeps them in step.

### D6. Done-checks stay on the batch's cards, and the refusal says which batch holds the card

Decision: a table row pins that `postBatchChecks` hands `DoneChecks` the batch's own cards only, for a multi-card batch beside unbegun cards.
`failCardNotDone`'s reasons open with the batch's id and its card list (`batch 01-headcontains-bounded-walk holds cards 1-…, 2-…, 3-proc-group-kill`), so a finding keyed on a later card of the batch reads as the batch's own.
Rationale: the code already scopes the checks; the seen refusal named a third card of a three-card batch, and the text gave no way to see that.

### D7. An implementer fork or recovery strand cannot drive fabric

Decision, three parts:

- `webster-body-implementer.md`, shared by the fork and the recovery prompt, states the sandbox rule: never clone the hub's remotes, never run a mutating `lyx fabric` verb; a scratch hub is a Go test fixture over local bare repositories (`hubforge`), and lyx refuses the verbs from your strand.
- `fabriccli`'s persistent pre-run refuses every fabric subcommand but the readers from a session whose strand name (`agentname.StrandNameEnv`) parses with a slug and a role that `agentname.MatchesRole` matches against neither the driver's (`agentname.RoleDriver`) nor the orchestrator's (`agentname.RoleOrch`, which a nested pair's orch carries with a slug).
  The readers are `list`, `pairs`, `status`, `diff`, `shortname` with no argument, `prune` and `cleanup` without `--apply`, and every `--help`; any other subcommand, one added later included, is refused, and a test walks every registered subcommand and pins it reader or refused.
  The refusal's way forward names the sandbox rule's step: build a scratch hub as a `hubforge` test fixture, and when the work itself needs the refused verb, stop and report `status: FAILED`, so the run ends stuck and the orch runs the verb.
  A `push` carrying the hidden `--weft-path` or `--warp-path` bypass flag passes: it is the detached push child every `Fabric.Commit` spawns, which inherits the strand name, so refusing it would silently drop the records push of every slugged role that commits through lyx, Master's bracket verbs included.
  Under a refused role the guard refuses a bypass flag on every subcommand but `push`, since today's bypass refusal covers only the weft verbs and a topology verb ignores the flags.
  The guard is the first check of `fabric`'s persistent pre-run, ahead of the weft pre-run's early return for a topology verb, and it runs only when the `fabric` command has a parent, as `main.go` mounts it; `configcli`'s in-process sync runs `Command()` as its own root through `RunCLIIn` and so passes.
  The per-worktree write that sync follows is bounded by `configcli`'s hub-wide guard and syncs the session's own worktree only.
  The subcommand walk and the guard rows mount `Command()` under a bare parent command, as `main.go` does.
  Every refusal-spec row and every error text whose way forward names a fabric verb the guard refuses gains that the orch runs the verb when the refusing session cannot; the plan writer derives that set by searching the spec and the Go error texts for `lyx fabric`, never from a list here.
  Master's stencil already ends stuck on a fabric-sync failure.
  Bound: like `configcli`'s hub-wide guard it reads the strand name and nothing else: an unset, empty or unparseable name passes, as does a slug-free one, so it stops a mistaken call from a task session, not a fork that clears or rewrites the variable.
  The bypass flags are hidden and set only by Go, but nothing stops a fork from typing them: a bypass `push` pushes the branches of the worktree paths it names, and the guard refuses the flags on every other subcommand.
  A raw `git push` is outside lyx and stays the stencil's and the audit's.
  `lyx gate test` and `verifytree.Verify` drop the strand-name variable from the environment of the processes they run, so an implementer's or Master's gate or verify command never hands it to a test, in-process or through a `lyx` subprocess, and no existing fabric test changes.
  This opens nothing the guard's bound does not already concede: a test is code the fork writes, which could clear the variable itself.
  A guard row sets the strand name itself with `t.Setenv`, as `configcli`'s guard rows do, and so runs without `t.Parallel` under a comment naming that variable.
- `lyx fabric add`'s long help states that it pushes the new code and records branches to origin.

Rationale: the driver's stencil names fabric verbs, and the orch is the operator's delegate the refusals route a refused verb to; the conflict resolver's stencil forbids any git, and every other task-session role has no business in fabric, so deny-by-default with that allowlist.

### D8. A read-only fabric reference is a warning, not a failed batch

Decision: `fabricengine.IsReadOnlyCommand` stays the one classifier, closed and static, and keeps its own quote-aware segment split, now with `||` read as a separator.
It keeps today's refusals: any output redirection, command or process substitution, backgrounding, and a first word that is quoted or not a listed command, so a command behind `env`, `xargs`, `bash -c` or a variable is rejected.
Every word of a `lyx` or `git` segment must be plain, holding no quote, backslash, `$`, brace or glob character (`*`, `?`, `[`) and no leading `~`, so the word the classifier reads is the word the shell hands the command; a segment with any other word is rejected.
Every segment must then be one of:

- a reader from today's list;
- a `lyx` help invocation: `lyx`, then unquoted lowercase command words only, then `--help` or `-h` as the last word, so no flag can take the help word as its value;
- `lyx fabric` followed by one of D7's readers and no other word: `list`, `pairs`, `status` or `shortname` alone, `diff` with exactly one word not starting with `-`, `prune` or `cleanup` with no flag, so the classifier admits a subset of the guard's readers, built from the same verb names;
- a `git` read whose only global option is `-C <dir>`, in one of these forms, where every word starting with `-` before a `--` must be one the form lists, matched exactly, so an abbreviated long option or a synonym such as `--exec` rejects it, and every other word must be a positional word the form admits, so a subcommand word such as `add` after a listed flag rejects it:
  - `remote get-url` with `--push` or `--all`, then one remote name; `remote -v` with no other word; `remote show` with `-n`, then remote names;
  - `ls-remote` with `--heads`, `--tags`, `--refs`, `-h` or `-t`, then an optional repository and patterns;
  - `rev-parse` with `--abbrev-ref`, `--short`, `--verify`, `-q`, `--show-toplevel` or `--git-dir`, then revisions;
  - `log` and `show` with `--oneline`, `-n` followed by a word of digits, `-<digits>`, `--format=…`, `--pretty=…`, `--stat`, `--name-only`, `--name-status`, `--graph`, `--decorate`, `--all`, `-s` or `--no-patch`, then revisions and, after `--`, paths;
  - `diff` with `--stat`, `--name-only`, `--name-status`, `--cached`, `--staged` or `--no-color`, then revisions and, after `--`, paths;
  - `status` with `-s`, `--short`, `--porcelain`, `-b` or `--branch`, then paths;
  - `branch` with `-a`, `--all`, `-r`, `--remotes`, `-v`, `-vv` or `--show-current` and no positional word, or with `--list`, then patterns;
  - `cat-file` with exactly one of `-t`, `-s`, `-e` or `-p`, then one object name;
  - `config` with `--get` or `--get-regexp`, then one name and at most one value pattern.

`websterengine` keeps importing no `fabricengine`: the classifier reaches it by injection, as `AcceptBatchFabricReference` already takes it.
`ClassifyViolation` and `RecoverDeps` take the classifier the same way, and `webstercli` supplies `fabricengine.IsReadOnlyCommand` to both.
`ClassifyViolation` classes a fabric reference whose command the classifier accepts as policy, so the batch records an audit warning and passes.
`RecoverSpawnOrAttach` accepts in line a terminal failed batch whose `Uncheckable` entries are all pathless fabric references with read-only commands, through `AcceptBatchFabricReference` and its evidence rule (HEAD at the batch start, or the start an ancestor and the tree clean), recording the warnings and spawning the recovery.
A command the classifier rejects still refuses `needs_fresh`; failed evidence refuses with `AcceptBatchFabricReference`'s `ErrAuditNotAcceptable` text, whose way forward names `recover-batch` for the re-run when the in-line path meets it, and `recover-batch` carries `audit_not_acceptable: true` on that refusal's envelope.
The Master stencil gains a rung: `recover-batch <NN>` refusing with `audit_not_acceptable` writes `outcome: stuck` with a `stuck_reason` quoting the refusal, and stops, since Master cannot clean the tree.
Bound: the classifier sees static shape only and admits nothing outside the forms above; a command it rejects keeps today's correctness class.
`AcceptBatchFabricReference`'s bound, that no listed reader can write, is rewritten as no command the classifier accepts writes tracked content, a ref, config or a remote; `git status` may refresh the index's stat cache, which changes no content.
The rewrite flips `TestIsReadOnlyCommand`'s `git status` row, adds rows for the `lyx` forms, the `git` forms and `||`, and rewrites the classifier's doc comment, which says `lyx` and `git` are never read-only.

### D9. One shell-wait rule, keyed on the output files, and every shell a run ends over is recorded

Decision: shuttle's wait no longer distinguishes a transcript-reported from a payload-reported shell.
An ungated run keeps today's rule: a turn end with every output file present counts at once, whatever is outstanding.
A turn end with an output file missing waits on every outstanding task, a shell of either signal included, until the run's own deadline and the liveness check.
A gated run's waiting turn end with every output file present keeps today's deferral, since the files may be left over from an earlier arrival, with one exception: it counts at once when every outstanding task is an unawaited shell of either signal, the run was started fresh rather than attached or resumed, and the gate has evaluated no arrival of this run yet, a pending `PassOnCap` or `MayHold` answer included.
Only then is no earlier arrival possible and the files this session's.
A fork, an awaited-prefix shell or an empty outstanding list holds a gated waiting turn end whatever the files, so `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` stands.
Every other gated waiting turn end waits on its shells as a missing-file turn end does, and only a later turn end with nothing outstanding is the arrival.
`background_shell_wait_min` stays in `shuttle.yaml` as the threshold after which an outstanding shell is logged once at Warn and shown in the wait marker; it expires nothing.
The shell expiry leaves webster's turn-end reading too: `ShellWaitExpires` is removed and `TurnEndedAfter` counts a turn end only with nothing outstanding, so a recovery strand waiting on a shell of either signal runs until its report exists, which `Classify` reads before any turn end, or until `recovery_timeout_min`.
`Result.ExpiredShells` is renamed `Result.EndedShells` and becomes a record of each shell outstanding when the run ends, whatever ends it: an ungated or gated at-once finish, a later arrival, the run deadline or the liveness check; each entry holds label, id, signal and how long it was outstanding.
Webster's summary section and friction note render the record, and the strand removal that ends the run logs that it ended those shells.
Bound: lyx kills no shell by itself, since the provider reports no pid; a shell detached from the pane's process tree survives the removal, and the record says the removal ended the shells it could.
A stale shell can now hold a run open until its deadline but never ends a turn early: files missing keeps it waiting, and in a gated run files present count only where no earlier arrival could have left them.
The rewrite reverses the transcript-expiry cases in `internal/shuttleengine/waiting_test.go` (`TestPollEventsTick_ShellExpiry`, `TestWait_GatedShellExpiryEvaluatesGate`) and `TestRun_ExpiredShells` in `internal/websterengine/runlevel_test.go`.
`TestTurnEnded` drives an engine without the session-signal capability, where no shell is ever outstanding, so `TurnEndedAfter`'s outstanding-shell cases need a new test over a signal-parsing fake engine.
Rationale: 7462c1dc5 made a payload shell never expire because a gate behind a slot wait can outlast ten minutes; the transcript fallback is the same work reported through a weaker signal, and the output files, not the signal, say whether the agent is finished.

### D10. The round gate re-runs the top-level test

Decision: `publishFailureCommand` anchors `-run` on the first `/` segment only (`^TestX$`) and collapses the record's tests to one step per package and top-level test; `PublishFailureNote` keeps the full subtest path.

### D11. The resume-reconcile test is made deterministic by reproduction

Decision: the plan's card reproduces `TestLoomResumeReconcilesTheHubConfigBeforeArming` under at least twenty repeated runs of the package's integration tier, names the dependency it finds and removes it; the fix is not predicted here.
The card shows the named dependency failing the test before the fix, by forcing it, and passing after; when no run reproduces the failure, the card documents the attempt and the dependencies it ruled out.

### D12. A dead batch whose recovery left commits gets one more recovery

Decision: `BatchState` gains `Recoveries`, the count of the batch's counted recovery spawns, and `RecoveryStartSHA`, HEAD when the record's recovery was spawned.
`recoverSpawn` reads HEAD into `RecoveryStartSHA` before `Starter.Start`, and the record it returns after a successful start carries `Recoveries` as the prior's plus one; `recover-batch` saves that record before it awaits the strand, as it saves every spawned record today; `StartSHA` keeps inheriting the original bracket's start.
A start that fails counts nothing, so a transient start failure burns no recovery.
A spawn whose prompt renders an amendment no earlier spawn rendered, Master's `card_amended` re-run, keeps the prior's count: each amendment forces at most one re-run, and only a plan edit makes one.
A rebaseline keeps batch records and so the count, and a re-begin (`BeginBatch`) carries `Recoveries` and `RecoveryStartSHA` over beside `StartSHA`, `AuditWarnings` and `ForkTranscripts`.
`lyx webster reset --to batch-start --batch NN` moves the branch and keeps the record, so it clears batch NN's `Recoveries` and `RecoveryStartSHA` and saves the state; `--to start` archives the whole record, which clears every count.
`RecoverSpawnOrAttach` refuses a counted spawn for any caller once `Recoveries` is 2, with `recovery_exhausted`, naming `lyx webster reset --to batch-start --batch NN` then `recover-batch`, or `reset --to start`, and `recover-batch` carries `recovery_exhausted: true` on that refusal's envelope; below the cap it respawns over a terminal prior as today.
The recover-batch envelope and each batch entry of `lyx webster status` carry `recoveries` and `recovery_retry`, computed by one `websterengine` function: true for a terminal `dead` recovery-kind prior while `Recoveries < 2` when the dead recovery committed, that is HEAD descends from a non-empty `RecoveryStartSHA` and the first-parent range from it to HEAD holds a commit that is not a merge; an empty `RecoveryStartSHA`, as on an older record, a HEAD a reset moved off it and a range of merge-ins only all give false.
A git read that fails, a `RecoveryStartSHA` absent from the store included, gives false and is logged at Warn, so neither verb fails on it.
Both of the Master stencil's dead rungs, the `recover-batch` ladder's and the entry-time one read from `status`, say: `dead` with `recovery_retry: true` runs `recover-batch <NN>` once more, backgrounded; otherwise the run ends stuck as today.
The sentence that a batch whose recovery is exhausted ends the run is reworded to a dead batch with `recovery_retry: false`.
`begin-batch`'s terminal-dead remedy and its `report already present` spec row are reworded the same way, through the same function: `recovery_retry: true` runs `lyx webster recover-batch NN`, otherwise the run ends stuck naming the batch.
A new rung says: `recover-batch <NN>` refusing with `recovery_exhausted` writes `outcome: stuck` with a `stuck_reason` quoting the refusal, and stops.
Bound: at most two counted recoveries per batch for every caller, plus one uncounted re-run per amendment, and Master retries only when the dead recovery itself committed; an operator may still run the second recovery over a dead recovery that committed nothing.
A record save that fails after a started strand leaves that spawn uncounted, as it leaves the strand itself unrecorded today; the cap then allows one spawn more.
Rationale: Master is the caller that already loops on `card_amended`; the Webster row stays Go that runs Master, and the orchestrator is out of the loop.

### D13. A verbatim `PATTERN.md` entry line is measured at the plan gate

Decision: `planparser` gains the blocking check `pattern-entry-line-cap`: a line inside a card's fenced block that starts with `` - `PATTERN- `` and exceeds `pattern.MaxEntryLineChars` runes fails the card, naming the entry, the length and the cap, with the way forward to shorten the line or move the detail to the entry's background file.
An entry line carries backticks of its own, so no inline span holds one whole, and only fenced blocks are scanned.
`loom-template-plan.md` tells the plan writer that a prescribed addition to an entry line must leave it under the cap.
Bound: the check measures verbatim lines only; a prose instruction to append a sentence is the stencil's rule.

### Added after Discussion (parent, 2026-10-10): D8 classifier is an allow-list of plain words

Decision: D8's read-only classifier admits a lyx or git segment only when every word is plain (no quote, backslash, $, brace, glob character or leading ~); anything else is rejected and stays a correctness halt. The round-4 fixer's wording stands; Plan and code review check the rule's Acceptance rows.

Rationale: Rounds 1-4 each closed one bypass shape of a deny-list; the round-4 fix replaces the deny-list with an allow-list, which closes the class the judge named. A fifth Discussion round would only re-review that wording, which Plan review and code review cover.

## Constraints

- PATTERN-refusal-way-forward: every new or reworded refusal in webster, shed or loom (D1's Stuck reason and rebaseline refusals, D3, D6, D8's `ErrAuditNotAcceptable` way forward and `audit_not_acceptable` envelope, D12's `recovery_exhausted` and `begin-batch`'s terminal-dead remedy) names its way forward, lands its spec row in the same commit and is reached by a test; `fabriccli`'s D7 refusal is outside the bound modules and still names a way forward, and D7's rewording of every row and error text naming a refused fabric verb lands with it.
- PATTERN-no-denied-recovery: no way forward names a denied form; D12's names `lyx webster reset`.
- PATTERN-friction-capture: D12's `recovery_exhausted` refusal writes a friction note as every webster refusal does; D7's `fabriccli` refusal is outside the bound modules and writes none.
- PATTERN-sole-parsers: D2's hash compare and D13's line scan read bytes and raw card text, never a second parse of the plan.
- PATTERN-plan-generation: D1 changes nothing about rotation or archiving; a goto to Plan-Write still starts a new webster run.
- PATTERN-batcher-registry: D1's partition replacement is a rebaseline, the one allowed replacer.
- PATTERN-review-round: D2 skips no fix round and changes no convergence rule.
- PATTERN-transient-stop: D1's Stuck carries no transient mark.
- PATTERN-agent-name: D7's role allowlist cites the owning modules' constants and matches numbered roles through `agentname.MatchesRole`.
- PATTERN-shuttle-provider-seam: D9 changes `shuttleengine` only; `claudeengine`'s event parsing is untouched.
- PATTERN-completion-signal: D9's gated at-once branch and its deadline and liveness ends consult `allOutputFilesExist` before any negative verdict, and `completionsignal_enforcement_test.go`'s audited function set changes in the same commit as `expiredTurnEnd`'s rewrite.
- PATTERN-wait-mark-display-only: D9's marker change is display only.
- PATTERN-gitexec-checked-call: D4 adds no raw call site.
- PATTERN-gate-self-check-parity: D2's and D13's checks run in the gate closure and in `lyx loom validate-plan` through one function.
- PATTERN-stencil-ownership and PATTERN-producer-pointer-rule: D2's and D7's stencil text points at the rules' owners and duplicates no contract.
- PATTERN-test-economy, PATTERN-test-speed, PATTERN-test-isolation: new cases are rows in existing tables at the public surface; D7's guard rows set the strand name with `t.Setenv` and name that global state in place of `t.Parallel`, and `lyx gate test` and `verifytree.Verify` keep the inherited one from every other test; D11 stays in the integration tier.
- PATTERN-documentation-lifecycle and PATTERN-comment-line-breaks: `doc.go` of `websterengine`, `shuttleengine`, `fabriccli`, `loomshed` and `planparser` change in the same commits.
- Test budget: a card that pushes `websterengine`, `planparser`, `loomshed`, `fabriccli`, `fabricengine`, `shuttleengine` or `loomcli` past its row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.

## Auto-mode assumptions

- #476 is in scope: the board entry lists it among the task's issues, which close at landing, and its body carries the section.
- D1 rebaselines at the Webster row rather than resetting webster state at the goto, since a reset discards implemented work.
- D7 keys the fabric refusal on the strand's role with the driver and orch roles allowed, not on comparing remote URLs.
- D9 unifies on the payload rule, not the ten-minute expiry.
- D12 keeps Master as the caller of the second recovery.

## Open risks

- D9: a stale shell of either signal holds a run until its own timeout, where a transcript-reported one expired after ten minutes before.
- D1: after a crash, an edit to a card of the in-flight batch is adopted as an amendment on row entry without an operator naming it.
- D7 and D8 guard static shape and the exported strand name; a fork that clears the variable, types the hidden bypass flags on `push`, or hides a command behind `bash -c` is caught by review, not lyx.
- D8's plain-word rule rejects a quoted or globbed read such as `git log --format='%h %s'`, so that reference stays a correctness halt.
- D7 refuses the conflict resolver's role too; a landing whose resolver needed a fabric verb would meet the refusal, though its stencil forbids any git.
- D11 may not reproduce on a quiet machine; the card then documents the attempt and the dependencies it ruled out.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d12-begin-batch-dead-refusal-stale (class consistency, severity MEDIUM, open in rounds 4)
  - d8-reader-write-flags-unbounded (class design, severity BLOCKING, open in rounds 1, 2, 3, 4)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 3: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/plan/round-3-review.md; fixer report: _lyx/reviews/plan/round-3-fixer-report.md.
  - card14-classifier-rows-miss-glob-and-tilde (class scope, severity MEDIUM, open in rounds 3)
  - card4-merriambase-failure-escapes-stuck (class scope, severity MEDIUM, open in rounds 3)
  - thin-test-rows-cards-13-11-17-21 (class scope, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/webster/round-2-review.md; fixer report: _lyx/reviews/webster/round-2-fixer-report.md.
  - loop-agrees-stale-waited-out-comment (class consistency, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- A run with webster state, moved by goto to Plan-Bouncer, whose Plan loop adds a follow-up card after the last begun batch, continues at Webster over the kept batches with the new card batched, the restamped state is on disk before the first batch starts, and the run summary carries a warning naming the rebaseline.
- The same run whose Plan loop edits a done card blocks at Webster as Stuck with the rebaseline refusal, whose way forward names re-stepping the row and names neither `lyx webster run` nor `lyx webster rebaseline`, never `failed`; `lyx webster run` on that worktree still refuses with the fingerprint text.
- A begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside the Card Index each block the Webster row as Stuck with the rebaseline's refusal text; a transient rebaseline failure blocks it as Stuck too; no auto-path refusal text contains `lyx webster rebaseline`.
- An accepted auto-rebaseline followed by a stuck batch gives a Stuck reason that opens with the rebaseline warning.
- After a crash mid-batch, an edit to a card of the in-flight batch is accepted at row entry, recorded as an unrendered amendment, and the next recovery's prompt renders it.
- The plan gate reports `done-card-edited` for an edited done card and nothing for an unedited done card or an unbegun card, with a webster record, without one, and with a record that has no recorded hashes.
- The Plan-Review rubric renders `none` without a webster record and the begun batches, done cards and follow-up rule with one.
- `rebaseline` over a record with no frame hash logs the fallback and refuses naming the cause, once each for no recorded overview hash, an absent baseline copy and a copy without a Card Index, the last two naming the path.
- `commitsNamedBy` with a prefix naming a commit, a blob and a tree runs one batch-check and returns the commit; a `missing` line is an error.
- `verify-nested-module` and `verify-module-wide` find a `go` command after `||` and after `|`, `a || b` is one `||`, a separator inside quotes is text, `go test \` continued on the next line is one command, and a `cd` in a pipeline stage or after `||` ends the cross-module scan.
- A three-card batch beside unbegun cards: done-checks see the three cards only, and the failure reasons open with the batch id and its card list.
- `lyx fabric add`, `clone`, `push` and `sync` refuse under a webster, recovery or conflict strand name and run under the driver and slugged orch names, under a slug-free name, under an unparseable name and under no name; `list`, `pairs`, `shortname` with no argument, `prune` and `cleanup` without `--apply`, and `add --help` pass under every name; `prune --apply`, `cleanup --apply` and `shortname x` refuse under a webster strand name; a bypass-mode `push` carrying `--weft-path` runs under a webster strand name, and `add` or `clone` carrying `--warp-path` refuses under it; `lyx config`'s per-worktree write syncs under a webster strand name; every registered fabric subcommand is pinned reader or refused.
- `lyx gate test` and `verifytree.Verify` run with a webster strand name exported hand their processes no strand name, and the existing fabric tests pass under it.
- `lyx fabric add --help` states that the verb pushes to origin, and the implementer stencil states the sandbox rule.
- The classifier accepts `lyx fabric add --help`, `lyx fabric diff 1a2b3c`, `cat _lyx/fabric/origin.json`, `git -C x remote get-url origin`, `git ls-remote --heads`, `git branch -a` and a pipe of readers, and rejects `lyx fabric add x`, `lyx fabric add "x --help"`, `lyx fabric add -- --help`, `cat a > b`, `git push`, `git log --output=f`, `git ls-remote --upload-pack=x`, `git diff --ext-diff`, `git branch -D x`, `git config x y`, `env git status`, a command after `$(`, `lyx fabric add --branch --help`, `git log --out=f`, `git ls-remote --upload-pa=x`, `git ls-remote --exec=x`, `git remote -v add x u`, `git branch -v x`, `git log -n --output=f`, `git cat-file -p -t x`, `git -c core.pager=x log`, `git --git-dir=x log`, `git ls-remote --heads "--upload-pack=x"`, `git log '--output=f'`, `git log \--output=f`, `git log {--output=f,x}`, `git log $X` and `lyx fabric diff`.
- A fork transcript with only read-only fabric references records-batch done with audit warnings; one with `lyx fabric add` fails as today.
- recover-batch over a failed batch whose uncheckable entries are all read-only fabric references spawns a recovery and records the warnings; a non-read-only entry still refuses `needs_fresh`; a dirty tree refuses with the evidence text naming `recover-batch` and `audit_not_acceptable: true`, and the Master stencil's rung for it ends stuck.
- A fresh gated run with output files present and a transcript-reported shell outstanding, before the gate has evaluated any arrival, finishes at once with the shell in the record; after a gate evaluation, a pending `PassOnCap` answer included, and on an attached run, it waits on the shell; a gated waiting turn end with files present and nothing outstanding, a fork or an awaited-prefix shell is not an arrival; an ungated run with files present finishes at once whatever is outstanding; with an output file missing, both signals wait past `background_shell_wait_min`, log once and end `OutcomeTimeout` at the run deadline, and an awaited-prefix shell and a fork keep waiting.
- A recovery strand's turn end with a shell of either signal outstanding does not count; with its report present the batch classifies from the report.
- The run's end logs the shells the strand removal ended, and webster's summary lists them with signal and duration; an ungated at-once finish, a deadline end and a liveness end each record the shells outstanding at that end in `EndedShells`.
- `publishFailureCommand` for `TestA/sub_1` and `TestA/sub_2` in one package emits one `-run '^TestA$'` step; the note keeps both subtest paths.
- The resume-reconcile test fails with the named dependency forced and passes after the fix under repeated integration runs of `internal/loomcli`; when no run reproduces the failure, the card records the attempt and the dependencies ruled out.
- A dead recovery with commits past its `RecoveryStartSHA` reports `recovery_retry: true` and `recoveries: 1` on both `recover-batch` and `status`, and gets a second recovery; a dead recovery without its own commits, over a fork that committed, a record with no `RecoveryStartSHA`, a HEAD a reset moved off it, a range of merge-ins only and a dead recovery at two recoveries each report `recovery_retry: false`; a failed start leaves `recoveries` unchanged; a third counted spawn refuses `recovery_exhausted` with the named way forward and `recovery_exhausted: true`, while a `card_amended` re-run at two recoveries spawns uncounted; a re-begin and a rebaseline keep the count, a batch-start reset clears it and `reset --to start` drops it with the record; a failed git read gives `recovery_retry: false` without failing `status`; both Master dead rungs and `begin-batch`'s terminal-dead remedy name the retry, and a rung ends stuck on `recovery_exhausted`.
- A card whose fenced block holds a `PATTERN-` entry line over the cap fails `pattern-entry-line-cap` with the shorten-or-move way forward; one under the cap passes.

## Notes for the plan writer

- D1: `internal/websterengine/runlevel.go` (the fingerprint case of Run and `freshWarnings`), `rebaseline.go`, `internal/webstercli/rebaseline.go` for the deps constructor, `internal/shedadapters/webster.go` for the Stuck mapping, `internal/loomshed/webster.go` for the row.
- D2: `internal/shedadapters/rubric.go` and `bouncer.go` for the marker plumbing, `contracts/stencils/loom/loom-rubric-plan-review.md`, `internal/loomshed/gates.go` (`ValidatePlan`), `websterengine.DoneCards` and `BatchState.CardHashes`.
- D3 and D4: `internal/websterengine/rebaseline.go`, `gitwrap.go`, `internal/gitexec`.
- D5 and D13: `internal/planparser/verifymodule.go`, `internal/pattern/check.go` for the cap.
- D6: `internal/websterengine/recordbatch.go` (`postBatchChecks`, `failCardNotDone`).
- D7: `internal/fabriccli/fabric.go` and its persistent pre-run, `internal/configcli/hubwide.go` as the model, `internal/hubgeom/parent.go` for the nested-pair orch name, `internal/fabriccli/weft_verbs.go` for the bypass flags, `internal/configcli/configcli.go` for the nested sync, `contracts/stencils/webster/webster-body-implementer.md`.
- D8: `internal/fabricengine/refscanner.go`, `internal/websterengine/audit.go` (`ClassifyViolation`), `auditledger.go`, `recoverbatch.go`, `internal/webstercli` for the injected classifier.
- D9: `internal/shuttleengine/wait.go` (`expiredTurnEnd`, `ShellWaitExpires`), `run.go` (`Result.ExpiredShells`), `internal/websterengine/summary.go`, `triage.go`, `runlevel.go`, `strand.go`, `classify.go`.
- D11: `internal/loomcli/smoke_prebootstrap_integration_test.go`, `smoke_helpers_test.go`; dependencies to rule out first are the subprocess timeouts of a cold `lyx` under a loaded hub and `sharedLyxBinary`'s temp dir outside the test's own.
- D12: `internal/websterengine/state.go`, `recoverbatch.go` (`recoverSpawn`), `beginbatch.go` for the re-begin carry-over, `reset.go`, `internal/webstercli/reset.go` for the count clear, `internal/webstercli/recoverbatch.go` and `status.go` for the envelopes, `contracts/stencils/webster/webster-template-master.md`.
- D7's gate and verify env: `internal/gatecli/run.go`, `internal/verifytree/verify.go`.
- Spec rows: `contracts/specs/refusal-spec.md`'s webster section.
