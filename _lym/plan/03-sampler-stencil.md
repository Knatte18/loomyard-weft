# Card 3 — sampler-stencil

**Create:**
- `contracts/stencils/sampler/sampler-template-agent.md`
- `internal/samplerengine/spec_test.go`
- `plan:contracts/stencils#SamplerTemplateAgent` -> `var SamplerTemplateAgent []byte`
- `plan:internal/samplerengine#Role` -> `const Role = "sampler"`
- `plan:internal/samplerengine#SessionInputs` -> `type SessionInputs struct`
- `plan:internal/samplerengine#SessionSpec` -> `func SessionSpec(in SessionInputs, cfg Config, reg modelspec.Registry) (shuttleengine.Spec, error)`
- `plan:internal/samplerengine#TestSessionSpec` -> `func TestSessionSpec(t *testing.T)`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`

**Extends:**
- `internal/samplerengine/doc.go`

**Uses:**
- `internal/darnengine#DarnTable`
- `internal/landingshed#PRReview.sessionSpec`
- `docs/sandbox-howto.md`

**Intent:**
D5, D8 and D9: the sampler agent's opening prompt and the spec its session starts under.
`sampler-template-agent.md` is a new stencil under `contracts/stencils/sampler/`, opening with the leading HTML comment the other stencils carry (what fills it, and that every marker is a top-level non-empty substitution with no conditionals).
Its markers are `parent_directive`, `edit_directive`, `slug`, `task_worktree` and `decision_record_path`; it carries paths only, never file content (`PATTERN-stencil-ownership`).
It tells the agent (D8): read the decision record at `{{.decision_record_path}}` as the task's brief, say in a line or two what it understood, and wait for the operator; make the result with the operator and never write a plan or a record of its own; work only in `{{.task_worktree}}` and commit as it goes, one commit per settled step; write any board result through `lym board` as a note, since a run's agent may never create or promote a task or change its own run's entry, which the board refuses, and the operator promotes a note when they want one; record each thing the operator settles that changes the task's scope with `lym loom decision add --by operator`; run `lym loom sampler-done` only when the operator says the work is done, after committing, since lym ends the build only on a clean worktree.
For a trial (D9): never run a dev build of lym in this hub, since reed runs one tmux server per hub and a newer build re-pins its server-wide bindings, and try one only in another hub on the machine, on lym's own repository the sandbox hub `docs/sandbox-howto.md` describes; start any tmux the trial needs outside lym on a server of its own, `tmux -L <name>` with a name holding the slug `{{.slug}}`, and kill that server before running `lym loom sampler-done`; commit what the trial settled as an ordinary change.
It names no step the agents' settings deny (`PATTERN-refusal-way-forward`): no push, no `gh`, and no ask-the-operator instruction beyond waiting for the operator's message in the pane.
`stencils.go` embeds it as `SamplerTemplateAgent`, adds `{"sampler-template-agent", &SamplerTemplateAgent}` to `entries` after `darn-template-write`, and maps the role `sampler` to it in `roleOpeningStencils`, so `TestStencils_OpeningStencilsRenderParentDirective`, `TestStencils_OpeningStencilsRenderEditDirective`, `TestStencils_NoUnboundedAskOperator`, `TestStencils_NoSkillLoadInstructions` and `TestRegistry_MatchesOnDiskTree` cover it unchanged.
`samplerengine.Role` is the agent-name role of the session (`PATTERN-agent-name`).
`SessionInputs` holds `StencilsDir`, `ParentName`, `Slug`, `WorktreeRoot` and `DecisionRecordPath`.
`SessionSpec` refuses an empty `StencilsDir`, `Slug`, `WorktreeRoot` or `DecisionRecordPath` by name, parses and resolves `cfg.Agent` against `reg` (an unresolvable spec is an error naming `agent`), reads the stencil through `stencilstore.Read` at call time, renders `parentdirective.Directive(StencilsDir, ParentName, true)` (the interactive form) and `editdirective.Directive`, fills the stencil with `stencil.Fill`, and returns a `shuttleengine.Spec` with `Interactive` and `Unawaited` true, `DenyPublish` true (so the session gets `internal/shellguard`'s existing publish row and steer), `Role` `Role`, `Segment` `segmentcolor.Darn` (the build segment's colour), `Skills` `scribe:prose` and `scribe:conversation`, and `ModelChoice` from `shuttleengine.NewModelChoice` (`PATTERN-shuttle-provider-seam`, `PATTERN-parent-directive`), the shape `PRReview.sessionSpec` builds.
`spec_test.go` carries `//go:build tier0`.
`TestSessionSpec` seeds the shipped stencils with `stencilkit.Seed`, builds a spec with parent `hub:orch` and a registry resolving `opus[high]`, and asserts the spec's flags, role, segment and skills, and that the prompt holds the interactive parent directive the same call to `parentdirective.Directive(dir, "hub:orch", true)` renders, the edit directive, the record path and the slug; that it names `lym loom sampler-done`, `lym loom decision add`, `--by operator`, a note through `lym board`, `tmux -L` and the sandbox hub (D8, D9 acceptance); and that it holds neither `git push` nor `gh pr`.
Rows refuse an empty `Slug` naming it and an unresolvable `agent` naming the key.
`samplerengine/doc.go` gains a paragraph on `SessionSpec` and the stencil.

**ImpactSummary:** one stencil joins the registry and the role table; the stencil-wide scans run over it unchanged, and `SessionSpec` has no caller yet.

**Commit:** 3: sampler-template-agent stencil and the sampler session's spec
