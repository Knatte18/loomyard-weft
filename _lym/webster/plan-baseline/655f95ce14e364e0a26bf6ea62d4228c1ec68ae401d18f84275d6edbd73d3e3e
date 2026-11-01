# Card 7 — loomshed-part-checks

**Create:**
- `plan:internal/loomshed#NewPartsGate` -> `func NewPartsGate(anchorPath, recordPath string, max int) shuttleengine.Gate`
- `plan:internal/loomshed#ErrPartUnknown` -> `var ErrPartUnknown = errors.New("loomshed: no such part")`
- `plan:internal/loomshed#ValidatePart` -> `func ValidatePart(anchorPath, worktreeRoot, part string, index planindex.Index, pc PlanContext) ([]planindex.Finding, error)`
- `plan:internal/loomshed#NewPartPlanGate` -> `func NewPartPlanGate(anchorPath, worktreeRoot, scratchDir, part string, index planindex.Index, pc PlanContext) shuttleengine.Gate`
- `plan:internal/loomshed#NewPartAccept` -> `func NewPartAccept(anchorPath, worktreeRoot, part string, index planindex.Index, pc PlanContext) func(shuttleengine.Result) error`
- `plan:internal/loomshed#JoinParts` -> `func JoinParts(anchorPath, worktreeRoot string, index planindex.Index, pc PlanContext) (*planparser.JoinResult, error)`
- `plan:internal/loomshed#TestNewPartsGate` -> `func TestNewPartsGate(t *testing.T)`
- `plan:internal/loomshed#TestValidatePart` -> `func TestValidatePart(t *testing.T)`
- `plan:internal/loomshed#TestJoinParts` -> `func TestJoinParts(t *testing.T)`

**Edit:**
- `internal/loomshed#planGate`
- `internal/loomshed#NewPlanGate`
- `internal/loomshed#NewLensReviewPlanGate`
- `contracts/specs/loom-plan-spec.md`

**Extends:**
- `internal/loomshed/doc.go`

**Uses:**
- `plan:internal/partsparser#Validate`
- `plan:internal/partsparser#Parse`
- `plan:internal/partsparser#File`
- `plan:internal/partsparser#PartDir`
- `plan:internal/partsparser#PartOverview`
- `plan:internal/partsparser#Parts.External`
- `plan:internal/partsparser#Parts.SpotPaths`
- `plan:internal/planparser#PartScope`
- `plan:internal/planparser#JoinParts`
- `plan:internal/planparser#JoinPart`
- `plan:internal/planparser#ExportMet`
- `internal/loomshed#planContextFindings`
- `internal/loomshed#TestNewPlanGate`
- `internal/planparser#IsHandleRef`

**Intent:** Build the three checks of the split plan in loomshed (D3, D7, D8): the `parts` gate, the part check, and the join's re-check and call.

`NewPartsGate` is the `Parts-Write` row's `parts` gate closure: it calls `partsparser.Validate(anchorPath, recordPath, max)` once, maps no finding to `GateResult{Passed: true}`, findings to `GateResult{Passed: false}` with one line per finding (`check[/part]: detail`), and a returned error to the gate's error.

`ValidatePart` is the part check, shared by `lym loom validate-plan --part`, the `plan-part` gate and a writer seat's Accept:
- it parses the parts file; an absent file is an error saying the parts file is gone, that a writer or the chair reports it to the parent, and that the parent's way forward is `lym loom goto --to Parts-Write`; a malformed one is an error naming `lym loom validate-parts`, with the same report-to-the-parent clause;
- a part name the file does not hold is an error wrapping `ErrPartUnknown` that names the parts the file holds;
- it parses the part directory `partsparser.PartDir` with `planparser.ParsePlan` and sets `plan.Part` to a `PartScope` whose `Handles` and `Paths` split `Parts.External(part)` by `planparser.IsHandleRef`, and whose `SpotPaths` are `Parts.SpotPaths()`;
- it returns `index.ValidateFormat(plan, worktreeRoot, nil)`'s findings followed by `planContextFindings` over every card with pc's `DecisionRecordPath` cleared, so `pattern-changes-mismatch` and `pattern-line-unprescribed`, whose verdicts rest on the joined overview, are left to the joined plan's `plan` gate, as `caller-uncovered` is by the part scope; the parent's added decision that the part check leaves `pattern-line-unprescribed` to the joined plan is the licence for clearing `DecisionRecordPath`;
- it then adds one blocking `part-export-missing` finding per export of the part's contract that `planparser.ExportMet` reports unmet over the parsed part, the same test `planparser.JoinParts` applies.
  `ExportMet` matches a `plan:` handle by the declaration it names, so a handle the resolve pass rewrote on disk to its canonical spelling still meets the export the parts file spells, and the parts file never changes.
  Its way forward: create the export on a card of the part, declaring a `plan:` export on a `**Create:**` bullet with the parts file's spelling, or re-sign or delete it on a card; when the part's area cannot meet it, message the chair naming the contract line.
  This is D7's fourth difference (the parent's added decision that the part check flags a missing export), so a part writer learns of a missing export from its own check and fixes its own cards before the join.
