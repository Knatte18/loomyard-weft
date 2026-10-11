# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close the Webster faults outside the run-exit audit that stop or mislead runs, the board entry's items 1 to 6.

1. A background `lyx gate test` shell still running at Master's turn end is waited for, and when the wait gives up, the run records that its result was discarded.
2. On a verify-gate failure, Master is told to spawn the verify fixer fork in the message it acts on.
3. A second recovery of a batch counts the first recovery's uncommitted files as the batch's own work, and a terminal non-done `recover-batch` exits non-zero.
4. The recovery prompt names the gates the strand always runs and the tagged steps it leaves to the verify tiers.
5. A `create-not-done` refusal says how the target was looked up and what quarry answered.
6. A member declared once per mutually exclusive build-constraint set counts as one member at Plan time and at record-batch.

The record faces the code at base `800a0e892`, with quarry at `v0.2.1`.

## Scope

In:

- `internal/shuttleengine`: matching awaited shell prefixes, a bounded hold for gate shells, and a caller-supplied re-prompt per gate entry.
- `internal/websterengine` and `internal/webstercli`: Master's awaited prefixes, the discard record for a gate shell, the verify gate's re-prompt, recovery write attribution, and the `recover-batch` exit code.
- `contracts/stencils/webster/`: the Master template's gate-failure and recover-batch rungs, and the recovery prefix's gate section.
- `internal/planglyph`: the build-split reading of an `ambiguous` answer, the `glyph-build-split` finding, and the `create-not-done` detail.
- `contracts/specs/refusal-spec.md` and `contracts/specs/loom-plan-spec.md` rows for every changed refusal and check.
- The `doc.go` of each touched package.

Out:

