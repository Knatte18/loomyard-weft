# Card 5 — orch-stencils

**Create:**
- `contracts/stencils/orch/orch-template-start.md`
- `contracts/stencils/orch/orch-template-handoff.md`
- `contracts/stencils/orch/orch-template-resume.md`
- `internal/orchengine/prompt.go`
- `internal/orchengine/prompt_test.go`

**Edit:**
- `contracts/stencils/stencils.go`

**Uses:**
- `internal/websterengine/render.go`
- `internal/stencilstore/stencilstore.go`
- `internal/stencil/stencil.go`
- `internal/orchengine/config.go`

**Intent:**
Ship the three orch stencils and the functions that render them, per the Stencil Ownership Invariant (read at call time from a told directory) and D13 (each injected stencil renders to one line).

Each stencil opens with the repo's usual leading `<!-- -->` comment naming its filler and markers:

- `orch-template-start.md` — the fresh-launch prompt (D6 case 3), handed to shuttle as the launch prompt file, so multi-line is fine.
  It tells the session it is the hub orchestrator running in the hub's prime worktree, to read the board first, and that `lyx orch` cycles its context automatically: when a message asks it to write a handoff, it writes one to the named path and ends its turn.
  It carries no markers.
- `orch-template-handoff.md` — the handoff instruction D10 types in `handoff-requested`: `/scribe:handoff {{.handoff_path}}` followed on the same line by focus text.
  The focus text tells the session its context is about to be cleared and asks the handoff to list every live background watcher (Monitor, background Bash, subagent) with its id and purpose, so the resumed session can check each one before re-arming it — the decision record's first open risk.
- `orch-template-resume.md` — the resume prompt, both typed in `resuming` (D10) and used as the launch prompt when a fresh launch resumes (D6 cases 1 and 2).
  One line: it tells the session it is the hub orchestrator resuming after a context cycle, to read `{{.handoff_path}}` and continue from it, and to check each listed watcher before re-arming it.

Register all three in `contracts/stencils/stencils.go`: one `//go:embed` var each (`OrchTemplateStart`, `OrchTemplateHandoff`, `OrchTemplateResume`) and three `entries` rows named after their files.

`prompt.go` adds three functions, each reading its stencil through `stencilstore.Read(stencilsDir, name)` and filling it with `stencil.Fill`, which already drops the leading comment, exactly as `websterengine`'s renders do:

- `RenderStartPrompt(stencilsDir string) (string, error)`
- `RenderHandoffInstruction(stencilsDir, handoffPath string) (string, error)`
- `RenderResumePrompt(stencilsDir, handoffPath string) (string, error)`

The two injected renders trim surrounding whitespace and return an error naming the stencil when the result still contains a newline or carriage return: shuttle's `Send` refuses multi-line text, and an operator override that breaks the rule should fail at render time with a message that names the file to fix.

`prompt_test.go` seeds a temporary stencils directory with the shipped defaults (the `contracts/stencils` registry's `Default` bytes written at `stencilstore.Path`) and covers:
each render succeeding;
the handoff and resume renders each being a single line that contains the handoff path;
the handoff render starting with `/scribe:handoff `;
and a multi-line override of the resume stencil failing with an error that names it.

**ImpactSummary:** Adds three registry rows; `lyx stencil list` and the root pre-run's seeding pick them up, and no existing stencil changes.
