# Card 8 — orch-stencils

**Create:**
- `contracts/stencils/orch/orch-template-adopt.md`
- `contracts/stencils/orch/orch-template-handoff-soft.md`
- `plan:internal/orchengine#RenderAdoptPrompt` -> `func RenderAdoptPrompt(stencilsDir string) (string, error)`
- `plan:internal/orchengine#RenderSoftHandoffInstruction` -> `func RenderSoftHandoffInstruction(stencilsDir, handoffPath string) (string, error)`

**Edit:**
- `contracts/stencils/stencils.go`
- `internal/orchengine/prompt.go`
- `internal/orchengine/prompt_test.go`

**Intent:**
Add two orch stencils, each with a leading HTML comment in the existing orch stencils' style naming its renderer, its single-line requirement and its markers.

- `orch-template-adopt`: the launch prompt for an adopted session, with no markers.
  One line telling the session it is now hosted by `lyx orch` as the hub orchestrator, that it continues its current work, and that a message asking for a handoff names the file to write and then ends its turn.
  It renders to one line so the open-risk fallback (resume without a prompt, then send it) stays available.
- `orch-template-handoff-soft`: the soft-trigger handoff request, with the one marker `{{.handoff_path}}`, one line.
  It tells the session the cycle is optional: if work a `/clear` would lose is in flight (a subagent or fork mid-task, an edit half done), reply with the single word `DEFER` and nothing else, and write no file;
  otherwise run `/scribe:handoff {{.handoff_path}}`, listing every long-lived watcher (background shell, Monitor) with its id and purpose so the resumed session re-arms it, since watchers are not in-flight work.

Register both in `stencils.go` with an embed variable each and an `entries` row, so `TestRegistry_MatchesOnDiskTree` covers them.
In `prompt.go`, add the two stencil-name constants, `RenderAdoptPrompt` (single line) and `RenderSoftHandoffInstruction` (single line, `handoff_path` filled), and add both stencils to the file doc comment.

Tests: each renders from the shipped default to one line;
the soft instruction carries the handoff path and the literal `DEFER`;
a multi-line override of each fails naming its stencil.

**ImpactSummary:** Two new registry rows and two new render functions; existing stencils and renders are unchanged.
