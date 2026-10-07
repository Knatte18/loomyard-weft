---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-handover-position.md:3
    summary: Card 2 changes the formBatches signature but leaves runlevel.go, which holds the first-init call, out of its Edit list, so the card does not build on its own
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/plan/11-accept-committed-readonly.md:20
    summary: Card 11 tells the read-only classifier to reuse blankQuoted for everything quoted, which would pass a double-quoted command or backtick substitution as read-only, against D14's no-substitution rule
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-gate-fabric-reference.md:26
    summary: Card 10 does not say that validate-plan --require-approved and webster's own validate go through planglyph.Validate and so never run card-fabric-reference, and its spec row and help text would misstate which entry points run it
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-gate-fabric-reference.md:33
    summary: Card 10's test rows need a card Verify value that plankit cannot render, so the rows either hand-write plan bytes against PATTERN-sole-parsers or leave plankit out of the Edit list
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/06-profile-cost-quality.md:25
    summary: Card 6 reads a run's plan commit as calibrate.go does, but that read is inline in Calibration.addRun and card 6 does not target calibrate.go, so the read is planned twice
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/02-handover-position.md:31
    summary: Card 2's second cost_test row repeats card 1's over-budget one-card row, which already covers a card over Budget at a late position
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/08-location-free-ref-rules.md:19
    summary: Card 8 plans a new table that re-lists the hits and misses TestRefScanner_Matches already covers through Matches, where only the returned matched text is new behavior
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/10-gate-fabric-reference.md:34
    summary: Card 10's TestNewPlanGate and TestValidatePlanCmd rows cover what the TestGateParity_PlanGate row already covers, and the rework row belongs in TestGateParity_ReworkPlanGate
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/plan/03-tokencount-one-repo-flag.md:20
    summary: Card 3's end-to-end test expects the fixture's calibration row, but run reads base trees from the process working directory and the card does not say how the fixture becomes it
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/plan/11-accept-committed-readonly.md:39
    summary: Card 11 leaves the existing HEAD moved past the start refusal row unchanged although its go run fixture command is not a listed reader and the refusal text now differs
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/plan/05-calibrate-per-fork.md:29
    summary: Card 5's ImpactSummary restates the Intent instead of concluding on blast radius
---

### [MEDIUM] Card 2 leaves out the first-init call site of `formBatches`

**Location:** `_lyx/plan/02-handover-position.md:3` (Edit list) and `:23` (Callers paragraph).
**Issue:** The card gives `formBatches` and `batchCards` a `before` parameter and says first init passes 0.
First init is `newPartition` in `internal/websterengine/runlevel.go` (`formBatches(plan, deps.Batcher, sizes)`), and `runlevel.go` is not in the Edit list; only `runlevel_test.go` is.
Taken literally the card breaks the build, since the card's own targets leave `runlevel.go` unchanged.
**Fix:** Add `internal/websterengine/runlevel.go` to the Edit list and name `Run`'s first-init partition as the 0 call site.

### [MEDIUM] Card 11's classifier lets a double-quoted substitution through

**Location:** `_lyx/plan/11-accept-committed-readonly.md:20` (Classifier paragraph) and `:38` (classifier test table).
**Issue:** D14 requires "no command or process substitution" and rests its evidence on "no listed reader can write".
The card says separators and operators inside quoted spans are text and to reuse `blankQuoted`.
`blankQuoted` blanks double-quoted spans as well as single-quoted ones, so `cat "$(rm x)"` and a backtick inside double quotes would read as `cat` with no substitution, and the call would accept a command that runs `rm`.
A shell expands `$(` and a backtick inside double quotes; only single quotes make them text.
The test table lists `$(…)` and a backtick only unquoted, so nothing would catch it.
**Fix:** State that the substitution check blanks single-quoted spans only, so `$(` and a backtick inside double quotes still refuse, while `|`, `;`, `&&`, `&` and `>` inside either kind of quote stay text.
Add classifier rows for a double-quoted `$(…)`, a double-quoted backtick, and a single-quoted `$(…)` that stays read-only.

### [MEDIUM] Card 10 does not bound the entry points that skip the check

**Location:** `_lyx/plan/10-gate-fabric-reference.md:26` and `:29`.
**Issue:** `validate-plan --require-approved` runs `planglyph.Validate` (`internal/loomcli/validate.go`), and `lyx webster validate` runs it as well (`internal/webstercli/validate.go`).
Neither goes through the index, so neither gets the check, although the verb's Long help calls `--require-approved` "the same format-only set plus the plan-unapproved approval check".
Card 10 names only the default and `--rework` modes, and its spec row says the check runs "at both gate sites and `validate-plan`", which is false for that mode.
The spec's own framing sentence says every check but `plan-unapproved` is in the format-only set `ValidateFormat` runs, which the new row, emitted by the index, also breaks.
The attack-surface question asks the card to state what the check can let through and what bounds it.
**Fix:** State in the Intent that the check is a Plan-Gate check run by the index in `ValidateFormat` and `ValidateRework`, so `validate-plan --require-approved` and webster's own validate do not run it, and that the implementer audit stays the guard.
Make the spec row, the framing sentence in the spec's validation-checks section and the `validate-plan` Long help say the same.

### [MEDIUM] Card 10's tests assume a card `**Verify:**` that `plankit` cannot write

