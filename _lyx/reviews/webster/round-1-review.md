---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/frictionengine/reflect.go:67
    summary: a zero Deps.Timeout, which friction_timeout_min 0 documents as deferring to shuttle's run_timeout_min, now makes every reflection fail
  - id: F2
    severity: LOW
    location: internal/webstercli/cli.go:291
    summary: cobra Args-validator and flag-parse refusals of wrapped webster verbs go unnoted, but addVerbs' doc and the Friction Capture Invariant state only the validate and PersistentPreRun exceptions
  - id: F3
    severity: LOW
    location: internal/frictionengine/reflect.go:45
    summary: many new and changed Go comments in the diff are fixed-column wrapped mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F4
    severity: LOW
    location: internal/loomcli/run.go:25
    summary: reflectFriction's lock paragraph still says blocked-path call and archive the directory, which the halt hooks and covered-files archive made stale
  - id: F5
    severity: LOW
    location: contracts/stencils/friction/friction-template-reflection.md:12
    summary: the reflection stencil's new and changed prose lines are fixed-column wrapped, against CLAUDE.md's semantic-line-break rule for every .md file
  - id: F6
    severity: NIT
    location: internal/frictionengine/reflect.go:316
    summary: archiveCovered moves the record before the report, so a failed report move leaves a report the next call deletes as stale
---

Review range: `_lyx/shed/agent-self-reflect/status.json` gives `product.parent` `main`, and `_lyx/loom/rework/` does not exist, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (merge base `1e9cd1252`, commits for cards 1–9).
The plan's verify commands (build, untagged tests, smoke vet) pass on HEAD.

### [MEDIUM] A zero reflection timeout now fails every reflection

**Location:** `internal/frictionengine/reflect.go:67` (`remaining`), `:146` (budget gate), `:95` (settle-path spec)

**Issue:** `loomengine.Config` validates `friction_timeout_min` with "use 0 to defer to shuttle's run_timeout_min" (`internal/loomengine/config.go:435`), and before this diff a zero `Deps.Timeout` reached `Spec.Timeout` as 0, which shuttle resolves to its config default.
The new budget computes `remaining()` as `deps.Timeout - elapsed`, so with a zero timeout the spawn path hits `budget <= 0` and returns `failed` without spawning, and the settle path hands `Attach` a negative `Spec.Timeout`, which `normalizeAttachSpec` rejects, so it also returns `failed`.
An operator who sets the documented value 0 gets a reflection that never runs and notes that accumulate forever.
No test covers a zero timeout.

**Fix:** treat a non-positive `Deps.Timeout` as "no budget": pass 0 through to both specs so shuttle defers to its default, and skip the budget gate.
Document the zero meaning on `Deps.Timeout` and in the one-Timeout sentences, and add a test that a zero timeout spawns with `Spec.Timeout` 0.

### [LOW] Unnoted webster refusals beyond the documented exceptions

**Location:** `internal/webstercli/cli.go:291` (`addVerbs` doc), `internal/webstercli/refusalnote.go:18`, `CONSTRAINTS.md` Friction Capture Invariant

**Issue:** `begin-batch`, `await-batch`, `record-batch` and `recover-batch` declare `cobra.ExactArgs(1)`, and `rebaseline`, `restore-plan` and `accept-audit` declare `cobra.NoArgs`.
Cobra runs `ValidateArgs`, and parses flags, before `PersistentPreRunE` and `RunE`, and `clihelp.RunRootCtx` prints the resulting envelope outside the tee, so a wrong argument count or a bad flag writes no note.
This is the same class as the `resolvePersistentPreRun` exception card 5 states (the friction directory is not resolved yet), so noting them would need behaviour outside the card's scope.
The defect is the documentation: `addVerbs`' doc names only `validate` and the PersistentPreRun refusals, and the CONSTRAINTS.md clause says every refusal but `validate`'s writes a note.
No verb's `RunE` returns a non-nil error and every verb prints through `cmd.OutOrStdout()` inside `RunE`, so no other refusal escapes the tee.

**Fix:** state in `addVerbs`' and `noteRefusals`' docs and in the invariant that refusals cobra raises before the verb's `RunE` (argument-count, flag parsing, PersistentPreRun) stay unnoted.

### [LOW] New and changed Go comments are fixed-column wrapped

