# Card 10 — stencils-no-watcher

**Edit:**
- `contracts/stencils/orch/orch-template-start.md`
- `contracts/stencils/orch/orch-template-resume.md`
- `contracts/stencils/orch/orch-template-adopt.md`
- `contracts/stencils/orch/orch-template-handoff.md`
- `contracts/stencils/orch/orch-template-handoff-soft.md`
- `internal/orchengine/prompt_test.go`
- `internal/orchcli/doc.go`

**Intent:**
Take watcher re-arming out of the orch stencils and teach the session the batten notice rule (D11, D14).

- Start stencil: the orch relies on driver messages and batten notices and never starts a polling shell or Monitor for a run; a line starting with `[batten notice]` is a batten notice, checked with `lyx batten status <slug>` and acted on.
  Since compact is the default cycle (D5, D6), it also drops any wording that assumes `/clear` is how the context is cycled, and adds nothing beyond that correction.
- Resume stencil: drop "check each watcher it lists before re-arming it", and add the same no-watcher rule in one clause.
- Adopt stencil: add that the previous process's background shells and Monitors are gone and none is re-armed, plus the no-watcher rule.
- Handoff and soft handoff stencils: drop the watcher list; the soft stencil keeps its DEFER rule.
- The single-line stencils still render to one line.

`internal/orchcli/doc.go`'s run-loop step 2 is rewritten to the same rule: no watch loop of the orch's own; batten notices arrive as typed turns from the orch watcher, and the orch checks the run with `lyx batten status <slug>`.

`prompt_test.go` asserts the start prompt names `[batten notice]` and `lyx batten status`, the resume, handoff and soft renders no longer ask for watchers, and the adopt render says none is re-armed, each still one line where it was.

**ImpactSummary:** Rewords five orch stencils and one doc step; any test asserting the old watcher wording is updated here.
