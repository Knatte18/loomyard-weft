```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced the diff's changes to the board entry's goal points.

**Decisions.** There is no plan and no `Added after Discussion` entries, so the goal points come from the board entry under `## Goal`:
- **G1:** nothing is ever built in `.scratch`, `.lym` or `_lym`, which are scratch and record folders.
- **G2:** no agent creates a git worktree, clone or checkout under them.
- **G3:** `.scratch` stays a link and does not become a real directory.
- **G4 (To do):** tell agents, in the instructions lym ships (stencils and skills), never to build, restore or compile under `.scratch`, `.lym` or `_lym`, and never to create a worktree, clone or checkout there.

**Per change:**
- **C1, both edit-directive copies** (`contracts/stencils/engine/claude/edit-directive.md:18-20`, `contracts/stencils/engine/codex/edit-directive.md:18-20`): the two build-resolution lines are replaced by the ban. This builds G1, G2 and G4.
  Removing the build-resolution lines is forced by the ban. They told agents how to make a build under `.scratch/` work, which contradicts G1.
- **C2, orch role note** (`contracts/orchrole/orch-role.md:452-453`): the same ban goes into the orch notes. This builds G1, G2 and G4 for orch sessions, whose instructions come from the role text.
- **C3, package doc** (`internal/editdirective/doc.go:16-19`): forced by C1. The package doc restates what the directive says, and C1 made the old build sentences stale.
- **C4, test** (`internal/editdirective/editdirective_test.go:20-21, 41-43`): the `scratchBuildBan` assertion traces to C1, so it is forced by a traced change. Whether it is good test economy is the code-tests lens's question, not scope.
- **C5, commit e3ca0c11:** it re-breaks the C3 doc comment, so it is forced by C3.

**Per decision:**
- **G1:** built by C1 and C2.
- **G2:** built by C1 and C2.
- **G3:** needs no change. The diff leaves the link untouched; no fabric code is in the range.
- **G4:** built by C1, which reaches every spawned role on both engines through the opening stencil's required marker, and by C2 for orch.
  Skills: no skill under `plugins/ly/skills/` tells an agent to build or check out under these folders. They use `.scratch/` only for drafts (`board-policy/SKILL.md:28`).
  The heddle shipped layer has no `.scratch` instruction either.
  `contracts/stencils/retro/retro-template-task.md:23` passes `--work .scratch/forkaudit`, which is a directory for intermediate files, not a build or checkout (`tools/forkaudit/forkaudit.sh`, the `--work` flag doc). It does not contradict the ban.
  No uncovered instruction remains, so I raise no finding.

**Findings:** none. Every change traces to G1, G2 or G4 or is forced by a traced change, and every goal point is built or needs no change.

Tool calls:
1. Searched `contracts/heddle*` and `internal/heddleengine` for `.scratch`, to check whether heddle's shipped layer carries a `.scratch` instruction G4 would need. The manifest did not cover heddle.
2. Searched heddle's production files for `scratch`. The hits concern the pair's never-tracked scratch directory and a render lock, not agent instructions.