**Location:** `internal/frictionengine/reflect.go:45-58,130-132,175-177,191-192,302-306`, `internal/frictionengine/deps.go:14-15,43-44,57-58,69-76,86-87`, `internal/frictionengine/doc.go:10-33`, `internal/loomcli/arm.go:344-352`, `internal/loomcli/cli.go:123-128`, `internal/loomcli/halt.go:1-2,56-57,71-73,97-98`, `internal/loomcli/run.go:1-3,25-30,84-89`, `internal/loomcli/wiring.go:510-511`, `internal/loomshed/interruptpolicy.go:35-37`, `internal/shedverbs/spec.go:42-44,58-65`, `internal/shedverbs/step.go:42-48`, `internal/friction/doc.go:34-35`, `internal/battencli/postrun_test.go:1-2`, `internal/shedverbs/step_test.go` header, `internal/shedverbs/run_test.go:222-227` and the new `reflect_test.go` and `step_test.go` test docs

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line for new and changed comments.
These comments break mid-sentence at a fixed column; `run.go:27` is a changed line that now runs long and breaks mid-sentence on both sides.
Comments already in sentence-per-line form (`refusalnote.go`, the `halt.go` function docs other than the lines listed) comply.

**Fix:** rewrap each listed comment to one sentence per line, leaving untouched legacy lines alone.

### [LOW] reflectFriction's lock doc is stale after the halt hooks

**Location:** `internal/loomcli/run.go:25-33`

**Issue:** the paragraph still calls the non-waiting caller "the blocked-path call" and "The blocked path (wait false)", though card 6 makes it the halt path for `blocked` and `failed` under both verbs, and still says a second reflection "would archive the directory out from under" the first, though card 1's archive moves only covered files and never the directory.

**Fix:** name the halt path and describe the hazard as archiving covered notes out from under the first reflection's live agent.

### [LOW] Reflection stencil prose is fixed-column wrapped

**Location:** `contracts/stencils/friction/friction-template-reflection.md` (header comment and every changed paragraph)

**Issue:** the repository's CLAUDE.md requires one sentence per line in every `.md` file.
The rewritten stencil wraps at a fixed column mid-sentence, for example "reads every friction note left / behind by the agents".

**Fix:** rewrap the changed lines to one sentence per line, keeping the claims `claims_test.go` pins intact.

### [NIT] Archive order can lose the report

**Location:** `internal/frictionengine/reflect.go:316`

**Issue:** `archiveCovered` moves the notes, then the record, then the report.
If the report's rename fails after the record moved, the next call sees no record, treats the report as stale and deletes it, losing the filed issue URLs.
Moving the report before the record keeps every partial failure re-settleable: a remaining record either re-archives a remaining report or, with the report gone, is discarded over notes that are already archived.

**Fix:** move the record last.

## Focus verdicts

- Dead or timed-out prior reflection: `dispositionCandidate` returns `verdictRespawnEligible` for a dead pane and for a terminal record, so `Attach` reports not found and Reflect discards the record and respawns; an attached run that ends not-done is finalized with a terminal outcome and is not found by the next call.
  No wedge.
  The fake-level test `TestReflect_RecordWithoutReport_DiscardedAndReflectedAgain` covers the not-found answer a dead run produces; the dead/alive distinction itself is shuttle's and tested there.
- Budget: the settle-path spec carries `remaining()` and the spawn the remainder, so a positive timeout bounds the whole call (F1 covers zero).
  The stale-report delete runs only with no record or after the settle archive moved the report; every not-done attach returns before it.
- Partial archive failure: the record and report stay until the end of the move list, so a re-invocation re-settles and archives the rest into a second directory, skipping moved notes (F6 covers the one order gap).
  `ReportPath` and `NoteCount` describe the last archive, as card 1 states.
- `failedHalt`: `stepLocked` persists `callErr.Error()` and returns `callErr` unwrapped, and `MarkTransient`'s `Error()` is the wrapped error's text; `Shed.Run` returns `stepLocked`'s error unwrapped.
  `TestStep_ProducerErrorReflectsOnTheErrorEnvelope` drives a real `shedengine.Shed` producer error.
  Not a defect.
- Envelope merge: `output.ErrFields` writes `ok` and `error` after the extras, so they always win, and `run`'s error envelope has no `kind`.
  `battenPostRun` returns nil on an error, and `ErrFields` with nil fields is `Err`, so batten's error envelope is byte-identical.
- Per-card mechanical checks: no Rename or Delete groups.
  `WriteRefusalNote` and `writeHaltNote` have no prior equivalent (`writeTriageFrictionNote` writes a different note, and card 4 names it as the precedent).
  No comment, doc or skill text still cites `armedVerb` or `shouldReflectFriction`.
  Every file in the diff belongs to a card's Edit, Create or Prosa list.
