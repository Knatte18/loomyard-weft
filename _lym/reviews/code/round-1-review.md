---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: internal/heddleengine/agentdefaults.go:187
    summary: The decision record asks for model and effort defaults rendered from agents.yaml, but the diff renders only the effort default.
    fix: Leave the code in place and defer this finding to the run's parent. The summary says agents.yaml holds no model key, so the model half of the Wanted bullet has no source to render. The parent records or refuses that correction with lym loom decision add. If it is refused, a follow-up adds a model key to agents.yaml and a modelDefault member to agentDefaultsEntry.
    origin: handler
  - id: F2
    severity: LOW
    class: design
    location: internal/heddleengine/overview.go:37
    summary: RenderOverview ignores DerivedWarning and resolves agents.yaml from Dirs{Board} only, so HEDDLE.md silently omits lym-agent-defaults when agents.yaml is invalid and can show a default the worktree file does not get.
    fix: In RenderOverview, pass the derived warning into overviewText and, when it is set, write a one-line note under the claude-settings section that the effort defaults are left out and why. Document in the RenderOverview comment that a machine-layer agents.yaml is not read there, or resolve the dirs the way Render does. Add a row to TestRenderOverview with agents.yaml holding no default, asserting the render succeeds and the note shows.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/heddleengine/replayer_test.go:75
    summary: No test drives Replayer.Apply with a HEDDLE.md in the request or a commit whose overview differs from origin's, so the claim that the commit's own overview never lands is unpinned.
    fix: Add a replayer subtest that passes a ReplayFile for HEDDLE.md holding stale text beside a heddle.json change, then asserts the HEDDLE.md on disk equals the text RenderOverview produces from origin's merged store.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/heddleengine/overview_test.go:41
    summary: Two RenderOverview branches have no row, fence widening for text holding a backtick run and a JSON-object entry whose text is not an object. The repository-over-derived precedence on a conflicting key is also unasserted.
    fix: Add a row to TestRenderOverview with an entry whose text holds a triple-backtick run, asserting a four-backtick fence. Add a claude-settings entry with non-object text, asserting it shows as written in a json block. Add a row to TestRenderClaudeSettings in settings_test.go where a repository entry sets effortLevel, asserting its value wins over the derived one.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: cmd/lym/main.go:161
    summary: The registration comment says the heddle replayer merges heddle.json by entry, but it now also claims HEDDLE.md and renders it afresh after each replay.
    fix: Reword the comment on the RegisterBoltReplayer call for heddleengine.Replayer to say it merges heddle.json by entry and renders HEDDLE.md afresh after each replay.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: pattern/PATTERN-heddle-instructions.md:36
    summary: The pattern edit deleted the blank line before the ## Lym instructions in code heading, so the HEDDLE.md bullet runs into it.
    fix: Insert one blank line between the HEDDLE.md bullet and the ## Lym instructions in code heading.
    origin: lens:code-affected-set
---

### [MEDIUM] Model default from agents.yaml is not rendered

**Location:** internal/heddleengine/agentdefaults.go:187
**Issue:** The record's Wanted section says the machine-wide model and effort defaults are rendered into the heddle file from agents.yaml.
The diff derives only `effortLevel` and the subagent effort variable.
The change description names this under Design decisions: agents.yaml as `agent-skills-and-effort` landed it holds efforts and no model.
That is a deviation from the amended entry that no `Added after Discussion` entry records, so it is an unrecorded parent correction.
The fix is the parent's to record or refuse, and the code stays.

### [LOW] HEDDLE.md omits the effort defaults silently and may disagree with the worktree file

**Location:** internal/heddleengine/overview.go:37
**Issue:** `Render` reports `DerivedWarning`, and `RenderOverview` drops it.
With an agents.yaml that fails to load, the overview shows no `lym-agent-defaults` entry and says nothing.
`RenderOverview` also resolves agents.yaml from the Bolt checkout alone, while `Render` adds the anchor.
A machine-layer value of the efforts table applies only when both are set, so the overview can show a default the worktree file does not carry.
The overview is meant to show everything heddle sets, so an unexplained gap or a differing value defeats it.
No test covers the invalid agents.yaml case for the overview.

### [LOW] Replayer's discarding of a commit's own HEDDLE.md is untested

**Location:** internal/heddleengine/replayer_test.go:75
**Issue:** `Apply` renders the overview from origin's state and never reads the commit's HEDDLE.md, which is what keeps two hubs' overviews from conflicting.
Every replayer request passes only `heddle.json`, and the fabric replay test commits `heddle.json` alone locally.
Nothing fails if `Apply` starts taking the commit's overview.

### [LOW] Overview branches without a row

**Location:** internal/heddleengine/overview_test.go:41
**Issue:** `entryBlock` widens the fence past three backticks and passes a non-object JSON entry through as written, and neither has a row.
`Layers.Ordered` places derived entries before repository ones so the repository wins a conflicting key, and `settings_test.go` only merges non-conflicting keys.

### [LOW] Stale replayer registration comment

**Location:** cmd/lym/main.go:161
**Issue:** The comment names heddle.json alone.
The replayer's claim and its render step both changed in this diff.

### [LOW] Blank line lost before a heading

**Location:** pattern/PATTERN-heddle-instructions.md:36
**Issue:** The hunk removes the blank line that preceded `## Lym instructions in code` and adds none.

## Rejected

- Lens code-correctness, MEDIUM: `promptSuggestionEnabled` and `attribution` reach a session only through the worktree file, so a worktree without a heddle render loses them. The Wanted bullets ask for exactly this move, and the change description lists it under Risks.
- Lens code-correctness, `Replayer.Apply` leaving `heddle.json` saved when the overview render fails: the error returns to fabric, whose existing apply-failure fixture covers a dirty tree, and the lens did not show a defect.
- Lens code-tests, `saveLocked`'s "saved but not rendered" error and the overview read and write error paths have no row: both are plain error wraps with no branch of their own, and a seam for them would cost more than it pins.
- Lens code-tests, `lym heddle remove`, override verbs and `ChangeStore` do not assert the overview: all go through `saveLocked`, which the set row covers.
- Lens code-affected-set, no `hubreconcile` reinterpretation for `session_scope.enabled` governing less: no hub file value changes meaning, so there is nothing to rewrite.
- Declared PATTERN changes: the directive declares none, so none are owed.
