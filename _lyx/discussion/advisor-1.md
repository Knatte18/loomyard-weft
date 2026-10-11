# Advisor 1 notes — twb-seat-blind-1

All code facts are read at base `800a0e892` (`git show 800a0e892:<path>`), except the quarry facts, which come from the external module `quarry v0.2.1` (same version at base and at HEAD) and from live `lyx quarry resolve` runs.
The working tree is past base; nothing below relies on it.

## Board

- The task is one entry, `twb-seat-blind-1`, with six numbered sections.
  `twb-seat-lead-1` is the same text in a sibling pair; ignore it.
- `fork-wait-noise` (Ready) touches section 1's neighbourhood: it wants forks to wait for their gate inside the turn so Merriam gets one notice per fork, and edits the same stencils (`webster-template-master.md`, `webster-body-implementer.md`).
  It is not this task; a design that changes how forks run gates must not contradict it.
- No other entry touches quarry ambiguity, recovery attribution or the Create check.

## Section 1 — background gate shell cut off (#505, #509, #517)

The board text ("after `background_shell_wait_min` the wait stopped") no longer describes base.
At base the threshold expires nothing (`ShellWaitBound`, `wait.go`; `template.yaml`; `shuttleengine/doc.go` "A turn end that leaves background work outstanding").
The surviving path is `countsAtOnce` in `internal/shuttleengine/wait.go`:
a gated run (Webster always is, through the `verify` gate), started fresh, with no gated arrival yet, whose Master turn end lists only unawaited shells and has every output file present, finishes Done at once.
Strand removal in finalize then ends the shell.
That rule exists so a shell a fork left behind (a server, `sleep`, `tail -f`) cannot hold Master forever.
What base already does with such a shell: `Result.EndedShells`, the `webster-background-shell` friction note, a `RunResult.Warnings` entry per shell, and summary.md's "Background shells at the run's end" section (done-only).
The warning text says lyx did not stop the shell and names the outcome; it never says the shell's result was discarded, and the outcome stays `done`.

Modules: `internal/shuttleengine` (wait loop, `Spec.AwaitedShellPrefixes`), `internal/websterengine` (`masterAwaitedShellPrefix = "lyx webster recover-batch"`, `triage.go` warning text, `summary.go`, `doc.go`), `internal/shuttleengine/claudeengine` (shell label = the Bash `command`, else description, else id).

Options seen:
- Await gate shells like a fork: Webster adds a gate prefix to `Spec.AwaitedShellPrefixes`.
  Cheap and uses the existing seam, but matching is `HasPrefix` on the command: `cd <abs> && lyx gate test …` or a leading env assignment misses.
  The stencils would have to fix the command shape (also what `fork-wait-noise` wants: no `cd`).
  A hung gate shell holds until the run deadline or liveness check, not forever.
- Keep counting at once, but make the record say the result was discarded, and optionally demote `done`.
  Demotion would turn every leftover-server case into stuck, so it needs a way to tell a gate shell from a leftover.
- lyx killing the shell: the provider reports no pid (`doc.go` says lyx kills no shell), so only strand removal ends it.

## Section 2 — Master fixes a verify failure itself

Where Master reads a verify failure at base:
- The re-prompt is `gateRepromptText` in `shuttleengine/gate.go`: "Gate findings recorded at %s — read it, fix every finding, and end your turn."
  It is generic across every gated producer and tells Master to fix, which contradicts the stencil's "Do NOT localize or fix the failure yourself" (`webster-template-master.md`, section "A gate failure: spawn one fixer fork").
- The findings file text is `renderVerifyGateFindings` (`websterengine/verifygatereport.go`).
  It names no next step.
  Its clean-tree arm says "Commit or remove these paths", which is also an instruction to Master to act itself.
- The stencil rule exists in its own section and in the "What you never do" list; the fork spawn prompt is `{{.verify_fix_prompt_path}}`, rendered by Go at run entry (`RenderVerifyFixPrompt`).

