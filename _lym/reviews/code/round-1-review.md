---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: 'lens:code-tests'
    location: 'internal/loomcli/wiring_darn_test.go:116'
    summary: 'No test shows the wired PreflightStencils refuses an unfilled marker; the wiring test only checks it is non-nil and passes on the shipped stencils'
    fix: 'In internal/loomcli/wiring_darn_test.go, after wire() of a darn seed, add a case that writes a copy of the darn-template-write stencil with one extra top-level marker no value fills into the stencils directory wireDarn was given, calls c.env.PreflightStencils(), and asserts the error names seat "darn", the stencil and the extra marker. Without it, a body of return nil keeps the suite green and silently disables the check in every real run.'
  - id: F2
    severity: LOW
    class: design
    origin: 'lens:code-correctness'
    location: 'internal/seatengine/prompt.go:311'
    summary: 'fills counts a present-but-empty or whitespace-only non-Optional table or seat value as filling a marker, but FillWith refuses it, so CheckMarkers can pass a table whose real fill fails'
    fix: 'In Table.fills, count a table or seat value only when strings.TrimSpace of it is non-empty; keep the Optional and reserved-marker exemptions as they are. Add one row to TestTable_CheckMarkers with a blank non-Optional table value for a marker the lead stencil uses, expecting the refusal naming that marker.'
  - id: F3
    severity: LOW
    class: design
    origin: 'lens:code-correctness'
    location: 'internal/loomcli/darnwiring.go:261'
    summary: 'A failure to build the darn table inside PreflightStencils is reported under the reason prefix seat stencils cannot be filled and carries no way forward, though the fault is config or model-spec'
    fix: 'In the PreflightStencils closure in wireDarn, wrap an error returned by darnTable with a prefix that names the table build and ends with the same fix-and-re-step way forward the other Preflight refusals carry, so only a CheckMarkers failure reads as an unfillable stencil. Cover it with a TestPreflight_StencilsCheck or wiring row if the wrap is added.'
  - id: F4
    severity: LOW
    class: design
    origin: 'lens:code-tests'
    location: 'internal/shedrecipe/entries_simple.go:25'
    summary: 'Nothing pins that preflightEntry forwards env.PreflightStencils into Checks.Stencils; dropping it would disable the check in every real run'
    fix: 'In the shedrecipe tests that build the Preflight row, add a case that sets Env.PreflightStencils to a failing func, calls Call on the producer from preflightEntry, and asserts Stuck with the stencils reason prefix. Place it as a row in the existing Preflight entry test rather than a new test.'
  - id: F5
    severity: LOW
    class: design
    origin: 'lens:code-tests'
    location: 'internal/seatengine/prompt_test.go:283'
    summary: 'The role-named error for the empty-name lead, the reported bug, is pinned on the Validate path only; the compose, start, probe, strandNames and seatValues errors have no lead case'
    fix: 'Turn TestComposeSeatPrompt_RefusesAMarkerAbsentFromTheValueMapNamingTheSeatRole into a table with a lead row expecting seat "multi" beside the advisor row. Add no per-site tests for the start and probe errors unless the seat fakes already reach them as a row.'
  - id: F6
    severity: LOW
    class: design
    origin: 'lens:code-tests'
    location: 'internal/seatengine/table_test.go:181'
    summary: 'TestTable_CheckMarkers has no row for a stencil parse error, for a marker used in both the stencil and a block appearing once, or for refusal of the first failing seat when several fail'
    fix: 'Add three rows to TestTable_CheckMarkers in internal/seatengine/table_test.go: a stencil with a malformed template action expecting the error naming the seat and stencil, a marker present in both lead stencil and its block expecting it listed once, and two seats both unfilled expecting only the lead named.'
  - id: F7
    severity: LOW
    class: consistency
    origin: 'lens:code-affected-set'
    location: 'internal/preflightshed/preflight.go:1'
    summary: 'The file header still names NewPreflightWithBatcher, which this diff replaced with NewPreflightWithChecks'
    fix: 'Change the first line to say the file implements NewPreflight and NewPreflightWithChecks, the general Preflight producer with its optional checks.'
---

### [MEDIUM] Wired stencils check is never shown to refuse

**Location:** internal/loomcli/wiring_darn_test.go:116
**Issue:** `wireDarn` builds `PreflightStencils` from `darnTable` and `CheckMarkers`. The only wiring test asserts the func is non-nil and returns nil on the shipped stencils. Each half is tested alone (`TestTable_CheckMarkers`, `TestPreflight_StencilsCheck`), but nothing joins them, so the main deliverable of the board entry, Preflight rejecting the unfillable seat, rests on an untested closure.

### [LOW] CheckMarkers counts blank values as filling

**Location:** internal/seatengine/prompt.go:311
**Issue:** `FillWith` treats an empty or whitespace-only value as unfilled unless the name is Optional (`internal/stencil/stencil.go:298`). `fills` tests key presence only. The darn table cannot reach this today, since `DarnTable` refuses empty required values and its empty ones are Optional, but the method's doc says a passing table composes no prompt with an unfilled marker, which does not hold for a blank value.

### [LOW] Table-build failure shown as an unfillable stencil

**Location:** internal/loomcli/darnwiring.go:261
**Issue:** An error from `darnTable` (empty `SubagentModel`, unresolvable writer model-spec, project-record read) becomes the Stuck reason `seat stencils cannot be filled: ...` with no way forward, which misnames the fault.

### [LOW] preflightEntry forwarding is unpinned

**Location:** internal/shedrecipe/entries_simple.go:25
**Issue:** The line that passes `env.PreflightStencils` on has no test in the diff.

### [LOW] Lead role label is pinned only on Validate

**Location:** internal/seatengine/prompt_test.go:283
**Issue:** The reported error was a compose-time `seat ""`. The compose test uses an advisor, so a lead-specific regression there is unpinned.

### [LOW] CheckMarkers branches without a row

**Location:** internal/seatengine/table_test.go:181
**Issue:** The parse-error return, the dedup across stencil and block, and first-failing-seat ordering have no row.

### [LOW] Stale file header

**Location:** internal/preflightshed/preflight.go:1
**Issue:** The header names a removed symbol.

## Rejected

- Missing `presentButEmptyBranchMarkers` equivalent in `CheckMarkers`: the doc comment scopes the check to top-level markers, and the entry asks for markers no value fills, not for branch-internal ones.
- Scope creep in `internal/shedrecipe`, `internal/loomcli/darnwiring.go` and `internal/testkit/envkit`: each edit is the seam the Preflight check needs (`Env` field, wiring, nil-legal entry), licensed by the entry's second ask.
- The marker or a hardcoded model dropped anywhere: the diff keeps `subagent_model` in `DarnTable` and the stencil untouched.
- Remaining `seat.Name` quotes at table.go:275, 280 and 283: they quote the name under validation on purpose, as the summary says.
- Summary claims (stale hub stencils from a newer binary as the cause) and the two commit messages: the diff matches the stated changes; the cause itself is an inference the diff cannot prove, and the entry itself calls it likely.
- No `Pattern-Departure` trailer needed and no declared PATTERN change: the directive declares none, and no pattern entry is violated by the diff.
