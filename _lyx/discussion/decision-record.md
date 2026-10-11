## Goal

Close the Webster faults outside the audit that stop or mislead runs: a run finishing `done` over an unfinished background gate shell, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, a recovery prompt that does not name its allowed gates, a recovery exit code that hides a terminal failure, Create done-checks that fail on unexported or test-file members, and build-tagged or per-GOOS members that resolve `ambiguous` and can never pass.
The code is read at base 800a0e892; 1e79825ff and everything after it counts as not landed.

## Scope

In scope, one change per numbered fault of the board entry:

1. Background gate shell cut off at a turn end (#505, #509, #517): `internal/shuttleengine` wait and finalize, plus how the run record and Webster's outcome surface a discarded shell.
2. Master fixing a verify failure itself: the `Gate findings recorded at …` message text in `internal/shuttleengine/gate.go` and the Master stencil `contracts/stencils/webster/webster-template-master.md`.
3. Recovery attribution (#522) and the `recover-batch` exit code: `internal/websterengine/runwrites.go`, `recoverbatch.go`, `internal/webstercli` recover verb.
4. The recovery prompt's allowed gates: `contracts/stencils/webster/webster-prefix-recovery.md` and its claims test.
5. Create done-check resolution and refusal text (#511): `internal/planglyph/donecheck.go`.
6. Build-constrained and per-GOOS members (#500, #507): `internal/planglyph` DoneChecks and the plan-time resolve status policy.

Out of scope: changing the quarry dependency itself, any change to `background_shell_wait_min` semantics beyond what item 1 needs, a new Webster verb for spawning fixers, and the audit work the board entry excludes.

## Decisions

1. **A turn end never reads as done while a background shell is outstanding.**
   Decision: when the latest turn end lists an outstanding shell (either signal), the run is not finalized `done` on its output files alone; the wait keeps holding, ending only on the run's deadline, the liveness check or an explicit stop.
   When a run is nevertheless finalized with a shell still outstanding (deadline, liveness, stop), `run.json` records each such shell (id, label, first-seen time) as discarded, and Webster's run outcome surfaces it as a finding so the outcome never reads as "tests passed".
   Rationale: the board states the shell was killed by strand removal while the run reported `done`; holding is the first outcome the entry names and recording is its stated fallback.
   Bound: the hold adds no new expiry and no new skip; it ends on the same deadline and liveness exits as today, and PATTERN-completion-signal still applies, so output files remain the return value when a deadline expires.
   The recorded discard is informational for `done` runs and changes no outcome string.
   The planner verifies at base whether `pollEventsTick` and `expiredTurnEnd` already hold the reported path (a waiting turn end with a shell is held at base) and fixes whichever path still finalizes `done` over a live shell.

2. **The verify-failure rule is stated where Master reads the failure.**
   Decision: the `Gate findings recorded at …` message built in `gate.go` for a Webster Master names the fixer path in the message itself: do not edit or commit, spawn one fixer fork with the verify-fix prompt.
   The Master stencil's gate-failure section keeps its rule and gains the same first sentence.
   Rationale: the message is what Master acts on at the moment of failure; the stencil rule sat in a section read at run start.
   Bound: this is a text change only; it blocks nothing, and the run-exit parent-write audit remains the enforcing guard.
   The message text change is limited to Webster's gate; other gate consumers keep their text.

3. **Recovery attribution is by batch, not by Master session.**
   Decision: the files a prior recovery strand of the same batch wrote count as "Written by this run" in the next recovery's prompt, by recording each recovery strand's session id on the batch record and including it in the audited sessions of `loadRunWrites`.
   Rationale: at base `recordedSessions` audits only Master sessions, so a stopped recovery's edits appear unattributed and land under "Not written by this run".
   Bound: attribution widens only to sessions the batch record names, for that batch; a path no recorded session wrote stays foreign, and the strand still never stages a foreign path.

4. **`recover-batch` exits non-zero when the batch ends failed and terminal.**
   Decision: the verb returns a non-zero exit with the digest still on stdout when the recovery ends terminal `failed` or `dead`.
   Rationale: exit 0 over a terminal failure misleads any caller reading the code.
   The Master stencil's failure-ladder text says a non-zero exit with a terminal digest is the expected failed-batch signal, read from the digest, so the backgrounded call's non-zero exit is not treated as a harness fault.
   Bound: only terminal failed or dead outcomes change; a refusal keeps its current exit, and a stuck non-terminal recovery keeps its current code.

5. **The recovery prompt names its allowed gates.**
   Decision: `webster-prefix-recovery.md` states that `lyx gate test` and `lyx loom lint-comments` for the card and batch gates are always allowed despite the ban on fabric commands, and that tmux-tagged card steps are left to verify and only compile-checked.
   Rationale: the cold recovery had no such instruction and was unsure which commands the ban covered.
   Bound: the allowance names those two command families only; every other fabric command stays banned.

6. **The Create check's refusal says how the target was looked up.**
   Decision: the `create-not-done` detail reports the resolve key used, the returned status and, for a non-resolving answer, the candidates, so a missing symbol reads differently from a resolver limit.
   The planner first establishes whether quarry resolves unexported identifiers and identifiers declared in `_test.go` files; if it does, #511 is a stale-tree or key problem the clearer detail exposes, and if it does not, the check falls back to a declaration lookup in the target's file for those two cases.
   Rationale: the board asks to find whether the resolver is the limit, and to make the refusal distinguish a missing symbol from a resolver limit.
   Bound: the fallback applies only to a Create target quarry answers `not_found` for whose unit exists; it can pass a Create only when the identifier is declared in that unit's files, and never changes Delete or Rename verdicts.

7. **DoneChecks accept an ambiguous answer whose candidates have disjoint build constraints.**
   Decision: a single predicate in `internal/planglyph` decides disjointness from each candidate file's `//go:build` expression and filename `_GOOS`/`_GOARCH` suffix; DoneChecks treat such an `ambiguous` answer as resolved for `create-not-done` and `rename-not-done-new`.
   The plan-time `glyph-ambiguous` finding uses the same predicate: a disjoint-constraint ambiguity passes for Edit, and a genuine ambiguity keeps its blocking finding, with its detail naming whether the candidates' build constraints overlap.
   Rationale: the board offers accepting disjoint-constraint ambiguity or honouring constraints in quarry, and quarry is outside this repository.
   Bound: acceptance requires every candidate to sit in a file with a constraint pairwise disjoint from the others; any candidate without a constraint, or any overlapping pair, stays ambiguous and blocking.
   `delete-not-done` is unchanged: an ambiguous answer there still blocks.

## Constraints

- PATTERN-completion-signal: every negative finalize in `internal/shuttleengine` consults `allOutputFilesExist` first; item 1 adds a hold and a record, not a new negative verdict.
- PATTERN-shuttle-stop: a Webster-side stop of a strand goes only through shuttle's stop verb.
- PATTERN-sole-parsers and PATTERN-ref-shape-registry: `planglyph` reads plan refs only through `planparser`; the build-constraint predicate parses Go build lines and filenames, not plan syntax.
- PATTERN-stencil-ownership and PATTERN-parent-directive: prompt text changes live in `contracts/stencils/` and are read at call time, with no new embedded text and no stencil telling an agent to ask the operator.
- PATTERN-no-denied-recovery and PATTERN-refusal-way-forward: the new exit and any new refusal name a way forward, add a spec row and a test, and never name a command the agents' settings deny.
- PATTERN-shuttle-provider-seam: Claude-specific payload parsing for outstanding shells stays under `internal/shuttleengine/claudeengine`.
- PATTERN-test-economy and the test budget: new tests extend existing tables; a package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Docs land in the same commit: `doc.go` of `internal/shuttleengine`, `internal/websterengine` and `internal/planglyph` for changed behavior; no new PATTERN entry is expected.

## Auto-mode assumptions

- No operator was available; every choice below is a self-pick recorded in the support log's question ledger.
- Item 1 assumes the shell hold already holds for a waiting turn end at base and that the remaining defect is a path that finalizes `done` over a live shell or loses the record; the planner confirms and fixes the actual path.
- Item 2 assumes a text change meets "the rule is stated where Master reads a verify failure" and that no new verb is wanted.
- Item 3 assumes recovery strand session ids can be recorded on the batch record without changing the state schema's compatibility; if a schema change is needed it is additive.
- Item 6 assumes the quarry dependency stays untouched and a resolver limit, if present, is bridged inside `planglyph`.

## Open risks

- Quarry may resolve unexported and test-file identifiers correctly, in which case #511 has another cause (a stale tree, or a plan handle's resolve key); the detail from decision 6 is what exposes it.
- A hold on an outstanding shell lengthens a run whose shell never ends until the deadline; the recorded discard bounds the damage but not the time.
- Disjointness of arbitrary build expressions is evaluated over the tags both expressions mention, so an exotic constraint may be judged overlapping and stay blocking; that fails closed.
- Recording recovery sessions on the batch record touches a state file that a resumed run reads; an older record without the field must read as no recovery sessions.

## Acceptance criteria

- A turn end listing an outstanding background shell, with all output files present, does not finalize `done` until the shell ends or the deadline, liveness check or a stop ends the run.
- A run finalized with a shell outstanding records the discarded shell in `run.json`, and Webster's outcome shows it as a finding.
- The `Gate findings recorded at …` message a Webster Master receives tells it to spawn the fixer fork and not to edit.
- A second recovery of a batch lists files the first recovery's strand wrote under "Written by this run", and a path no recorded session wrote stays under "Not written by this run".
- `recover-batch` exits non-zero when the batch ends terminal `failed` or `dead`, keeps its digest on stdout, and keeps exit 0 for a recovery that completes.
- The rendered recovery prompt names `lyx gate test` and `lyx loom lint-comments` as always allowed and says tmux-tagged steps are left to verify.
- A Create target for an unexported identifier, and one declared in a `_test.go` file, passes `create-not-done` once declared, and a missing one reports its lookup key, status and candidates.
- An ambiguous answer whose candidates sit in files with disjoint `//go:build` expressions, and one whose candidates are `_linux`/`_windows` files, passes `create-not-done` and the plan-time Edit check.
- An ambiguous answer with an unconstrained candidate or overlapping constraints still blocks, in DoneChecks and at plan time.
- `delete-not-done` still blocks on any ambiguous answer.

## Notes for the plan writer

- Item 1 starts at `internal/shuttleengine/wait.go` (`pollEventsTick`, `recordWaiting`, `expiredTurnEnd`, `finalize`) and the `master-fork-running` corpus case under `claudeengine/testdata/corpus`.
- Item 2: `gate.go` builds the message; `webster-template-master.md` has the gate-failure section and the `verify_fix_prompt_path` marker.
- Item 3: `runwrites.go` (`loadRunWrites`, `recordedSessions`) and `recoverbatch.go` (`uncommittedPathsBlock`, `recoverSpawn`); the claims test in `contracts/stencils/claims_test.go` pins recovery prompt wording and needs the new lines.
- Items 5 and 6: `planglyph/donecheck.go` (`doneCheckVerdicts`), `resolve.go` (`statusFindings`) and `create.go` share the status policy; keep one predicate.