**Location:** `_lyx/plan/10-gate-fabric-reference.md:33`.
**Issue:** The parity row names "a sibling path in a card Verify".
`internal/testkit/plankit` renders no `**Verify:**` field (its `Card` has `Intent`, `Commit` and `ImpactSummary`), and `PATTERN-sole-parsers` makes `plankit` the one writer of valid plans in tests, with raw bytes allowed only for a malformed plan.
The row therefore either hand-writes plan bytes or edits `plankit`, which is not a target.
`plankit.Plan.Sections` can carry a `verify:` section, which is one of the spans the check scans.
**Fix:** Build the rows with `plankit` through a `verify:` section in `Sections` carrying the sibling path.

### [LOW] The plan-commit read is planned twice across cards 5 and 6

**Location:** `_lyx/plan/06-profile-cost-quality.md:25`.
**Issue:** "Read as `calibrate.go` reads it" points at logic that sits inline in `Calibration.addRun` (newest plan commit before the first fork, parsed with `planparser.ParsePlanFrom` over the history).
Card 6 targets `profile.go` only, so it either copies about twenty lines or edits `calibrate.go` outside its targets.
**Fix:** Card 5, which already edits `calibrate.go`, extracts that read into one function, and card 6 calls it.

### [LOW] Card 2 repeats card 1's over-budget row

**Location:** `_lyx/plan/02-handover-position.md:31`; covering row: `_lyx/plan/01-batcher-start-model.md:45`.
**Issue:** Card 1's `cost_test.go` row already asserts a card whose peak at a late position exceeds `Budget` comes out as a one-card batch with the over-budget `Estimate` and its recorded position.
Card 2's second row asserts the same behavior reached through `before`, and its first row already shows the `before` offset moves the position.
**Fix:** Drop the second row and fold the `before + k` position assertion into the first.

### [LOW] Card 8's new table restates hits and misses `TestRefScanner_Matches` covers

**Location:** `_lyx/plan/08-location-free-ref-rules.md:19`; covering test: `TestRefScanner_Matches` in `internal/fabricengine/refscanner_test.go`.
**Issue:** After the refactor `Matches` runs on the shared rule, and `TestRefScanner_Matches` already has rows for a quoted separator path, a bare word, a string literal, a heredoc body, a slug containing the suffix, command position, a separator and a quoted search pattern.
The new behavior is only the returned matched text and the path rule and spelling rule being separately callable.
**Fix:** Limit the new table to rows that assert the matched text (taken from the quote-blanked copy for the bare and spelling patterns, from the quoted text for the separator path) and one row per rule that hits where the other does not.

### [LOW] Card 10 asserts one behavior at three test sites

**Location:** `_lyx/plan/10-gate-fabric-reference.md:34`; covering tests: `TestGateParity_PlanGate` and `TestGateParity_ReworkPlanGate` in `internal/loomcli/parity_test.go`.
**Issue:** The parity row already requires the gate closure and `validate-plan` to give the same finding.
The extra `TestNewPlanGate` and `TestValidatePlanCmd` rows cover the same behavior again.
The rework gate has its own parity test, which is where its row belongs.
**Fix:** Keep the plan parity row, add the rework row to `TestGateParity_ReworkPlanGate`, and drop the `gates_test.go` and `validate_test.go` rows; those files stay targets for the `NewIndex` argument.

### [LOW] Card 3's end-to-end test does not say how the base tree is supplied

**Location:** `_lyx/plan/03-tokencount-one-repo-flag.md:20`.
**Issue:** `run` builds the base-tree reader as `gitrepo.New(wd)` from the process working directory, so the run's `BaseSHA` is found only when the fixture is that directory.
Otherwise the run is skipped with a "base is not in the repository" reason and there is no fixture row.
`t.Chdir` bars `t.Parallel`, and `PATTERN-test-speed` wants a comment naming the global state.
**Fix:** State that the test makes the fixture the working directory and carries that comment.

### [LOW] Card 11 leaves an existing refusal row stale

**Location:** `_lyx/plan/11-accept-committed-readonly.md:39`; existing row: `HEAD moved past the start` in `TestAcceptBatchFabricReference_Refusals`.
**Issue:** That row expects `git reset --keep` after `g.commit()`, and its fixture command is `go run …`, which is not a listed reader.
Under card 11 the start is an ancestor of HEAD, the tree is clean and the entry is not read-only, so the call refuses with the reset-to-start and fresh-run steps, which do not contain that text.
The card lists the new rows but not this change.
**Fix:** Say the row is reworded to the reset-to-start step, since a moved HEAD no longer refuses on its own.

### [NIT] Card 5's ImpactSummary restates its Intent

**Location:** `_lyx/plan/05-calibrate-per-fork.md:29`.
**Issue:** "The calibration section changes shape; its tests follow" repeats the Intent instead of concluding on reach.
**Fix:** Conclude on blast radius: only tokencount's own report format and tests change, and no internal package or command consumes the section.

## Checked without a finding

- Fidelity: D1 to D15 and every item under Constraints map to a card, and no card touches an item the record puts under Out.
- Bounds: cards 1, 2, 9 and 11 state what the new behavior can skip or let through and what bounds it, apart from the gap F3 and F2 name.
- Granularity: cards 5 and 6 and cards 9 and 10 split along independently reviewable units; the shared-read gap is F5.
- Verify: the `## verify:` section covers every targeted package through `./...` and follows the template's order.
- Card order: no `Uses` reaches a later card's target, and card 8 precedes its consumers in cards 9, 10 and 11.
