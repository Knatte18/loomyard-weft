---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/loomshed/planwrite.go:127
    summary: The rotator moves the stale plan before rendering the prior-plan block, so a render failure aborts once and the retry spawns a session with no block while the prior plan sits unannounced in the archive
  - id: F2
    severity: LOW
    location: internal/loomshed/planwrite.go:116
    summary: New doc comments on NewPlanDirRotator and rotateStalePlanDir are fixed-column wrapped mid-phrase, violating the Comment Line-Break Convention
  - id: F3
    severity: LOW
    location: internal/shedadapters/singlellm.go:89
    summary: New comments in singlellm.go (constructor doc, Call doc, the inline comment in Call) and the changed doc.go paragraph are fixed-column wrapped mid-phrase
  - id: F4
    severity: LOW
    location: internal/shedrecipe/entries_planwrite.go:15
    summary: The changed planWriteEntry doc line breaks mid-phrase at a fixed column
  - id: F5
    severity: LOW
    location: internal/loomshed/doc.go:7
    summary: The new loomshed/doc.go sentence and the changed loomengine/plan.go file-comment sentence break mid-phrase instead of at their clause boundary
  - id: F6
    severity: LOW
    location: internal/shuttleengine/claudeengine/command_test.go:366
    summary: buildResumeCmd notice coverage is pwsh-only, and no command_test row pins the notice inside the fork WithEnv wrap or uses a real notice sentence with its colon and semicolon
  - id: F7
    severity: NIT
    location: internal/loomshed/planwrite.go:136
    summary: The render error stutters, since loomshed and loomengine both wrap it with render prior-plan block
  - id: F8
    severity: NIT
    location: internal/loomengine/priorplan.go:14
    summary: PriorPlanBlock's doc comment breaks at phrase boundaries rather than one sentence per line
---

The range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main` in `_lyx/shed/plan-write-respawn/status.json`), which holds the six card commits.
`go build ./...`, `go vet -tags smoke ./...` and `go test -tags integration ./...` all pass on HEAD.

Every card's Edit and Create targets are touched as specified.
The plan has no `Rename` or `Delete` group, so the per-card mechanical check is the Edit-group blast radius.
Every non-nil `prepareFreshSpawn` supplier (`loomshed.NewPlanDirRotator` and the two closures in `singlellm_test.go`) takes the `func() (string, error)` shape, and every other caller passes an untyped `nil`, `smoke_attachprobe_test.go` included.
The one production construction of the PlanWrite row (`internal/loomcli/wiring.go`) sets `Env.StencilsDir`.
`loomrecipe`'s `seedBouncerStencils` seeds the new stencil, and the other fixtures build their Env through `newTestEnv`, which sets it.
The registry insertion after `loom-rubric-webster-review` shifts nothing any test indexes: `internal/stencilcli`'s integration test indexes positions 0, 4 and 5 and derives its total from `Registry().Names()`.
`sh.Quote` single-quotes the notice on both shells, and `:` and `;` are literal inside single quotes in posix sh and pwsh alike.
The documentation items in cards 2 and 6 are present and match the code.

### [MEDIUM] Render failure after the move loses the prior plan on retry

**Location:** internal/loomshed/planwrite.go:127

**Issue:**
The closure calls `rotateStalePlanDir`, which moves every top-level `.md` file into the archive, and only then calls `loomengine.PriorPlanBlock`.
If the render fails (for example, a stencils directory without `loom-template-prior-plan`, which the `MissingStencilWithPriorFilesIsAnError` subtest exercises), the closure returns an error and the producer aborts, but the files have already moved.
On the next `Call`, after the operator repairs the stencil and resumes, the plan directory holds no top-level `.md` file, so the rotation is a no-op, the amendment is empty, and a fresh session spawns without being told where the prior plan went.
This is the failure the card's error path exists to prevent ("so the producer aborts before spawning a session that would silently rewrite the plan"): the abort holds for one call only.

**Fix:**
Render the block before moving anything.
Split the rotation so the stale-file list and the free archive path are computed without side effects, render the block from them, and only then create the archive and move the files.
A render failure then leaves the plan directory untouched, and the retry renders again.
Extend the missing-stencil subtest to assert the prior files are still in the plan directory after the error.

### [LOW] NewPlanDirRotator and rotateStalePlanDir doc comments are fixed-column wrapped

**Location:** internal/loomshed/planwrite.go:116

**Issue:**
The new paragraph on `NewPlanDirRotator` wraps at a fixed column: "otherwise a blank line followed by / loomengine.PriorPlanBlock", "holds the / loom-template-prior-plan stencil", and a line that ends one sentence and starts the next.
On `rotateStalePlanDir`, the new sentence breaks as "when nothing moved / (absent plan directory, ...)".
CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line on new and changed comments.

**Fix:**
Rewrite the new lines one sentence per line, breaking inside a sentence only at an independent-clause boundary.

### [LOW] singlellm.go and shedadapters/doc.go new comments are fixed-column wrapped

**Location:** internal/shedadapters/singlellm.go:89

**Issue:**
The new constructor paragraph (lines 89–92), the added `Call` doc sentence (line 122: "appended to the prompt of the / spec passed to RunGated"), and the inline comment in `Call` (line 170: "already saw / the spec as composed") all break mid-phrase at a fixed column.
The changed `doc.go` list item (line 32) packs the new sentences onto the tail of an existing line and wraps them mid-phrase, leaving one line holding the end of one sentence, a whole second one, and the start of a third.

**Fix:**
Put each new sentence on its own line, with a break after the semicolon in the inline comment and in the `doc.go` sentence.

### [LOW] planWriteEntry's changed doc line breaks mid-phrase

**Location:** internal/shedrecipe/entries_planwrite.go:15

**Issue:**
The changed line ends with `resolves the row's "gate"/"gate_attempts"` and continues `Config keys through resolveGateSpec` on the next line, a fixed-column break inside a phrase.