- Quarry itself: it stays build-constraint-blind, and lyx reads the candidates' constraints.
- A Create handle reusing a name whose every declaration the same card deletes (#507's first option).
  A card that deletes every per-GOOS declaration and creates the name again stays refused at Plan time; the plan spells that change as an Edit of the build-split member (Decision 9).
- Reviving a dead batch through `lyx loom goto` (#511's third point).
- How forks wait on their own gates, and the noise of interim fork notices: owned by `fork-wait-noise`.
- Child-process lifetime in `internal/proc`: owned by `proc-child-lifetime`.

## Decisions

### 1. A gate shell holds Master's turn end, up to a bound

Master's spawn declares `lyx gate test` as an awaited shell prefix beside `lyx webster recover-batch`, with a hold bound.
A gated run's turn end with every output file present keeps waiting while such a shell is outstanding, instead of finishing at once through `countsAtOnce`.
The hold ends at the first of:

- a later turn end, which the shell's completion notification starts when it reaches Master;
- the hold bound, a new shuttle config key `gate_shell_hold_min`, measured from the shell's first sighting;
- the run's deadline or the liveness check.

When the hold bound passes, the turn end counts as an arrival and goes through the gate as today.
The `recover-batch` prefix keeps its unbounded hold, which `recovery_timeout_min` already bounds.
Rationale: today a fresh gated turn end whose outstanding tasks are all unawaited shells finishes at once.
Shuttle then removes the strand and the gate shell dies unread (#505, #509, #517).
A shell a fork left running is listed in Master's turn end (corpus case `master-fork-running`), but nothing shows its completion starts a Master turn.
The bound keeps such a run from idling to its deadline.
Bound: the hold delays a run by at most `gate_shell_hold_min` past the shell's first sighting.
It never lets a run through that today would wait.
Ungated runs keep counting such a turn end at once.

### 2. A gate shell cut off at the run's end is recorded as discarded

Every shell on `Result.EndedShells` whose label matches the gate prefix is a discarded gate run.
That covers a run ended by the hold bound, the deadline or a liveness finalize.
At the deadline, `classifyDeadlineExpiry` still answers `done` when the output files exist, per `PATTERN-completion-signal`.
For each such shell, the existing `webster-background-shell` friction note, the `RunResult.Warnings` entry and summary.md's "Background shells at the run's end" section say:

- the shell was a gate run still running when the run ended;
- its result was discarded and never observed;
- the plan-level verify at HEAD is the run's test verdict.

The outcome is unchanged.
Rationale: a `done` run has already passed the plan-level verify gate at HEAD, so `done` is not a false test verdict.
The board's fault was a discarded result that nothing named.
lyx kills no shell, since the provider reports no pid; strand removal ends it, as today.

### 3. Awaited prefixes match at the start of any command segment

`awaitedLabel` matches a prefix at the start of the label, and also at the start of any segment that follows `&&`, `||`, `;` or a newline, after leading whitespace.
The shell's label is its command text (`payloadTask`, `launchKindAndLabel`).
Rationale: a gate run is spelled both `lyx gate test … | tail` and `cd <pair> && lyx gate test …`.
Bound: a label that carries the prefix after a separator inside a quoted argument also matches.
It is then held up to Decision 1's bound and recorded by Decision 2's wording.
No shell is reaped or skipped because of the match.
The prefix set must follow the gate spelling that `fork-wait-noise` settles on.
A gate wrapped in a command that does not start a segment with `lyx gate test` is not matched.

### 4. A gate entry may carry its own re-prompt text

A `GateEntry` gains an optional caller-supplied re-prompt, a function of the findings path that returns one line.
An entry without one keeps `gateRepromptText`'s generic line.
A caller re-prompt that returns an empty or multi-line string is a returned gate error, a caller defect, never a fallback to the generic line.
Rationale: on a verify failure, Master receives the generic line, "read it, fix every finding, and end your turn".
That line contradicts the Master template's "Do NOT localize or fix the failure yourself", and in `driver-context` Master fixed the failure itself.
Bound: the field changes only the text sent.
It changes no gate verdict, attempt count or deadline.

### 5. Webster's verify gate re-prompt sends Master to the fixer fork

Webster's verify-gate entry supplies a re-prompt that does four things:

- names the findings file;
- forbids Master from editing or committing;
- tells Master to spawn the one verify fixer fork per the template section `A gate failure: spawn one fixer fork`;
- tells Master to end its turn.

The template section opens with the same rule.
Rationale: the rule now sits in the one line Master acts on.

### 6. Recovery writes are run writes

`recover-batch` records each recovery strand's Claude session id on the batch record.
It records the id at spawn, once the strand's session is known, and persists it with the spawn's state commit, before any stop or reclaim can reach the strand.
The record carries the ids across respawns as it carries `Recoveries`.
`loadRunWrites` audits those sessions too, and their writes count as written by this run.
A recorded recovery session whose transcript cannot be read adds no evidence, so its paths read as foreign, and a warning names the session.
Rationale: today `loadRunWrites` reads only `State.MasterSessionID` and each batch's `SessionID`.
A recovery strand is neither, so its uncommitted edits land under "Not written by this run" in the next recovery's prompt (#522).
Consequences:

- `uncommittedPathsBlock` lists the first recovery's files under "Written by this run".
- `ownTrackedPaths` counts them as the run's own, so a reset no longer refuses over them as foreign.
- `contractFileStatus` reads Master and fork writes only, so a recovery's writes never clear a contract-file finding.

Bound: only sessions that `recover-batch` itself spawned and recorded are added.
A path written by no recorded session still counts as foreign.

### 7. A terminal non-done `recover-batch` exits non-zero

Every call that returns a terminal digest with `status: stuck` or `status: dead` exits non-zero, a re-entrant call included.
Its envelope carries the same digest fields as today (`status`, `dead_reason`, `recovery_retry`, `recoveries`, `warnings`) beside the error message.
`status: done` and the `--wait` running snapshot still exit 0.
The message names the way forward: run `lyx webster recover-batch NN` again when `recovery_retry` is true, otherwise the batch is exhausted and the run ends stuck.
The refusal gets its row in `refusal-spec.md`.
The Master template's recover-batch rungs say a stuck or dead digest arrives as a non-zero exit carrying the same fields.
Rationale: a caller or operator reading the exit code saw success for a failed batch.
Bound: no Go caller branches on `recover-batch`'s exit code; Master reads the envelope fields, which are unchanged.

### 8. The recovery prompt names its gates

The recovery prefix gains a section stating:

- The card gates, the card's `**Verify:**` steps and the batch gate in this prompt are always the strand's to run.
  `lyx gate test` and `lyx loom lint-comments` are not fabric verbs, and the sandbox rule's ban covers mutating `lyx fabric` verbs only.
- A step tagged `tmux` or `llm` is not run by the strand, which names each skipped step in its final reply.
  The section points at `loom-plan-spec.md`'s `verify model` section for which gate runs those tiers, and does not restate it.

Rationale: the `quick-recipe` recovery could not tell whether gates fell under the fabric ban, and could only compile-check tmux-tagged steps.
The board names the `tmux` tier.
The `llm` tier is added because the verify model already keeps it out of every automatic gate.
Bound: only steps carrying the `tmux` or `llm` tag are skipped.
Every untagged and `integration` step stays mandatory.
The in-session fork's prompt is unchanged.

### 9. The `create-not-done` detail says how the target was looked up

The detail names:

- the ref and the glyph actually resolved (a `plan:` handle's member-glyph half);
- that quarry resolved it against the worktree's current tree;
- quarry's answer: `not_found` with whether the unit was found, or `ambiguous` with each candidate's file.

The `rename-not-done` new-side detail gets the same answer text.
Rationale: quarry already resolves unexported members and members of `_test.go` files, so no resolver change is in scope.
The detail lets an operator tell a missing member from a build split.

### 10. A build-split ambiguous answer counts as one member

`planglyph` gets one predicate that reads an `ambiguous` answer as build-split when both of these hold:

- every candidate lies in a different file of the same unit;
- no single build configuration selects two of those files.

A configuration is one GOOS, one GOARCH and any set of other tags.
GOOS implications follow go/build: GOOS `android` also satisfies `linux`, `ios` also satisfies `darwin`, and `illumos` also satisfies `solaris`.
A `unix` term in a `//go:build` line holds exactly for go/build's Unix GOOS set.
A file's constraint is its `//go:build` line together with its file-name suffixes, read by go/build's rules:

- everything up to the first `_` of the name is not a suffix, so `linux.go` constrains nothing;
- `_test` is stripped before the suffixes are read;
- `_GOOS`, `_GOARCH` and `_GOOS_GOARCH` constrain the file;
- `unix`, `other` and any non-GOOS, non-GOARCH word constrain nothing.

Every planglyph reader of a resolve status treats a build-split answer as it treats `multipart`:

- Plan-time Edit, Delete and `Uses:` targets pass.
- A Create target that is already build-split is `create-already-exists`.
- At record-batch, a Create target or a Rename new side that resolves build-split is done.
- A Delete target or a Rename old side that is still build-split is not done.
- An Edit target's candidate files all belong to the batch's deviation union.

A Rename pair's old side stays found-only, as `renameDeclSource` requires today.
An `ambiguous` answer that is not build-split keeps today's blocking findings at both times.
Rationale: #500 and #507 are one fault, quarry answering `ambiguous` for one Go member split by build constraints.
One predicate shared by Plan time and record-batch means a glyph Plan accepts is one record-batch can pass.
Bound: the predicate fails closed.
These read as not build-split, so the target keeps today's blocking finding:

- a candidate file that cannot be read;
- a `//go:build` line that does not parse;
- a file with a `// +build` line and no `//go:build` line;
- two candidates in one file;
- candidates in different units.

### 11. Plan time flags a build-split target

A plan-time Edit, Delete or `Uses:` target that resolves build-split gets an informational finding, `glyph-build-split`, naming each candidate file.
A Create cannot be flagged at Plan time, since its declarations do not exist until the card runs; Decision 10 accepts it at record-batch instead.
Rationale: the board asks that such a glyph be flagged at Plan time rather than first met at record-batch.
The finding tells the plan writer and the implementer that the member has one declaration per listed file.
It is informational because the plan is coherent.

## Constraints

- `PATTERN-shuttle-provider-seam`: Decision 3's segment matching is shell syntax over a provider-neutral label, so it lives in `shuttleengine`; provider payload reading stays in `claudeengine`.
- `PATTERN-completion-signal`: no negative "did this run finish" answer changes, and Decision 2 relies on the deadline's `done` answer.
- `PATTERN-refusal-way-forward`: Decision 7's exit and Decision 9's changed detail land with their `refusal-spec.md` rows and a test reaching each.
- `PATTERN-stencil-ownership`: stencil text changes in `contracts/stencils/webster/` and is read at call time.
  Decision 5's line is a one-line send, like `gateRepromptText`, not a producer prompt.
  It names the template section it defers to, and the section carries the rule.
- `PATTERN-producer-pointer-rule`: Decision 8's section points at the plan spec's verify model and does not paraphrase it.
- `PATTERN-glyph-conversion-chokepoint`: Decision 10's same-unit comparison derives units only through `glyph`'s conversion functions.
- `PATTERN-no-denied-recovery`: the recovery section names only commands the strand's settings allow.
  `lyx gate test --tags tmux` is allowed; Decision 8 skips it by choice.
- `PATTERN-sole-parsers`: no new plan, discussion, summary or recipe parser.
  Decision 10 reads build constraints with the standard library's `go/build/constraint`.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend existing table tests where one covers the surface.
  Label matching and the hold are tested on shuttle's wait path in `shuttleengine`, not through a `claudeengine` corpus case.
- Test budget: a card whose new top-level tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Docs in the same commit:
  - `shuttleengine/doc.go`: awaited prefixes, the hold bound and the gate re-prompt;
  - `shuttleengine/template.yaml`: the new key;
  - `websterengine/doc.go`: background shells in Master's wait, recovery write attribution and the `recover-batch` exit;
  - `planglyph/doc.go`: build-split and `glyph-build-split`;
  - `loom-plan-spec.md`'s check list.

## Auto-mode assumptions

- Hold gate shells up to a bound, and record a cut-off one as discarded (Decisions 1, 2).
- No new verb for the verify fixer (Decision 5).
- Attribution by batch for recovery writes (Decision 6).
- Build constraints read in lyx, not quarry (Decision 10).
- `glyph-build-split` is informational (Decision 11).
- The tmux and llm skip is in the recovery prompt only (Decision 8).
- `gate_shell_hold_min` defaults to 20, longer than a typical integration-tier gate run.

## Open risks

- A run with a gate shell that never reports its end now waits up to `gate_shell_hold_min` before finishing.
- The tmux tier a recovery skips runs only at Publish's `publish_verify`, where the repo sets it.
- Decision 10's GOOS, GOARCH and Unix lists can fall behind the Go toolchain.
  A new GOOS suffix then reads as no constraint, which fails closed.
- A recovery from a binary older than Decision 6 has no recorded session, so its writes stay foreign.
- `weaver` plans to reuse the recovery strand as its normal path, so Decisions 6 to 8 become its defaults.

## Acceptance criteria

- A gated run started fresh, whose turn end has every output file present and one outstanding shell labelled `lyx gate test --tags integration ./x | tail -5`, keeps waiting.
  A later turn end with nothing outstanding finishes it `done` before the hold bound.
- The same holds for the label `cd /abs/pair && lyx gate test ./x`.
- With no later turn end, the run finishes through the gate once `gate_shell_hold_min` passes.
  The shell then shows on `Result.EndedShells` and in webster's warning and summary as a discarded gate run.
- A deadline reached during the hold finishes `done` when the output files exist, with the same discard record.
- A gated run whose only outstanding shell is unrelated (`sleep 600`) still finishes at once.
- An ungated run with an outstanding gate shell and every output file present still finishes at once.
- A failed verify gate sends Master webster's re-prompt line, which names the findings file and the fixer fork and forbids editing.
  A gated run with no caller re-prompt receives the generic line.
  A caller re-prompt that returns a multi-line string is a gate error.
- A first recovery reclaimed mid-work by a `lyx webster run` restart leaves uncommitted edits.
  The second recovery's prompt lists those paths under "Written by this run".
- A path no recorded session wrote stays under "Not written by this run".
- A recorded recovery session whose transcript is missing adds a warning, and its paths read as foreign.
- A recovery's writes never clear a contract-file audit finding.
- `recover-batch` exits non-zero on a terminal `stuck` or `dead` digest, with `status`, `dead_reason`, `recovery_retry` and `recoveries` on the envelope.
  A re-entrant call on that batch exits the same way.
  It exits 0 on `done` and on a running snapshot.
- The rendered recovery prompt names the gates as always allowed and names the tmux and llm skip.
  The fork prompt renders unchanged.
- A Create of `x#m` passes record-batch when `m` is declared in `a_linux.go` and `a_windows.go`.
  It also passes when one file has `//go:build tmux || llm` and the other `//go:build !tmux && !llm`.
- A Create of a member declared in any of these pairs still fails `create-not-done`, with a detail naming quarry's `ambiguous` answer and both files:
  - `a.go` and `b.go` with no constraints;
  - `a_unix.go` and `a_linux.go`;
  - `a_linux.go` and `a_android.go`.
- A Create target that resolves `not_found` fails with a detail naming the resolved glyph and whether the unit was found.
- At Plan time, an Edit and a Delete of a per-GOOS member pass with an informational `glyph-build-split` finding.
  At record-batch, an Edit touching both of its files reports no deviation for either file.
  A Delete whose target is still build-split after the batch fails `delete-not-done`.
- A candidate with an unparsable `//go:build` line, or with only a `// +build` line, reads as not build-split.

## Notes for the plan writer

- Master's awaited prefix lives in `internal/websterengine/runlevel.go` (`masterAwaitedShellPrefix`), and the matching in `internal/shuttleengine/wait.go` (`awaitedLabel`, `countsAtOnce`, `expiredTurnEnd`).
- The discard wording sits beside the existing `webster-background-shell` note, the `RunResult.Warnings` entry and `AppendBackgroundShells`.
- The generic re-prompt is `gateRepromptText` in `internal/shuttleengine/gate.go`.
- Recovery attribution: `internal/websterengine/runwrites.go` (`loadRunWrites`, `recordedSessions`), `recoverbatch.go` (`uncommittedPathsBlock`), `reset.go` (`ownTrackedPaths`), `contractevidence.go` (`contractFileStatus`).
- The `recover-batch` terminal envelope is built in `internal/webstercli/recoverbatch.go`.
- The status readers in `planglyph` are `statusFindings` (`resolve.go`), `createFindings` (`create.go`), `answerSymbols` (`deleteorder.go`) and `doneCheckVerdicts` (`donecheck.go`).
