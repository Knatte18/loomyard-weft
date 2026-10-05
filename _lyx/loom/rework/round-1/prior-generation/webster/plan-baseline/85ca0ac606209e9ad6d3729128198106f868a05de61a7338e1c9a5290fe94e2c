# Card 12 — The orch role stencil, the note template and the one-line pointers

**Create:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/orch/orch-template-note.md`
- `plan:internal/orchengine#RenderRoleFile` -> `func RenderRoleFile(stencilsDir, path string) error`
- `plan:internal/orchengine#RenderNoteTemplateFile` -> `func RenderNoteTemplateFile(stencilsDir, path string) error`

**Edit:**
- `contracts/stencils/stencils.go`
- `contracts/stencils/orch/orch-template-start.md`
- `contracts/stencils/orch/orch-template-resume.md`
- `contracts/stencils/orch/orch-template-adopt.md`
- `contracts/stencils/orch/orch-template-handoff.md`
- `contracts/stencils/orch/orch-template-handoff-soft.md`
- `internal/orchengine/prompt.go`
- `internal/orchengine/launch.go`
- `internal/orchengine/state.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/paths.go`
- `internal/orchcli/start.go`
- `internal/orchcli/doc.go`
- `internal/orchengine/prompt_test.go`
- `internal/orchengine/launch_test.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchcli/start_test.go`

**Uses:**
- `internal/shuttleengine#Spec`
- `internal/orchengine#Paths`
- `internal/stencilstore#Read`

**Intent:**
D3, D4 and D5: the orch's whole procedure lives in one role stencil that reaches the session as a rendered file and a one-line pointer, and `scribe:handoff` gives way to the orch note.

`orch-template-role.md`, multi-line, one section per D3 theme:
- the run loop: `lyx batten run <slug>` from the prime, detached from the orch's shell with `setsid nohup` until `lyx batten start` exists; `lyx batten status`, `pause` and `goto`; a line starting `[batten notice]` is the wake-up, so no polling shell or Monitor;
- parent-review: a one-shot fork per brief, checking scope against the board entry only, submitting with the brief's `lyx loom review` command, and no second fork on a repeat notice;
- escalations from a child: `lyx loom circling continue|accept`, `lyx loom goto` or `lyx shed goto` for a fresh budget, `lyx loom decision` for a design call the parent makes, then a message telling the driver to resume;
- PR-Gate: the orch approves or rejects itself after a one-shot reviewer fork, judging the run against its own board goal and its tests, never widening scope through rejects, fixing a small goal defect on the task branch and putting new hardening on the board;
- after landing: amend the board entries the run superseded;
- the board: policy only (a proposal goes on the board as a note at once, fewer and larger tasks, a small finding folds into an open entry whose work overlaps, triage drafts stay off the board), pointing at `ly:board` for the mechanics without restating them;
- bugs in lyx go to GitHub issues; design, features and hardening go to the board;
- act and fix without asking, ask the operator only about what to build, and relay an operator's instruction to a strand in the operator's own words and scope;
- every status report on a run gives its `cd <pair> && lyx reed attach` command;
- messaging: every lyx agent is reachable by name through `ListAgents` and `SendMessage` under `internal/agentname`'s form, so the orch messages a driver or writer directly and never types into a pane;
- which role can do what: a table naming, per spawned role, the limits its stencil sets (for example, the Webster master runs no git) and who carries out an instruction outside them.
It names no denied recovery command (`PATTERN-no-denied-recovery`) and asks no agent to load a skill.

`orch-template-note.md`, multi-line: one heading per item with one sentence on what belongs there: what the orch is doing right now, including any fork or edit half done; what is agreed with the operator as the next step, if anything; questions waiting on the operator.
It says runs, the board and anything else lyx can read stay out of the note.

The one-line stencils, each still one line:
- `orch-template-start` — read `{{.role_path}}` in full as your procedure, then read the board and continue;
- `orch-template-resume` — read `{{.role_path}}`, then read the note `{{.handoff_path}}` and continue from it;
- `orch-template-adopt` — read `{{.role_path}}`, then continue the work you were doing; the previous process's background shells and Monitors are gone and none is re-armed;
- `orch-template-handoff` — write the orch note to `{{.handoff_path}}` following the template `{{.note_template_path}}`, then end your turn;
- `orch-template-handoff-soft` — keeps its `DEFER` reply, and otherwise asks for the note exactly as the plain request does.
None names `scribe:handoff`, a parent-review fork or batten notices any more; that procedure is in the role stencil.

`orchengine.Paths` gains `RolePath` and `NoteTemplatePath`, which `orchcli/paths.go` declares as `role.md` and `note-template.md` under `.lyx/orch/` (`PATTERN-told-geometry`, `PATTERN-durable-vs-ephemeral-state`).
`RenderRoleFile` and `RenderNoteTemplateFile` read their stencil through `stencilstore.Read`, fill it, and write the file, creating `Paths.Dir`.
The pointer renders take the role path (and, for the note requests, the note template path) as told arguments and all render to one line; `RenderStartPrompt` becomes single-line too.
Every delivery re-renders its file first, so a stencil edit applies from the next delivery: `start` renders the role file before it launches, the watcher renders the note template file before it types a note request, and renders the role file before it types the resume pointer after `/clear`.
A render failure is handled where the stencil render failure is handled today.

`start` (D7, launch path): `orchSpec` sets `Skills` to `orchSkills` — `scribe:prose`, `scribe:conversation`, `ly:board`, declared beside `orchStrandName` — and `SkillLoadTimeout` to the orch config's `HandoffTimeout()`, for a fresh launch with or without a note and for `--adopt`; the spec's prompt is the pointer line `ChooseStartPrompt` or the adopt render picks.
Skills therefore load before the pointer on every launch path, through shuttle (cards 1 and 2), and nothing is typed into the pane from `start` (`PATTERN-orch-pane-single-writer`).

Tests: the role stencil carries a heading for each D3 theme and the note template its three headings; each pointer is one line naming the role path and, where it applies, the note and the note template path; no orch stencil names `scribe:handoff`; `ChooseStartPrompt` keeps its source order with the new pointers; the watcher's note request and post-clear resume write the template and role files before typing; the orch spec carries the three skills and the handoff timeout.
These replace `prompt_test.go`'s `/scribe:handoff` prefix assertion and its start-prompt assertions on parent-review and batten-notice text, which are reworded messages.
`orchengine/doc.go`'s start-prompt-order section and `orchcli/doc.go` describe the role file, the pointers and the note.

**ImpactSummary:** the orch stencil family and its renders, two told paths and the orch launch spec; the watcher's cycle machine is unchanged apart from the files it renders before typing.

**Commit:** 12: orch: role stencil, note template and one-line pointers
