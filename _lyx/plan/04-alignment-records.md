# Card 4 — alignment-records

**Create:**
- `internal/alignment/doc.go`
- `internal/alignment/testmain_test.go`
- `internal/alignment/seam_enforcement_test.go`
- `internal/alignment/records_test.go`
- `internal/alignment/prompt_test.go`
- `contracts/stencils/loom/loom-template-alignment.md`
- `plan:internal/alignment#Role` -> `const Role = "alignment"`
- `plan:internal/alignment#Point` -> `type Point string`
- `plan:internal/alignment#PointParentReview` -> `const PointParentReview Point = "parent-review"`
- `plan:internal/alignment#PointPRGate` -> `const PointPRGate Point = "pr-gate"`
- `plan:internal/alignment#VerdictApprove` -> `const VerdictApprove = "approve"`
- `plan:internal/alignment#VerdictReject` -> `const VerdictReject = "reject"`
- `plan:internal/alignment#VerdictEscalate` -> `const VerdictEscalate = "escalate"`
- `plan:internal/alignment#Attempt` -> `type Attempt struct`
- `plan:internal/alignment#Attempt.VerdictPath` -> `func (a Attempt) VerdictPath() string`
- `plan:internal/alignment#Attempt.ReportPath` -> `func (a Attempt) ReportPath() string`
- `plan:internal/alignment#Attempt.PromptPath` -> `func (a Attempt) PromptPath() string`
- `plan:internal/alignment#WriteAttempt` -> `func WriteAttempt(a Attempt) error`
- `plan:internal/alignment#ReadAttempt` -> `func ReadAttempt(dir string) (Attempt, error)`
- `plan:internal/alignment#ListAttempts` -> `func ListAttempts(root string) ([]Attempt, error)`
- `plan:internal/alignment#Verdict` -> `type Verdict struct`
- `plan:internal/alignment#ReadVerdict` -> `func ReadVerdict(a Attempt) (Verdict, bool, error)`
- `plan:internal/alignment#Snapshot` -> `func Snapshot(decisionRecord []byte, notes []string) string`
- `plan:internal/alignment#Settings` -> `type Settings struct`
- `plan:internal/alignment#PromptInput` -> `type PromptInput struct`
- `plan:internal/alignment#Prompt` -> `func Prompt(stencilsDir string, in PromptInput) (string, error)`
- `plan:internal/alignment#Spec` -> `func Spec(prompt string, a Attempt, s Settings) shuttleengine.Spec`
- `plan:internal/alignment#TestReadVerdict` -> `func TestReadVerdict(t *testing.T)`
- `plan:internal/alignment#TestAttempts` -> `func TestAttempts(t *testing.T)`
- `plan:internal/alignment#TestSnapshot` -> `func TestSnapshot(t *testing.T)`
- `plan:internal/alignment#TestPrompt` -> `func TestPrompt(t *testing.T)`
- `plan:internal/alignment#TestSpec` -> `func TestSpec(t *testing.T)`
- `plan:internal/alignment#TestToldGeometryInvariant_AllowlistOnly` -> `func TestToldGeometryInvariant_AllowlistOnly(t *testing.T)`
- `plan:internal/alignment#alignmentAllowedImports` -> `var alignmentAllowedImports []string`
- `plan:internal/alignment#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/shedrun#AlignmentDir` -> `func AlignmentDir(l *lyxcwd.Location, runID, point string) string`
- `plan:internal/shedrun#AlignmentRel` -> `func AlignmentRel(l *lyxcwd.Location, runID string) string`
- `plan:contracts/stencils#LoomTemplateAlignment` -> `var LoomTemplateAlignment []byte`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestDriveReportsRel`
- `internal/shedrun/doc.go`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Create D1's told-geometry package `internal/alignment` with everything about the alignment agent except its lifecycle (card 5): the role, the two points, the verdict schema and its validation, the attempt records, the prompt and launch spec, and the decision snapshot.
It imports the standard library, `internal/shuttleengine`, `internal/segmentcolor`, `internal/stencil`, `internal/stencilstore`, `internal/parentdirective`, `internal/editdirective`, `internal/logger` and `gopkg.in/yaml.v3`, never `internal/lyxcwd` (`PATTERN-told-geometry`).
`seam_enforcement_test.go` pins that set through `scankit.AssertImportAllowlistNoStale`, as `internal/landingshed`'s does, and `testmain_test.go` runs the package under `tmuxkit.Main`.

Records (`records.go`):
- `Role` is the agent-name role reed forms into the strand name. A second agent of a run takes the numbered form, like the arbiter's.
- `Attempt` is `attempt.json` in its directory. It holds `Number`, `Key` (the idempotence key the session reuses), `Point`, `Token` (random per spawn), `SpawnedAt`, `Deadline`, `StrandGUID`, `Snapshot`, `PRNumber` and `HeadSHA` (PR-Gate only), `Closed`, `ClosedReason`, `Applied` (the verdict Go applied, empty until then) and `Notified`, plus a `Dir` field that is never serialized.
- `VerdictPath` and `ReportPath` carry the token (`verdict-<token>.yaml`, `report-<token>.md`), so only the agent's prompt names them (D2's bound), and `PromptPath` is `prompt.md`.
- `WriteAttempt` creates the directory and writes the record. `ReadAttempt` reads one and sets `Dir`.
- `ListAttempts` reads every subdirectory of root that holds an `attempt.json`, sorted by number. An absent root lists none, a subdirectory without the file is skipped, and an unparseable file is an error naming it.
- `Verdict` is the agent's `verdict-<token>.yaml`: `verdict`, `reason`, `question` and `superseded` (a list of board slugs).
- `ReadVerdict` returns found false for an absent file. It refuses: a file that does not parse or carries an unknown key; a verdict word other than the three; an empty reason; an `escalate` with no question (D3: an escalation without a named question is invalid); `superseded` on anything but an `approve`; and a missing or empty report file, since D2 fixes the output set at both files, every time.
- `Snapshot` is the hex SHA-256 over the decision record's bytes and the notes' texts, in the order given (D5's decision snapshot). A changed record or note changes it, and so does an added or removed note.

Prompt and spec (`prompt.go`):
- `Settings` holds `Model`, `Effort`, `Version` and `Timeout`.
- `PromptInput` holds `Point`, `Slug`, `Worktree`, `DecisionRecordPath`, `SupportLogPath`, `PlanDir`, `BriefPath`, `ParentBranch`, `PRNumber`, `HeadSHA`, `VerdictPath` and `ReportPath`.
- `Prompt` reads `loom-template-alignment` through `stencilstore.Read` at call time (`PATTERN-stencil-ownership`) and fills every marker. An absent brief, PR number or head fills as the literal `none`. `review_format_path` comes from `stencilstore.Path(stencilsDir, "burler-review-format")`. The no-parent directive comes from `parentdirective.Directive(stencilsDir, "", false)`, as the arbiter's does, because the verdict file is the agent's only channel, and the edit directive from `editdirective.Directive`.
- `Spec` composes the `shuttleengine.Spec`: `Prompt`, no skills, `OutputFiles` set to the verdict and report paths in that order (`PATTERN-completion-signal`), the model triple and timeout from `Settings`, `Role` set to `Role`, `Segment` set to `segmentcolor.Review`, `Interactive` false, `ForkSubagents` false and `Parent` empty.

The stencil `loom-template-alignment.md` is the agent's whole procedure for both points, selected by the `{{.point}}` marker, which says which of its two read lists and judge lists applies:
- Read: at `parent-review` the brief, the run's discussion, the board (`lyx board list`, `lyx board get`) and `lyx loom peers`. At `pr-gate` the decision record, the plan and the diff against the parent branch, the board and `lyx loom peers`. Read-only git is allowed.
- Judge: at `parent-review` today's scope check against the board entry's goal and Done-when, no more and no less, with no duplication or contradiction of another entry. At `pr-gate` today's PR-Gate fork check, the run against its board goal and the operator's recorded decisions, with the carried-over `Merge-Review` open risks. Both add the overlap check: every other run `lyx loom peers` lists, plus the parent's commits.
- Escalate on a substantive overlap: a shared Go package or code file, a decision contradicting or duplicating one in a peer's record or board entry, or a change to the same PATTERN entry. Also escalate on a question needing an operator decision the board does not record, naming that question and the record or entry that leaves it open.
- A shared bookkeeping file only (`cmd/lyx/testdata/test-budget.yaml`, a module-table line in the overview doc, a new PATTERN index line) is named in the report and does not escalate.
- Otherwise approve, or reject on a scope deviation. An approve at `pr-gate` lists the board slugs the run supersedes under `superseded`.
- It writes exactly the report, in the format `{{.review_format_path}}` defines (a report with no findings on an approve), then the verdict file last.
- It runs no review, approval, rejection, decision, goto or resume verb, writes no board entry, sends no message and edits nothing else.

`contracts/stencils`:
- `stencils.go` embeds the stencil as `LoomTemplateAlignment` and lists it in `entries` after the arbiter's.
- `roleOpeningStencils` maps `alignment` to it, so `parentdirective_test.go` enforces both directive markers.
- `wordingClaims` gains its row. `wantAll`: `lyx loom peers`, `lyx board list`, `{{.verdict_path}}`, `{{.report_path}}`, `escalate`, `question`, `superseded` and `test-budget.yaml`. `wantNone`, since the agent moves nothing and messages no session: `SendMessage`, `lyx loom approve`, `lyx loom reject`, `lyx loom review`, `lyx loom decision`, `lyx loom goto`, `lyx loom resume` and `lyx board upsert`.

`shedrun` declares where the records live (`PATTERN-shed-run-directory`), beside the arbiter's:
- `AlignmentDir(l, runID, point)` is `RunDir(l, runID)/alignment/<point>`, under `_lyx`, so the records are durable (`PATTERN-durable-vs-ephemeral-state`).
- `AlignmentRel(l, runID)` is the anchor-relative `alignment` directory, for the status commit's pathspec in card 7.
- `TestPathConstructors` and `TestDriveReportsRel` gain their rows, and `doc.go` names the new paths beside the arbiter's.

Tests:
- `TestReadVerdict` has one row per acceptance and per refusal above.
- `TestAttempts` covers the write/read round trip with `Dir` and the token paths, `ListAttempts`' ordering, an absent root, a skipped directory without `attempt.json`, and an unparseable record.
- `TestSnapshot` covers equality for equal input and a change for each altered, added or removed input.
- `TestPrompt` runs over a stencils dir seeded with `stencilkit.SeedInto`. It checks that both points render every marker, that no `{{` is left, that the point's paths and `none` fills are named, and that both directives are present.
- `TestSpec` asserts each field above.

`docs/overview.md`'s module table gains `internal/alignment`, and `doc.go` describes the package.
`test-budget.yaml` gains `internal/alignment: 6`.

**ImpactSummary:** A new package, a new stencil and two new `shedrun` path functions; the stencil registry and claims tables gain one row each, and nothing existing changes behavior.