A part directory `ParsePlan` cannot read as a plan is an error carrying the parse error.

`contracts/specs/loom-plan-spec.md`'s sentence naming the modes the `loomshed` context checks are appended in ("… in every mode but `--require-approved` …") gains `--part`, the `plan-part` gate and the writer seat's Accept, which append them without the decision record, so `pattern-changes-mismatch` and `pattern-line-unprescribed` are left to the joined plan's `plan` gate; its validation checks gain a `part-export-missing` row stating the check above, its way forward and that only the part check raises it; and the `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted` rows, which name the functions that append them with the context checks, add `loomshed.ValidatePart`.

`planGate` takes the plan directory to parse instead of an anchor path, the same `string` parameter now holding `planparser.PlanDir(anchorPath)` from `NewPlanGate` and `NewLensReviewPlanGate`, whose behaviour is unchanged.
`NewPartPlanGate` is the `plan-part` gate: `planGate("Plan-Part-Gate", partsparser.PartDir(anchorPath, part), scratchDir, …)` over `ValidatePart`'s findings, so a parse failure of the part directory is a finding and an index error stays an error, as the plan gate does.
`NewPartAccept` returns the writer seat's Accept: it runs `ValidatePart` and returns nil when no blocking finding remains, else an error carrying the formatted findings; the result it is handed is never read, so a resume's zero result decides nothing (D5).

`JoinParts` is the join's Go half, called by `lym loom join-parts`:
- it parses the parts file, refusing an absent or malformed one as `ValidatePart` does;
- it refuses a part whose `PartOverview` is absent, naming the part and the path;
- it runs `ValidatePart` on each part in file order and refuses the first that has a blocking finding, carrying its findings;
- it then calls `planparser.JoinParts(planparser.PlanDir(anchorPath), parts.Title, …)` with one `JoinPart{Name, Dir, Exports}` per part in file order, and returns its result or refusal unchanged.
  A part with an unmet export is refused one step earlier, by its `part-export-missing` finding, so `planparser.JoinParts`' own unmet-export refusal is the format writer's backstop for a caller that joins unchecked parts, which card 6's `TestJoinParts` reaches.
Each refusal of its own names the part to fix and the way forward: the chair has that part's writer fix it, or fixes it itself when the writer no longer answers, then runs `lym loom join-parts` again.

`TestNewPartsGate` (tier0) covers a valid parts file passing, a file with a check finding failing with that finding's line, and an unreadable parts path returning the error.
`TestValidatePart` (tier0, over the real index `planglyph.NewIndex(fabricengine.NewReferenceRule())`, as `TestNewPlanGate` builds it, since `handle-dangling` and `part-spot-target` come only from the index's `ValidateFormat`) covers:
- a part using a dependency's export and a needed spot's export passing;
- an unknown part refused with `ErrPartUnknown`;
- an absent parts file and a malformed one, each refused with its way forward;
- a part directory `ParsePlan` cannot read refused with the parse error;
- a dangling handle no export covers reported as `handle-dangling`;
- a card targeting a spot reported as `part-spot-target`;
- an export of the contract no card creates reported as `part-export-missing`, one a card re-signs passing, and a `plan:` export the parts file spells without the owner chain that a card declares under its canonical owner-qualified spelling passing;
- a record whose `## PATTERN changes` the part's overview lacks reporting no `pattern-changes-mismatch`;
- `NewPartAccept`, handed a zero `shuttleengine.Result`, returning nil for the passing part and an error carrying the findings for the dangling-handle part;
- `NewPartPlanGate` passing the passing part, failing with the parse error as its findings for an unreadable part directory, and failing with the findings for the dangling-handle part.
`TestJoinParts` (tier0, over the same real index) covers two valid parts joined into the plan directory, an absent parts file refused with its way forward, a missing part overview refused naming the part, a part failing its check refused with its findings, and a `planparser` refusal passed through.

**ImpactSummary:** Adds the part checks and join beside the plan gate; planGate's two callers pass the plan directory they used to derive, with no behaviour change.
