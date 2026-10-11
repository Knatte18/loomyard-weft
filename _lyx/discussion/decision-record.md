# Decision record — twb-seat-blind-1

## Goal

Close six Webster faults that stop or mislead runs outside the audit, against the code at base `800a0e892`:
a background gate shell lost at Master's turn end, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, `recover-batch` exiting 0 on a failed recovery, gate commands the implementer prompt leaves in doubt, and Create checks that cannot pass for build-tagged or per-GOOS members.

## Scope

In:

- §1 (#505, #509, #517): Master's wait holds a running `lyx gate` shell; every record of a shell the run ended over says its result was discarded.
- §2: the verify gate's failure message and findings file point Master at the fixer fork; Master's stencil forbids running gates itself.
- §3 (#522): a recovery strand's writes count as the run's own; `recover-batch` exits non-zero on a terminal stuck or dead digest.
- §4: the shared implementer body names the always-allowed gate commands and leaves tmux- and llm-tagged steps to verify and Publish.
- §5 (#511): the `create-not-done` refusal says what was looked up and what quarry answered; resolution of unexported and `_test.go` members is pinned by a test.
- §6 (#500, #507): an ambiguous answer whose candidates sit in build-disjoint files is one member across `planglyph`, at Plan time and at record-batch.

Out:

- #507's other ask, a Create handle reusing a name every declaration of which the same card deletes: an independent design touching `createFindings`, `doneCheckVerdicts` and `deleteorder`. The orch files it as a board entry.
- #511's third ask, whether `lyx loom goto` into Webster revives a dead batch, and that halt's wording.
- Any quarry change.
- The fork-side gate-wait noise and `cd`/absolute-path shape, owned by board entry `fork-wait-noise`.
- `record-batch`'s exit code.

## Decisions

### §1 — background gate shells at Master's turn end

- **Premise at base.** `background_shell_wait_min` expires nothing. The cut-off is shuttle's `countsAtOnce` rule: a fresh gated run's first arrival with every output file present and only unawaited shells outstanding finishes Done, and strand removal ends the shell.
- **Decision: Master's spec awaits `lyx gate` shells.** Webster adds a `lyx gate` prefix beside `masterAwaitedShellPrefix` in `Spec.AwaitedShellPrefixes`. A held gate shell holds Master's turn end like a fork or a backgrounded `recover-batch`.
  Rationale: the result is read instead of lost; the rule stays caller data in `websterengine`, and `shuttleengine` stays generic.
  Bound: a held gate shell can keep Master's run alive until `master_timeout_min` or the liveness check, never past them; `lyx gate test`'s own slot wait (`cli_wait_sec`) bounds its start. A shell that does not match stays unawaited and keeps today's `countsAtOnce` behavior.
- **Decision: awaited-prefix matching is per command segment.** `shuttleengine`'s awaited-label test matches a prefix at the start of the label or at the start of any segment after `&&`, `||`, `;` or `|`, after leading whitespace.
  Rationale: `cd <dir> && lyx gate test …` must not slip through a bare `HasPrefix`.
  Bound: it widens which shells hold the turn end for every caller of `AwaitedShellPrefixes` (today only Webster's Master: `recover-batch` and `lyx gate`), still bounded by each run's own timeout and liveness. Environment-assignment prefixes (`FOO=1 lyx gate …`) do not match.
- **Decision: the record says the result was discarded.** `expiredShellWarning` (the run warning and the friction-note line) states that lyx never read the shell's result and its tests count as not run. The summary's "Background shells at the run's end" section is appended on every outcome where `summary.md` exists, not only on done.
  Rationale: a run that ends over any shell never reads as "tests passed".
- **Decision: Master's stencil never-list forbids running `lyx gate` or any test.** The verify gate and the fixer fork run them.
  Rationale: the advisor found no sanctioned Master gate run; the shells in #505/#509/#517 were Master off-script, tied to §2.

### §2 — Master fixing a verify failure itself

- **Cause.** `gateRepromptText` in `internal/shuttleengine/gate.go` sends every gated producer "read it, fix every finding, and end your turn"; it is the last instruction Master reads and contradicts the stencil's fixer-fork section.
- **Decision: `shuttleengine.GateEntry` gains an optional per-entry re-prompt.** An entry may supply its own one-line re-prompt built from the findings path; an entry without one keeps `gateRepromptText`.
  Rationale: the generic seam stays generic; only Webster's verify entry changes wording.
  Bound: the text must be one line (`validateSendText`); it changes what is sent, never when or how often.
- **Decision: Webster's verify entry re-prompt points at the fixer.** It names the findings file, says not to edit any file, and points at the stencil's "A gate failure: spawn one fixer fork" section and the verify-fix prompt file. It does not copy the fork prompt (`PATTERN-producer-pointer-rule`).
- **Decision: the verify-gate findings file opens with the same next step.** `renderVerifyGateFindings` leads with "spawn one fixer fork; do not edit"; its dirty-tree arm says the fixer fork commits or removes the paths, instead of addressing the reader.

### §3 — recovery attribution and exit code

- **Cause.** `loadRunWrites` audits `recordedSessions`: Master sessions and fork batches' `SessionID`. A recovery strand is its own Claude session, recorded nowhere, so its writes are unattributed and read "Not written by this run".
- **Decision: each recovery spawn persists its Claude session id on the batch record.** The ids form a list carried forward across recovery records the way `AuditWarnings` and `ForkTranscripts` are. `recordedSessions` includes them, so `loadRunWrites` attributes their writes as the run's own.
  Rationale: attribution by the run's sessions, which is attribution by batch, with no destructive reset.
  Bound: the same evidence feeds `PlanReset`'s own-path exemption (and fabric destroy's dirtiness check fed from it) and `contractWritesFor`. Those now also count a recovery's writes as the run's own: a reset may revert paths a recovery wrote, and a recovery's contract-file write is attributed. Paths no session of the run wrote stay foreign.
- **Decision: `recover-batch` exits non-zero on a terminal stuck or dead digest.** The envelope carries the same fields as today (`status`, `dead_reason`, `recovery_retry`, `recoveries`, `warnings`) through `output.ErrFields`. A done digest and a `--wait` running snapshot exit 0.
  Rationale: a failed recovery must not read as success in Master's completion notice; Master's ladder keys on the JSON, which is unchanged.
- **Decision: the recovery prefix needs no attribution wording change.** Its "Written by this run" group already says what to do.

### §4 — gate commands an implementer may run

- **Cause.** No deny blocks `lyx gate test` or `lyx loom lint-comments`. The body's sandbox rule ("never run a mutating `lyx fabric` verb") and the prefix's "never run `lyx webster`" sit beside Go-rendered gate steps with nothing saying gates are distinct, and the body's "a `tmux`-tier run goes through `lyx gate test`" reads as permission to run the tmux tier.
- **Decision: one short section in `webster-body-implementer.md`, after the sandbox rule.** The card and batch gate steps it lists, `lyx gate test` on the batch's packages and `lyx loom lint-comments` are always allowed and are not fabric or webster verbs. A tmux- or llm-tagged step is not run by an implementer: it compiles it with `go vet -tags <tag> <pkg>` and leaves the run to verify and Publish. The "Slot-busy exits and raw `go` runs" line is reworded to match.
  Rationale: forks and the recovery strand share the body, so the rule is stated once; `PATTERN-test-speed` runs the tmux tier at Publish and by hand.
  Bound: it removes no deny; the body names no raw module-wide or `go test -tags tmux` run (`PATTERN-no-denied-recovery`, `PATTERN-gate-slot`).

### §5 — `create-not-done` on unexported targets

- **Finding.** quarry resolves unexported and `_test.go` members (live: `cmd/testtiming#rusageStats`, `cmd/testtiming#runStandInGo` found). #511's refusals were #500's per-file ambiguity reported as "still does not resolve".
- **Decision: the `create-not-done` detail names the lookup.** It carries the glyph looked up (a `plan:` handle's glyph half), quarry's status, for not_found whether the unit exists, and for ambiguous each candidate's file with its build constraint and why the candidates are not build-disjoint. Each case names its way forward, and the refusal-spec "card not done" row is updated (`PATTERN-refusal-way-forward`).
- **Decision: no resolver change for visibility or test files.**

### §6 — build-tagged and per-GOOS members

- **Decision: `planglyph`'s single quarry resolve site rewrites a build-disjoint ambiguous answer to multipart.** When every pair of candidates sits in files with disjoint build constraints, the answer becomes multipart with the candidates as its symbols. Every rule reading that site follows: Plan-time status policy passes Edit and Uses; `createFindings` reports `create-already-exists`; DoneChecks pass a Create or Rename-new and treat Delete and Rename-old as still existing; `answerSymbols` returns every body, so delete-before-reference checks them.
  Rationale: multipart already means one glyph with several declarations the language allows; one site fixes every rule. quarry deliberately evaluates no build constraints and already returns each candidate's file.
  Bound: only an answer every candidate pair of which is proven disjoint is rewritten; an unproven or overlapping pair stays ambiguous and keeps blocking.
- **Decision: disjointness is decided on the candidate files and fails closed.** A file's constraint is its `//go:build` line (or legacy `// +build` lines when no `//go:build`) ANDed with its file-name GOOS/GOARCH constraint, parsed with `go/build/constraint`. Two files are disjoint when no assignment satisfies both: GOOS and GOARCH are exclusive groups over a closed pinned list, `unix` derives from GOOS, other tags are free booleans. An unreadable file, a parse failure, a `go1.N` or `ignore` tag, or a variable count above a fixed cap answers "not proven".
  Rationale: stdlib only, the worktree root told to the function (`PATTERN-told-geometry`), no Go-source parse outside build headers.
- **Decision: Rename of a build-split member stays refused, with a truthful message.** `renameDeclSource` requires found; its refusal says the member has one body per build constraint set and names the files.
- **Decision: the Plan validator adds an informational finding for a build-split member.** A new check id names the candidates and their constraints, so the author knows an Edit or Delete covers every body.
  Rationale: this is what Plan time can flag; a member a card creates resolves not_found before the card runs, so a split Create is carried by DoneChecks.

## Constraints

- Base `800a0e892`; commit `1e79825ff` and later are treated as not landed.
- `PATTERN-shuttle-provider-seam`: the awaited prefixes and per-entry re-prompt are provider-neutral caller data; nothing provider-specific moves out of `claudeengine`.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: stencil text stays under `contracts/stencils/webster`; Go-built one-line messages point at stencil sections and never copy the fork prompt.
- `PATTERN-refusal-way-forward`: the reworded `create-not-done` refusal and the Rename refusal name their way forward, with their `contracts/specs/refusal-spec.md` rows and a test.
- `PATTERN-no-denied-recovery` and `PATTERN-gate-slot`: no stencil names a denied command or a raw module-wide or tagged `go test`.
- `PATTERN-told-geometry`: the build-constraint classifier is handed the worktree root.
- `PATTERN-leaf-packages`: `planglyph`'s new imports are stdlib; check its admitted set.
- `PATTERN-test-speed`: the tmux tier runs at Publish and by hand, never in a card or batch gate.
- `PATTERN-test-economy` and `cmd/lyx/testdata/test-budget.yaml`: a card whose new top-level tests push `shuttleengine`, `websterengine`, `webstercli` or `planglyph` past its row lists that file under `Edit:`.
- Docs land in the same commit: `doc.go` of `shuttleengine`, `websterengine` and `planglyph`, `contracts/specs/loom-plan-spec.md` for the status policy and the new check, `contracts/specs/refusal-spec.md`, and `recover-batch`'s Long help.

## Auto-mode assumptions

- No operator answered; every pick was the chair's after weighing advisor `ly:twb-seat-blind-1:discussion-advisor-1`.
- The board's §1 premise (an expiry after `background_shell_wait_min`) does not hold at base; the record designs against base's `countsAtOnce` path.
- The §1 gate shells were Master's own, off its stencil; the run transcripts were not available to confirm it.
- The "terminal failed with exit 0" in #522 was a terminal stuck or dead digest, since the verb's failed path already exits non-zero.
- #511's `rusageStats` was first implemented per GOOS and later restructured; the base tree cannot reproduce the refusal.
- `AuditForks` reads a recovery strand's fork-less transcript and returns its parent writes; to be confirmed at Plan time.

## Open risks

- A gate shell that hangs now holds Master until `master_timeout_min`, where before it was cut off at the first arrival.
- Widening `PlanReset`'s own-path set to recovery writes lets a reset revert paths a recovery wrote, which is intended but newly reachable.
- The pinned GOOS/GOARCH list drifts from the Go toolchain; an unknown GOOS suffix reads as a plain name and an unlisted value fails closed (stays ambiguous).
- `record-batch` exits 0 on a stuck fork digest while `recover-batch` exits non-zero on a stuck recovery; the asymmetry is deliberate and must be stated in both verbs' help.
- #507's delete-and-recreate case stays refused until its own board entry lands.

## Acceptance criteria

- A fresh gated Master run whose turn end has output files present and a `lyx gate test` shell outstanding keeps waiting until the shell ends, the run's timeout or liveness; the same holds for `cd <dir> && lyx gate test …`.
- An unawaited non-gate shell at a fresh run's first arrival still finishes the run Done, as today.
- Every shell a Master run ends over is warned with a statement that its result was discarded, and the summary section appears on stuck and paused outcomes as well as done.
- A failed verify gate sends Master the verify entry's own re-prompt naming the findings file and the fixer-fork path; another gated producer still receives `gateRepromptText`.
- The verify-gate findings file opens with the fixer-fork next step, and its dirty-tree arm addresses the fixer fork.
- Master's rendered prompt forbids running `lyx gate` or tests.
- A second recovery of a batch whose first recovery left uncommitted writes lists them under "Written by this run"; a path no run session wrote stays under "Not written by this run".
- The recovery session ids survive a respawn and a re-begin of the batch.
- `reset --to start` after a stopped uncommitted recovery treats that recovery's tracked writes as the run's own.
- `recover-batch` exits non-zero with the digest fields on a stuck or dead terminal digest, and 0 on done and on a `--wait` running snapshot.
- The rendered fork and recovery prompts name the gate commands as always allowed and leave tmux- and llm-tagged steps to verify and Publish.
- A Create of an unexported member and of a member declared in a `_test.go` file passes DoneChecks.
- A `create-not-done` finding names the looked-up glyph, quarry's status, and for ambiguous each candidate file with its constraint.
- Per-GOOS files (`proc_linux.go`/`proc_windows.go`), a `!windows`/`_windows.go` pair and a `tmux || llm`/`!tmux && !llm` pair classify disjoint; `tmux && linux` against `tmux`, an untagged pair, and an unreadable or `ignore` file classify not proven.
- A build-disjoint member passes Create at record-batch, Edit and Uses at Plan time, reports `create-already-exists` for a Create at Plan time, blocks Delete until every body is gone, and its delete-before-reference check sees every body.
- A Rename of a build-split member is refused with a message naming the files; the Plan validator reports the informational build-split finding.

## Notes for the plan writer

- §1 code: `internal/websterengine/runlevel.go` (`masterAwaitedShellPrefix`, Master's `Spec`, `finishMasterDone`), `triage.go` (`expiredShellWarning`), `summary.go` (`AppendBackgroundShells`), `internal/shuttleengine/wait.go` (`awaitedLabel`, `countsAtOnce`).
- §2 code: `internal/shuttleengine/gate.go` (`GateEntry`, `gateRepromptText`), `internal/websterengine/verifygatereport.go` (`renderVerifyGateFindings`), the verify entry built in `runlevel.go`, `contracts/stencils/webster/webster-template-master.md`.
- §3 code: `internal/websterengine/runwrites.go` (`recordedSessions`), `recoverbatch.go` (`recoverSpawn`, `uncommittedPathsBlock`), `state.go` (`BatchState`), `reset.go` (`ownTrackedPaths`), `contractevidence.go`, `internal/webstercli/recoverbatch.go`.
- §4: `contracts/stencils/webster/webster-body-implementer.md`; gate steps rendered by `cardgate.go`; render tests pin the stencil text.
- §5–§6: `internal/planglyph/repo.go` (`resolveTargets`, needs the worktree root threaded from `openRepo`'s callers), `donecheck.go`, `create.go`, `resolve.go`, `deleteorder.go` (`answerSymbols`), `handle.go` (`renameDeclSource`); a tagged fixture exists under `internal/planglyph/testdata`.
- `internal/gatecli#runChild` does not exist at base; use `internal/proc#StartTime` as the live per-GOOS example.
