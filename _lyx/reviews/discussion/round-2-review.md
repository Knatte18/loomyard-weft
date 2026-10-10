---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:99
    summary: D6 promises Rename accepts a partitioned member, but the Rename declaration derivation is Found-only by design and D6 never decides which candidate it derives from
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D6 leaves the base build.Context unstated, so Compiler, ReleaseTags and ToolTags inherited from build.Default make a negated go1.N, gc or goexperiment tag unassignable and fail open
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:97
    summary: D6's free-tag cap is a small cap with no value, so the bound on the weakened guard is left for the plan writer to pick
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: D3 keeps the contract-file finding standing on a recovery write, but the refusal that finding produces still says a fork wrote the file
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:106
    summary: D6's rationale says the evaluation costs a few file reads per ambiguous answer, but MatchFile reopens the file for every environment
---

### [BLOCKING] D6 promises a Rename of a partitioned member without deciding how the new declaration is derived

**Location:** `_lyx/discussion/decision-record.md:99`, with Scope item 6 (line 18) and the acceptance criterion at line 157.

**Issue:** Scope item 6 says Rename accepts a partitioned member, and the acceptance criterion at line 157 names Rename-old.
D6 says a partitioned answer is resolved "like `multipart`" for `statusFindings`, including Rename endpoints.
A Rename's old side also passes through `renameDeclSource` in `internal/planglyph/handle.go`.
That function requires `quarry.StatusFound` and refuses `multipart` by design: its doc says accepting several declarations means arbitrarily choosing `r.Symbols[0]` out of declarations the language allows to differ.
A partitioned answer has the same shape, several declarations of one name.
Under D6 as written, a partitioned old side passes `statusFindings` and is then refused `rename-old-unresolved`, so the scope promise fails.
Alternatively, the plan writer has to invent the rule for picking a declaration.
D6 neither exempts Rename from the scope nor decides that question.

**Fix:** Decide the Rename-old case in D6 and state its bound.
One option: derive from any candidate, since partitioned candidates share one key and so one predicted glyph, and add an acceptance criterion where the old side's candidates carry differing signatures.
The other: keep `renameDeclSource` Found-only, and drop Rename-old from Scope item 6 and the line-157 criterion, naming the way forward for a partitioned rename, such as Delete plus Create or per-constraint Edit.

### [MEDIUM] D6's environment set depends on a base context it never states

**Location:** `_lyx/discussion/decision-record.md:94`, lines 94–96 and the bound at line 107.

**Issue:** D6 defines a free tag as any mentioned tag that is not a GOOS, GOARCH, `unix` or `cgo` name, and enumerates "every assignment" of the free tags.
In `go/build`, `matchTag` returns true for `ctxt.Compiler` and for any name in `ToolTags` or `ReleaseTags`, whatever `BuildTags` holds.
If the context starts from `build.Default`, the obvious starting point, `gc`, every `go1.N` up to the toolchain's version and the toolchain's `goexperiment.*` and `amd64.vN` tags are always true.
The false half of those assignments is then never evaluated.
So a `//go:build !gc` or `//go:build !go1.21` file beside an unconstrained file reads partitioned, though both files build together under gccgo or an older toolchain.
That breaks the bound "only an answer whose candidates provably never coexist passes".
D6's `unix` and GOOS-implication sentence (focus item 2) is right as written: it settles `unix` beside `_windows.go` as partitioned and `!windows` beside `unix` as `glyph-ambiguous`, because both are selected under `linux`.
The gap is the remaining built-in tags.

**Fix:** State in D6 that each environment is built from a zero `build.Context` carrying only `GOOS`, `GOARCH`, `CgoEnabled` and the free-tag assignment in `BuildTags`, with empty `Compiler`, `ReleaseTags` and `ToolTags`, so `gc`, `go1.N` and tool tags are free tags.
Alternatively, name those tag families as fixed true and accept the fail-open edge in Open risks.
Add a criterion row for `//go:build !go1.21` (or `!gc`) beside an unconstrained file staying `glyph-ambiguous`.

### [LOW] D6's tag cap has no value

**Location:** `_lyx/discussion/decision-record.md:97`

**Issue:** "more than a small cap of free tags" bounds both the guard's reach and the enumeration's cost, yet the number is not decided.
The Open risks entry at line 143 and the criterion at line 158 both depend on it.

**Fix:** Name the cap, for example four free tags, in D6.

### [LOW] D3's recovery-kept contract finding still refuses as "a fork wrote"

**Location:** `_lyx/discussion/decision-record.md:71`, with D3's Decision at line 62.

**Issue:** D3 makes a recovery write to `outcome.yaml` after Master's last write keep the contract-file finding standing.
The refusal that finding produces, `contractDeleteClause` in `internal/websterengine/contractevidence.go`, says "a fork wrote %s after Master's last write".
For a recovery write, that text is false.
`PATTERN-refusal-way-forward` asks the refusal to tell the reader what happened and the way forward.
D3 and the Constraints list name new and changed refusal rows for the recovery exit and "card not done", but not this one.

**Fix:** In D3, say the uncleared contract-file refusal names the writer as a fork or a recovery session, and add that wording to the line-152 criterion.

### [NIT] D6's cost claim does not match `MatchFile`

**Location:** `_lyx/discussion/decision-record.md:106`

**Issue:** `Context.MatchFile` opens and reads the file on every call.
Over every pinned GOOS/GOARCH pair, cgo on and off, and 2^k tag assignments per candidate, that is thousands of opens per ambiguous answer, not "a few file reads", unless the bytes are cached.

**Fix:** Say each candidate file is read once and served to the matcher through `Context.OpenFile`, or reword the cost sentence.

## Focus verdicts

- Focus 1 (D1 at the deadline): not a defect.
  `classifyDeadlineExpiry` in `internal/shuttleengine/wait.go` returns `OutcomeDone` whenever `allOutputFilesExist` holds, and `finalize` runs `evaluateGate(true)` on every `OutcomeDone`.
  So D1 line 35, the Open risks entry at line 139 and the criterion at line 148 agree with the code and with `PATTERN-completion-signal`.
- Focus 2 (D6 environment set): the `unix` and GOOS-implication sentence is right, and both pairs get the right verdict; the remaining gap is F2.
- Focus 3 (D3 contract evidence): `contractFileStatus` reads `writes.Master` (successful writes only) and `writes.Forks` (every write).
  D3's rule that recovery writes count beside fork writes is consistent with that, and with the bound at line 71 and the criterion at line 152; the refusal text is F4.
- Focus 4 (D3 reset own-path set): `ownTrackedPaths` feeds `ResetPlan.OwnPaths` and `dirtyTrackedExcept`.
  D3 line 63 and the bound at line 70 say a reset may discard a tracked path a recovery of this run wrote successfully, by path, and nothing else.
  The criterion at line 153 covers both the discard and the refusal for a path no session wrote.
  Not a defect.
