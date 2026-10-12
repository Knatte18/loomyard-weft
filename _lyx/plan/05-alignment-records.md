# Card 5 — alignment-records

**Create:**
- `internal/alignment/doc.go`
- `internal/alignment/testmain_test.go`
- `internal/alignment/seam_enforcement_test.go`
- `internal/alignment/records_test.go`
- `internal/alignment/prompt_test.go`
- `contracts/stencils/loom/loom-template-alignment.md`
- `internal/alignment#Role`
- `internal/alignment#Point`
- `internal/alignment#PointParentReview`
- `internal/alignment#PointPRGate`
- `internal/alignment#VerdictApprove`
- `internal/alignment#VerdictReject`
- `internal/alignment#VerdictEscalate`
- `internal/alignment#Attempt`
- `internal/alignment#Attempt.VerdictPath`
- `internal/alignment#Attempt.ReportPath`
- `internal/alignment#Attempt.PromptPath`
- `internal/alignment#WriteAttempt`
- `internal/alignment#ReadAttempt`
- `internal/alignment#ListAttempts`
- `internal/alignment#Verdict`
- `internal/alignment#ReadVerdict`
- `internal/alignment#Snapshot`
- `internal/alignment#Settings`
- `internal/alignment#PromptInput`
- `internal/alignment#Prompt`
- `internal/alignment#Spec`
- `internal/alignment#TestReadVerdict`
- `internal/alignment#TestAttempts`
- `internal/alignment#TestSnapshot`
- `internal/alignment#TestPrompt`
- `internal/alignment#TestSpec`
- `internal/alignment#TestToldGeometryInvariant_AllowlistOnly`
- `internal/alignment#alignmentAllowedImports`
- `internal/alignment#TestMain`
- `contracts/stencils#LoomTemplateAlignment`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Create D1's told-geometry package `internal/alignment` with everything about the alignment agent except its lifecycle (card 6): the role, the two points, the verdict schema and its validation, the attempt records, the prompt and launch spec, and the decision snapshot.
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
- Escalate on a substantive overlap: a shared Go package or code file, a decision contradicting or duplicating one in a peer's record or board entry, or a change to the same PATTERN entry. Also escalate on a question needing an operator decision the board does not record.
- Every escalate fills `question`: for a substantive overlap it names the peer and what overlaps, and for an operator question it names the question and the record or entry that leaves it open.
- A shared bookkeeping file only (`cmd/lyx/testdata/test-budget.yaml`, a module-table line in the overview doc, a new PATTERN index line) is named in the report and does not escalate.
- Otherwise approve, or reject on a scope deviation. An approve at `pr-gate` lists the board slugs the run supersedes under `superseded`.
- It writes exactly the report, in the format `{{.review_format_path}}` defines (a report with no findings on an approve), then the verdict file last.
- It runs no review, approval, rejection, decision, goto or resume verb, writes no board entry, sends no message and edits nothing else.

`contracts/stencils`:
- `stencils.go` embeds the stencil as `LoomTemplateAlignment` and lists it in `entries` after the arbiter's.
- `roleOpeningStencils` maps `alignment` to it, so `parentdirective_test.go` enforces both directive markers.
- `wordingClaims` gains its row. `wantAll`: `lyx loom peers`, `lyx board list`, `{{.verdict_path}}`, `{{.report_path}}`, `escalate`, `question`, `superseded` and `test-budget.yaml`. `wantNone`, since the agent moves nothing and messages no session: `SendMessage`, `lyx loom approve`, `lyx loom reject`, `lyx loom review`, `lyx loom decision`, `lyx loom goto`, `lyx loom resume` and `lyx board upsert`.

Tests:
- `TestReadVerdict` has one row per acceptance and per refusal above, and its acceptance rows include an overlap `escalate` with a filled `question`.
- `TestAttempts` covers the write/read round trip with `Dir` and the token paths, `ListAttempts`' ordering, an absent root, a skipped directory without `attempt.json`, and an unparseable record.
- `TestSnapshot` covers equality for equal input and a change for each altered, added or removed input.
- `TestPrompt` runs over a stencils dir seeded with `stencilkit.SeedInto`. It checks that both points render every marker, that no `{{` is left, that the point's paths and `none` fills are named, and that both directives are present.
- `TestSpec` asserts each field above.

`docs/overview.md`'s module table gains `internal/alignment`, and `doc.go` describes the package.
`test-budget.yaml` gains `internal/alignment: 6`.

**ImpactSummary:** A new package and a new stencil; the stencil registry and claims tables gain one row each, and nothing existing changes behavior.
