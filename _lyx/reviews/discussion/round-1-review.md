---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (Table values)
    summary: pattern_directive and friction_directive are empty whenever PATTERN is inactive or Tier 2 is off, and seatengine.Table.Validate refuses an empty table value, so the table D2 specifies fails on every such run
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: An empty-string scalar loads through ModelSpecList as a one-entry list holding an empty string, so the bound that the list length is the advisor count is false for that spelling, and parsing each entry at load would refuse the value D4 calls legal
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5, D8
    summary: The advisor seat and its advisor-to-chair message edge are introduced with limits stated only as stencil instructions; no Decision says what an advisor that ignores them can still do (write the chair's files, anything the whole-directory commit stages) and what bounds that
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: The chair-prompt amendment path needs a seatengine change the Scope does not list, and it is built for plan-multi-llm while this task's only preparation always returns an empty amendment
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6, D7, Acceptance criteria
    summary: Advisor reliance is recorded in two places (ledger per D6, Interview per D7) with different triggers, and no Acceptance criterion pins either, though Open risks rests on that attribution
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 Bound, Constraints (in-flight runs)
    summary: D3 bounds the preparation and archive by the probe finding no live chair, but the Constraints bullet admits the probe misses a live old-binary discussion writer, so the bound does not hold in that case and no step stops the old writer
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 Rationale
    summary: D5's rationale argues the rejected challenger role, a rejected alternative that the writer stencil keeps out of the decision record and that the support log already carries
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D10
    summary: roleOpeningStencils keys the advisor by its strand role discussion-advisor but keeps the chair under discussion, while the chair's strand role becomes discussion-chair
---

### [BLOCKING] Empty optional directives fail table validation

**Location:** `_lyx/discussion/decision-record.md` D2, Table values; Acceptance criteria, first bullet.

**Issue:** D2 puts `pattern_directive` and `friction_directive` among the table values, the friction one "swallowed to empty on a read failure as today".
Today both are optional markers: `internal/loomengine/prompt.go` fills them through `stencil.FillOptional` with both names in the optional list.
`pattern.Directive` returns `""` for an inactive PATTERN, and `friction.Directive` returns `""` for an empty note path, which `DiscussionSpec` passes whenever `cfg.Friction` is empty, so Tier 2 off.
The seat engine has no optional-marker path:
`seatengine.Table.Validate` → `validateValues` refuses a value whose `strings.TrimSpace` is empty (`internal/seatengine/table.go:205`), and `composePrompt` calls `stencil.FillWith(body, blocks, values, nil)` with `missingkey=error`, so leaving the key out fails too.
The table D2 describes therefore fails validation on every run with Tier 2 off, on every friction read failure, and in a repo without PATTERN.
The premise that the directives carry over "as today" is false, and the first Acceptance criterion tests only the advisor count, not these cases.

**Fix:** Add a Decision on how an inactive directive reaches a seat: for example a seatengine table field naming optional markers, passed to `FillWith` and exempt from the blank-value rule, added as a new field per the additive Constraint, or another mechanism with a stated bound.
Extend the first Acceptance criterion to cover PATTERN inactive, Tier 2 off and a friction read failure.

### [MEDIUM] The empty scalar breaks D4's count bound

**Location:** `_lyx/discussion/decision-record.md` D4.

**Issue:** D4 reads the key "with the same scalar-or-sequence shape as `review`", parses each entry at load, and calls "the empty list and the empty scalar" legal, with the bound "the list's length is the advisor count".
`ModelSpecList.UnmarshalYAML` (`internal/loomengine/config.go:352`) turns a scalar into a one-entry list, so `discussion_advisors: ""` loads as `[""]`, with length 1.
The per-segment keys already handle this spelling through `isUnsetModelSpecList`.
D4 states no such normalisation, so as written the empty scalar either yields one advisor or is refused when its empty entry is parsed, which contradicts D4's own "legal".
A sequence holding an empty string (`[""]`) is also unaddressed.

**Fix:** State in D4 that a load normalises the empty scalar, and null, to zero advisors before entries are parsed, and say whether an empty entry inside a sequence is refused.
Make the second Acceptance criterion name each spelling.

### [MEDIUM] Advisor bounds are stencil text only

**Location:** `_lyx/discussion/decision-record.md` D5 item 5, D8.

**Issue:** D5 and D8 introduce a new seat and a new routing edge, advisor to chair.
The limits that matter (write only the notes file, message only the chair, never mutate git) are instructions in the advisor stencil and the shipped advisor seat directive.
An advisor runs with the same tool surface as the writer.
The discussion commit stages the whole `_lyx/discussion/` directory (`internal/loomcli/wiring.go:293`), so an advisor that ignores its brief can rewrite the chair's decision record or add files there, and the commit carries them.
Its messages can also reach the chair as instructions.
The rubric's attack-surface rule requires the Decision to say what the new seat can now let through and what bounds it.
D5 and D8 say neither.
Open risks covers only the chair leaning on advice.

**Fix:** State the bound in D5 and D8.
For example: an advisor that ignores the brief is bounded as the single writer is, by the agents' settings denies, by the chair's gate and self-check over the two files, by the parent review and by the Discussion-Review perch; the chair seat directive makes an advisor message advice, never an instruction.
Name what is not bounded, such as an advisor edit to the decision record that passes the gate, so the plan can decide whether a check is needed.

### [MEDIUM] The amendment path is unscoped and built ahead of need

**Location:** `_lyx/discussion/decision-record.md` D3; Scope, In; Acceptance criteria, fourth bullet.

**Issue:** D3 appends the preparation's amendment "verbatim to the chair's prompt".
Prompts are composed inside `seatengine` (`composePrompt`, `seatValues`), and neither `Table` nor `Seat` has a field that carries text appended to one seat's prompt, so this needs a seatengine change.
The Scope's In list names only two seams on `MultiLLMProducer`.
This task's only preparation, the parent-review `PrepareRound` wrapper, always returns `""` (`internal/shedrecipe/entries_discussionwrite.go`).
The rationale justifies the append by `plan-multi-llm`, a note the Scope puts Out, which designs for a requirement this task does not have.

**Fix:** Either keep the preparation's signature but drop the append, refusing or ignoring a non-empty amendment, and leave the append to `plan-multi-llm`; or keep it and list the seatengine field in Scope with its bound.
Align the fourth Acceptance criterion with the choice.

### [LOW] Advisor reliance has two records and no pinned criterion

**Location:** `_lyx/discussion/decision-record.md` D6 (autonomous bullet), D7, Open risks (third bullet), Acceptance criteria.

**Issue:** D6 records an advisor-informed pick in the question ledger in autonomous mode.
D7 records "each advisor answer the chair relied on" in `## Interview`, in both modes.
The two disagree on the trigger, a pick against any relied-on answer, and on the place, and neither says how they relate.
Both name "the seat", while the chair knows advisors by strand name (`advisor_names`).
No Acceptance criterion pins either, though the third Open risk calls the attribution what makes reliance reviewable.

**Fix:** Say which record does what, for example the ledger marks the pick and the Interview carries the answer, or pick one place.
Name the identifier, seat or strand name, and add a chair-stencil claim for it to the Acceptance criteria.

### [LOW] D3's bound does not cover a live old-binary writer

**Location:** `_lyx/discussion/decision-record.md` D3 Bound; Constraints, last bullet; Scope, Out.

**Issue:** D3's bound rests on the probe finding no live chair before the preparation (`PrepareRound`) and the archive run.
The Constraints bullet says the seat probe looks for `discussion-chair`, so a live `discussion` writer from the old binary is not seen, and a fresh table starts beside it.
In that case the preparation and the archive of the two discussion files also run beside a live writer.
"Restarted, not migrated" says no step stops the old writer before the new binary drives the run.
The recipe header precedent assumes nothing live writes the files.

**Fix:** Qualify D3's bound for this case, and state in the Constraint who stops a live old-binary writer before the new binary resumes the run, for example the operator before landing.

### [LOW] A rejected alternative sits in D5's rationale

**Location:** `_lyx/discussion/decision-record.md` D5, Rationale, second sentence.

**Issue:** "A second, distinct challenger role is not built: …" argues a rejected alternative.
The writer stencil keeps rejected alternatives out of `## Decisions`, and the support log's Rejected alternatives already carries this one.
The requirement it carries, one brief for every seat, is already stated in D5's body.

**Fix:** Delete the sentence from D5.
Nothing has to move first.

### [NIT] Opening-stencil role keys disagree

**Location:** `_lyx/discussion/decision-record.md` D10.

**Issue:** `roleOpeningStencils` (`contracts/stencils/stencils.go:380`) keys roles by name.
D10 adds `discussion-advisor`, a strand-role stem, but keeps the chair under `discussion`, which is the seat table's prefix, while the chair's strand role becomes `discussion-chair`.

**Fix:** Key the chair as `discussion-chair`, or say why the prefix stays the key.

## Verdicts on focus items with no finding

- D6 against the chair's write surface: not a defect.
  Reading notes files and receiving `SendMessage` write no file, the stencil's fence still names only the two discussion files, and in interactive mode the operator's turns reach the chair alone, which the seat engine enforces (`seatSpec` makes only the chair interactive).
- D3 against D9's driver-crash path: consistent.
  `Engine.Probe` stops live advisors when the chair is not live, so advisors are stopped by the probe before the preparation runs, not by the preparation.
- D1, D10 and the Constraints' premises: the token report's `roleOf` drops a trailing `-N` from the last segment (`tools/tokencount/count.go:410`); the seat engine fills the markers the Notes list (`internal/seatengine/prompt.go`); `Table.Validate` requires every advisor output among the chair's inputs.
  All hold; the false premise is D2's, filed as F1.