Options: per-entry re-prompt text on `GateEntry` (generic seam, Webster's `verify` entry sets it); the next-step sentence in the findings file; both.
A verb that spawns the fixer cannot work as a fork: forks are in-session Agent-tool calls, so a verb could only print the prompt or start a cold strand (a new recovery-like path).
The run-exit audit already catches the self-fix after the fact (`forkaudit.go`); an early block would be a provider hook, a different feature.

## Section 3 and 4 — recovery attribution, exit code, allowed gates (#522)

Root cause of "Not written by this run": `uncommittedPathsBlock` (`websterengine/recoverbatch.go`) classes a path as own only if `loadRunWrites` saw a write to it.
`loadRunWrites` (`runwrites.go`) audits `State.MasterSessionID` plus each batch record's `SessionID` — Master sessions and their forks.
A recovery strand is a separate Claude session whose id is in neither, and `BatchState.SessionID` is documented empty for a recovery batch.
So the first recovery's writes are never evidence, and the second recovery sees them as foreign.
Attribution is by shuttle session, not by shuttle run id as the board says; the effect is the same.
`recoverSpawn` builds a fresh `BatchState` per spawn and carries only some fields forward (warnings, suspect paths, fork transcripts, amended cards, `Recoveries`), so any recovery session record must be carried the same way.

Options: record each recovery strand's session id on the batch record, carried forward, and add those sessions to `recordedSessions`; or treat every uncommitted path at a recovery respawn of the same batch as the batch's own (coarser, no new state; wrongly claims a path an operator edited); or reset to `RecoveryStartSHA` and offer the diff (destruction goes through fabric's chokepoint, `PATTERN-fabric-destruction-chokepoint`, and loses the first attempt's work).

Exit code: `webstercli/recoverbatch.go` ends a terminal digest with `output.Ok` whatever `Digest.Status` is (`done`, `stuck`, `dead`, `failed`).
`batch_failed`, `needs_fresh`, `recovery_exhausted` already exit non-zero through `ErrFields`.
The gap is a terminal non-done digest.
Master's stencil reads the envelope's `status` and `recovery_retry`, so a non-zero exit must keep the same JSON fields.

Allowed gates: the recovery prefix (`webster-prefix-recovery.md`) says "NEVER run any `lyx webster` command"; the shared body says "never run a mutating `lyx fabric` verb".
Neither names `lyx gate test` or `lyx loom lint-comments`.
The body's batch gate and "Slot-busy exits and raw `go` runs" sections already cover gates; tmux-tier handling for a recovery is the gap.

## Section 5 — Create check on unexported targets (#511)

Live check against quarry v0.2.1: unexported members (`cmd/testtiming#rusageStats`) and members declared in `_test.go` files (`cmd/testtiming#TestRenderCensus`, `#fixtureTmuxTests`) resolve `found`.
So the resolver does not have the limit the board suspects.
`DoneChecks` (`planglyph/donecheck.go`) renders a Create failure as `Create target %q still does not resolve` for every non-found answer, including `ambiguous` and `not_found`; the refusal names no status, unit status or candidates.
Most likely both #511 examples were ambiguous answers (a per-GOOS or per-tag pair) reported as "does not resolve", which makes #511 the same root cause as section 6.
At base `rusageStats` is declared once (`resources.go`) and the per-GOOS pair in that package is `maxRSSBytes` (`rusage_linux.go`, `rusage_other.go`); `tmuxAllowed` is declared once in `allow_none.go`.
I cannot see the tree the failing runs had.

The refusal is a webster refusal: `PATTERN-refusal-way-forward` needs a way forward in the message, a row in `contracts/specs/refusal-spec.md` and a test.

## Section 6 — build-tagged and per-GOOS members (#500, #507)

Quarry deliberately does not evaluate build constraints (`internal/engine/resolve.go` `statusForMatches`): a name declared in several places is `ambiguous` with `Candidates` carrying each declaration's `File`, except an owner-less `init`, which is `multipart`.
Verified live: `internal/proc#StartTime` is `ambiguous` with `proc_linux.go` and `proc_windows.go`.
`internal/gatecli#runChild` is `not_found` in the current tree (no longer a pair), so that example cannot be re-checked here.

Where ambiguity bites in `internal/planglyph`:
- `donecheck.go` `doneCheckVerdicts`: Create and Rename-new need `found` or `multipart`; Delete and Rename-old treat anything but `not_found` as still existing.
- `resolve.go` `statusFindings`: any `ambiguous` target is the blocking `glyph-ambiguous` at Plan time, so an Edit, Use, Delete or Rename of an existing per-GOOS member can never pass Plan validation.
- `create.go` `createFindings`: an `ambiguous` Create target is blocking `create-already-exists`.
- `BindHandles` matches `delta.Created` by `Symbol.ID` in a set, so duplicate IDs do not break binding.

The candidates carry a file path, so the disjointness test needs the file text: `//go:build` lines plus the `_GOOS`/`_GOARCH` filename suffix, evaluated with stdlib `go/build/constraint` over the tags that appear.
That is lyx-side work in `planglyph`; quarry offers no hook.
One shared classifier used by all three passes keeps them from drifting.
The Plan-time validator cannot know a Create will later be declared twice; it can flag a card or plan that spells the same glyph in two Create groups, and it can stop reporting existing disjoint pairs as `glyph-ambiguous`.

## PATTERN entries the design touches

- `PATTERN-refusal-way-forward`: any new or reworded webster refusal (sections 3 and 5).
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`: stencil text for Master, recovery, fixer pointers; point at the owning section, do not copy the fork prompt.
- `PATTERN-no-denied-recovery`: the recovery prompt may name `lyx gate test` and `lyx loom lint-comments`; it must not name a denied form (`cmd/lyx/norecoverydeny_test.go` scans `contracts/stencils/`).
- `PATTERN-shuttle-provider-seam`: shell-label and payload details stay under `claudeengine`; Webster supplies prefixes as caller data.
- `PATTERN-gate-slot`: gate commands run only in `gateslot`; the prompt wording must not suggest raw `go`.
- `PATTERN-friction-capture`, `PATTERN-sole-parsers` (plan parsing stays in `planparser`; the new classifier reads Go source headers only), `PATTERN-glyph-conversion-chokepoint` (do not convert glyph text outside `glyph`), `PATTERN-leaf-packages` (a new import in `planglyph` must fit its admitted set; I did not find `planglyph` named in the background file).
- `PATTERN-test-economy` and `cmd/lyx/testdata/test-budget.yaml`: new top-level tests in `shuttleengine`, `websterengine`, `planglyph` or `webstercli` may exceed their rows; CLAUDE.md says the card lists that file under `Edit:`.
- Docs in the same commit: `doc.go` of `shuttleengine`, `websterengine` and `planglyph`, and `contracts/specs/refusal-spec.md`.

## Conflicts seen

- Board section 1 describes an expiry that base does not have; the task text and base disagree, and the record should describe base.
- Board section 5 suspects an unexported or test-file limit; quarry resolves both.
- No conflict with a PATTERN entry found so far.