**Fix:**
Rewrap the changed sentence semantically: the sentence is being edited, so its line breaks are in scope.

### [LOW] loomshed/doc.go and loomengine/plan.go changed sentences break mid-phrase

**Location:** internal/loomshed/doc.go:7

**Issue:**
The new `loomshed/doc.go` sentence breaks as "naming that archive / to the respawned session's prompt".
The replaced sentence in `internal/loomengine/plan.go:16` breaks as "run as the / Plan-Write producer's fresh-spawn preparation" instead of at its semicolon.

**Fix:**
Put the `doc.go` sentence on one line.
In `plan.go`, break after the semicolon only.

### [LOW] Resume-line and fork-wrap notice coverage is thin

**Location:** internal/shuttleengine/claudeengine/command_test.go:366

**Issue:**
Card 2 asks for notice rows asserting the exact line on both posix and pwsh.
`TestBuildLaunchCmd` has both, but `TestBuildResumeCmd` hard-codes `shell.Pwsh()`, so the posix resume line with a notice is never pinned.
No row combines a notice with `forkSubagents`, so the card's placement rule (the flag sits inside the `sh.WithEnv` wrap) is unpinned: `prepare_test.go` checks only `strings.Contains`.
The test notices are short placeholders, so the real sentences' `:` and `;` never reach an exact-line assertion.

**Fix:**
Add a posix resume row carrying a notice, a fork-mode launch row carrying a notice (asserting the `WithEnv` wrap encloses the flag), and use a real notice constant in at least one exact-line row.

### [NIT] Render error message stutters

**Location:** internal/loomshed/planwrite.go:136

**Issue:**
`loomengine.PriorPlanBlock` wraps its fill error as `loom: render prior-plan block: ...`, and the closure wraps again as `loomshed: render prior-plan block: ...`.
The operator sees "render prior-plan block" twice in one chain.
No written convention forbids per-package prefixes, and `loomengine`'s own layering (`loom: compose plan prompt` wrapped by `loom: PlanSpec`) names a different step at each layer, so the duplicate verb is the only stutter.

**Fix:**
Have the loomshed layer name its own step, for example `loomshed: announce prior plan: %w`.

### [NIT] PriorPlanBlock doc comment breaks at phrase boundaries

**Location:** internal/loomengine/priorplan.go:14

**Issue:**
The first sentence spans three lines, broken after a comma and before a participial phrase and a compound "and".
Neither is an independent-clause boundary, so the convention wants the sentence on one line.

**Fix:**
Put the first sentence on one line.
