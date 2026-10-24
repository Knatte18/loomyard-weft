---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D11
    summary: D11 reads launch time from start events the gate logs never hold, so the rationale that records and logs hold every figure but the key is false and no pre-landing record yields a launch share
    fix: In D11, add a decision that lyx gate test keeps, per go-json tier run, each package's start-event time and its next event's time (for example a per-package launch gap list in RunRecord, or the raw event lines beside the rendered log), since goJSONCounter writes only event.Output into the log buffer; state that records written before this task carry no launch figure, so the report shows launch time only for records that hold it; replace the rationale sentence with one naming what the records hold today (wait start, start, end, counts) and what this task adds; update the D11 acceptance criterion to read launch time from the new field; and in Auto-mode assumptions, re-ground the OSL-1046 before/after measurement on what pre-landing records can give (slot wait, wall time, per gate kind), with launch share from after the deploy only.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D13, Constraints bullet 1, Open risks bullet 2
    summary: The first retag card's Verify line checks only the tierN tags, so a hub that keeps covers [0] passes it while the installed lyx drops tier0 as covered and the retagged tier0 tests run in no gate
    fix: In D13, extend the Verify line's check so it also fails when any tier in lyx config gate --resolved still carries a covers entry, and say so in the Constraints bullet and in Open risks bullet 2, since the installed gatecli drops a covered tier from a call and tier1's -tags tier1,integration no longer compiles tier0-tagged files.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D8
    summary: D8 settles pause_before against a skipped row but not pause_after, which today fires only when the named producer returns a routed outcome, so a pause after a skipped row is passed silently
    fix: In D8, add a sentence that a pause_after naming a row its entry gate skips fires as for a Done, pausing at the row the skip routes to with the reason paused after <row>, as requested, and clears both conditions as a fired condition does; add an acceptance criterion that a pause_after on a skipped row pauses at its on_done row.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5, D13
    summary: Two guards key on the old tmux and llm tag names, the card Verify check namesGateOnlyTier and the shell guard's raw go row, so after the rename a -tags tier2 or tier3 run passes both, a guard downgrade no decision states or bounds
    fix: In D5, add that internal/planparser's namesGateOnlyTier and internal/shellguard's raw go row pattern (rows.go, the -tags ... (tmux|llm) alternative) match tier2 and tier3 beside tmux and llm, and in D13 add both old spellings to what the follow-up entry removes after deploy; add an acceptance criterion that a card Verify line and a raw go test naming -tags tier2 or tier3 are refused.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D9
    summary: D9 says recipe build refuses an unknown module or key but names no source to check against, and batten's Env.Setting needs a task worktree that does not exist when the recipe builds
    fix: In D9, state that the build checks a module.key operand against the config registry's module templates (the known-key check configreg.SettingLookup makes), never through Env.Setting, so batten's build needs no task worktree; keep Env.Setting for the evaluation when the row is entered.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D14
    summary: Reconcile empties tree_scope when tree_scope_verify was empty, but a sparse hub file cannot tell an empty key from a pruned one, so every later reconcile wipes trees the operator adds under the new on-when-trees rule
    fix: In D14, make the migration fire only while the hub file still holds the tree_scope_verify key; when it holds it empty, empty tree_scope and prune the key; when it holds it set, prune the key and keep tree_scope; when the key is absent, change nothing. State that a hub whose file never wrote tree_scope_verify but holds trees turns tree scope on after migration, and add that case to the acceptance criterion with a second reconcile changing nothing.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Notes for the plan writer bullet 1
    summary: The note lands the one-tag scan before or with the first retag, but the scan passes the repository only once every file is retagged, so landed early it fails the gates of every later card
    fix: Move the ordering into a Decision (D5) and state it there, that D6 and D7 land before or with the first retag card, and the one-tag scan test lands with the last retag card; drop the scan from the note.
  - id: F8
    severity: MEDIUM
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4
    summary: D4 does not say whether a guard-step pass writes a green entry, and recorded as an empty-args package entry it would carry those packages' non-guard tests on a later all or diff run over the same tree
    fix: In D4, add that the guard step writes no green entry, since its -run filter runs only the guard tests, while an empty-args whole entry still covers a later guard-step request; add an acceptance criterion that a guard-step pass followed by an all run on the same tree runs the guard packages' tests.
  - id: F9
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8
    summary: A route back into the entered row through skipped rows only leaves the status naming that row as entered, so the arrival reads as a resume and its gate is not evaluated
    fix: In D8, state that every routed persist and every skip clears the entered row, so a row is entered only between its own entry evaluation and its verdict; add an acceptance criterion that a Stuck routed back to its own row through a skipped row evaluates that row's gate.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D9
    summary: A bare operand holds for a non-zero number, so a batten switch set to 1 or 0 reads as on or off where the Command producer refuses it today, so the claim that the non-bool refusal survives holds for strings only
    fix: In D9, either restrict a bare operand to a bool for module.key and a number for budget.<Segment>, refusing any other type as an evaluation error, or state that a numeric switch is now accepted and drop the claim that the Command producer's refusal survives.
  - id: F11
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5
    summary: D5 lists cmd/testtiming as a package load under integration,tmux,llm, which it holds none of, while its untagged default mode, its -full mapping to integration and spawnscan's tag classification all break after the retag
    fix: In D5, drop cmd/testtiming from the package-load list and add that its untagged default runs -tags tier0, -full maps to tier1, and spawnscan's tag switch reads tier1, tier2 and tier3 beside the old spellings.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: The carry's bound names the toolchain and inherited environment as outside the fingerprint but not ignored files, which DirtyPaths excludes and tests can read, such as .dev-bin
    fix: In D4's Bound sentence, add that ignored files are outside the clean check and the fingerprint, so a change to ignored state a test reads over the same tree is carried.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D3 Rationale
    summary: The rationale says per-site commands no longer share Merge-In's recorded pass, which gives the fix round its lint base today, but loom's told command is Webster's final call, so the base is Webster's end-gate pass
    fix: In D3's rationale, replace Merge-In's recorded pass with the pass of the told command, Webster's end gate in loom and the landing call in darn.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Notes for the plan writer bullet 2
    summary: The note that the retired landing failure kinds map onto the new keys on read is a requirement, but Notes are never a completeness requirement
    fix: Move the sentence into D1 as a decision (the landing failure record names the failing row's key, and a record naming landing reads as the key of the row that wrote it) and drop it from Notes.
---

### [BLOCKING] D11 measures launch time from events the logs never hold

**Location:** `_lyx/discussion/decision-record.md` D11, its acceptance criterion and the OSL-1046 auto-mode assumption.

**Issue:** `runTier` feeds the log buffer through the counter, `newCounter(run.tier.Counter, io.MultiWriter(out, &logBuffer))` (`internal/gatecli/tierrun.go:47-48`).
`goJSONCounter.handle` writes only `event.Output` and drops the event's Action and Time (`internal/gatecli/counters.go:131-150`).
A `start` event has no Output, so no log on any machine holds a `start` event or any event time, and `RunRecord` has no per-package timing (`internal/gateslot/runlog.go:29-53`).
The rationale "the records and logs already hold every figure but the key" is false.
The before/after measurement on OSL-1046, which the assumptions ground on pre-landing records within retention, can give no launch share for the "before" side.
All three other lenses raised the same defect, and the handler confirmed it independently.

### [MEDIUM] D13's Verify line misses a hub that keeps `covers`

**Location:** `_lyx/discussion/decision-record.md` D13, Constraints bullet 1, Open risks bullet 2.

**Issue:** The Constraint asks for both tag spellings and for `covers` gone; the Verify line checks only the tags.
The installed gate drops a tier that another listed tier covers (`internal/gatecli/doc.go`, Verb section), and the hub's `tier1` to `tier3` carry `covers: [0]` (`_board/_lyx/config/gate.yaml`).
With the tags changed but `covers` kept, `--tier 0 --tier 1` runs only `tier1` with `-tags tier1,integration`, which no longer compiles `tier0`-tagged files.
So the retagged `tier0` tests run in no gate, including the whole gates D13's bound relies on, and the only catch passes.

### [MEDIUM] D8 leaves `pause_after` on a skipped row undefined

**Location:** `_lyx/discussion/decision-record.md` D8.

**Issue:** `pause_after` "fires when the named producer returns a routed running outcome" (`internal/shedengine/doc.go`, field ownership).
A skipped row makes no call and returns no outcome.
A darn run at Code-Review budget 0 with `lyx shed pause <run> --after Describe` skips Describe and runs on through PR-Review and Finalize past the requested pause.
Raised by three lenses.

### [MEDIUM] Tag rename blinds two guards keyed on `tmux` and `llm`

**Location:** `_lyx/discussion/decision-record.md` D5, D13.

**Issue:** `namesGateOnlyTier` refuses a card Verify `go test` whose `-tags` names `tmux` or `llm` (`internal/planparser/verifymodule.go:135,146-150`).
The shell guard's raw go row matches `-tags ...(tmux|llm)` (`internal/shellguard/rows.go:175-181`), which the handler confirmed.
After D5, a line written with `-tags tier2` or `-tags tier3` passes both, and after D13's follow-up drops the old spellings every such line does.
No decision states or bounds the downgrade, which the rubric's attack-surface rule requires.

### [MEDIUM] D9's build-time key refusal has no source

**Location:** `_lyx/discussion/decision-record.md` D9.

**Issue:** `settingRef` checks only the `module.key` shape (`internal/shedrecipe/entries_settings.go:41-54`); today an unknown key is refused only at lookup time.
Batten's `Env.Setting` first calls `locateTask()` (`internal/battencli/wire.go:1350-1356`), and the task worktree is created by a row of the same recipe, after the build.
Checking through `Env.Setting` at build fails every batten build, and D9 names no other source.

### [MEDIUM] D14's `tree_scope` emptying is not idempotent

**Location:** `_lyx/discussion/decision-record.md` D14.

**Issue:** The template sets `tree_scope_verify: ""` (`internal/darnengine/template.yaml:5`), and `PATTERN-sparse-hub-base` keeps an untuned key out of the hub file.
After the first reconcile prunes the key, every later reconcile reads it as empty and, by the rule as written, empties `tree_scope` again.
That wipes trees the operator adds under D14's own on-when-trees rule, and it breaks the "second reconcile changes nothing" criterion.

### [MEDIUM] The one-tag scan cannot land with the first retag

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer bullet 1.

**Issue:** The acceptance criterion has the scan pass the repository, which holds only once every test file is retagged.
Open risks spread the retag of over a thousand files across cards.
Landed with the first retag card, the scan fails batch and end gates until the last retag card lands, which breaks the card-atomic rule.
The ordering is a requirement, so it belongs in a Decision.

### [MEDIUM] D4 is silent on whether a guard-step pass writes a green entry

**Location:** `_lyx/discussion/decision-record.md` D4.

**Issue:** The guard step runs a tier's command with empty `--` args plus its own `-run '^(names)$'` (`internal/gatecli/doc.go`, Guard step).
D4 writes an entry "after the tier passed", and lets an empty-args entry cover the guard filter.
If a guard pass is written as an empty-args package entry, a later `all` or `diff` run on the same tree carries those packages, and their non-guard tests never run before landing.

### [LOW] Routing back through skipped rows keeps a stale entered mark

**Location:** `_lyx/discussion/decision-record.md` D8.

**Issue:** X is entered and returns Stuck, routing to W. W's gate is false, so W is never marked entered, and W's `on_done` is X.
The status still names X as entered, so X is re-called without evaluation, against D8's "routing leaves the named row behind".

### [LOW] D9 lets numeric switches through

**Location:** `_lyx/discussion/decision-record.md` D9.

**Issue:** The Command producer refuses a switch that holds no bool (`internal/shedadapters/command.go:98-100`).
A bare operand that holds for a non-zero number reads `batten.open_ide: 1` as on, so the stated survival of that refusal holds for strings only.

### [LOW] D5 misstates `cmd/testtiming`'s exposure

**Location:** `_lyx/discussion/decision-record.md` D5.

**Issue:** `cmd/testtiming` holds no `integration,tmux,llm` load; its loads take the caller's tags (`cmd/testtiming/redundancy.go:594-615`).
What breaks is elsewhere:
- its untagged default mode runs `go test ./...` with no tags (`cmd/testtiming/main.go:162-167`), so it runs nothing once untagged files are tagged `tier0`;
- `-full` maps to `integration` (`main.go:153-160`);
- `spawnscan.go:127-132` reads `tier1` to `tier3` files as untagged.

### [LOW] D4's bound omits ignored files

**Location:** `_lyx/discussion/decision-record.md` D4, Bound.

**Issue:** `DirtyPaths` excludes ignored files (`internal/verifytree/verify.go:260-261`).
Ignored state such as `.dev-bin` can change over the same tree without breaking the carry.
The bound names only the toolchain and the inherited environment.

### [LOW] D3's rationale names the wrong recorded pass

**Location:** `_lyx/discussion/decision-record.md` D3, Rationale.

**Issue:** In loom the round form's told command is `planVerifySource`'s `baseCommand`, the `final --all` call (`internal/loomcli/wiring.go`).
`LatestPass(told)` is therefore Webster's end-gate pass, not Merge-In's (`internal/loomshed/verifygate.go:94`).

### [LOW] A requirement sits in the Notes

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer bullet 2.

**Issue:** `FailureKindLanding` is written by Merge-In and Finalize (`internal/verifytree/failure.go:18`).
Mapping it onto the new keys on read is behaviour the plan must build.
Notes are hints, never a completeness requirement, so the mapping belongs in a Decision; the plan writer reads the decision record, so moving it loses nothing.

## Rejected

- Interactions F3's second half, that a `gate_key` naming a missing or `card`-scoped key is refused only at run time: rejected, because `TierList` refuses an unknown gate at use today, naming `lyx config gate`, and that use-time refusal with its way forward is the gate config's existing convention.
- Existing-paths finding 5 (shell guard, unverified): not a separate finding; the handler verified it and merged it into F4.
