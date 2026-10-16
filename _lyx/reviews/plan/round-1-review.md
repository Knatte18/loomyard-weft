---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-role-file-repository-rules.md (Intent)
    summary: Card 1 adds a channel that inlines ORCHESTRATOR.md into the orch role file at every delivery, but its Intent never states what that channel lets through or what bounds it, as D6's Bound records
    fix: In card 1's Intent, after the paragraph on readRepositoryRules, add one sentence per line stating D6's Bound - the render reads one fixed path in the prime worktree (Paths.RepoRulesPath, set from WorktreePath), never a task pair's copy, and inlines the file with no size cap, adding no verb or flag; the file can carry any text into the orch's role file at every delivery and reaches the prime through any landing, including one the alignment agent lands with no orch turn; the only guard against a rule that contradicts the role is the `## This repository` section's own text, which tells the orch to follow the role and raise the conflict to the operator.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-role-file-repository-rules.md (Edit, internal/orchcli#orchPaths)
    summary: Card 1 rewrites the file comment of internal/orchcli/paths.go, which lies in no member span, while its targets name only the orchPaths member of that file
    fix: In card 1's Edit group, replace the entry `internal/orchcli#orchPaths` with the file path `internal/orchcli/paths.go`, which covers both the file comment and orchPaths; keep `internal/orchcli#PrimePaths` under Uses. Checked with lyx loom validate-plan --overlay - no finding beyond the baseline's informational create-new-unit.
---

### [MEDIUM] Card 1 states no bound for the ORCHESTRATOR.md inlining

**Location:** `_lyx/plan/01-role-file-repository-rules.md`, `**Intent:**`

**Issue:** The rubric's attack-surface bullet requires the card that adds a routing edge to state in its Intent what it lets through and what bounds it.
Card 1 adds one: tracked repository text that reaches the orch's prompt at every delivery, with no size cap.
D6 writes out the Bound (one fixed path in the prime, never a task pair's copy, no size cap, no verb or flag, reachable through any landing including an alignment-approved one, guarded only by the section's own text).
Card 1's Intent covers the read semantics, the refusals and the section text, but never states what can now pass into the role file or what limits it.
The section-text sentence in the stencil paragraph describes the guard's wording but never says it is the only guard.

### [LOW] paths.go file comment edit is outside every targeted span

**Location:** `_lyx/plan/01-role-file-repository-rules.md`, `**Edit:**` entry `internal/orchcli#orchPaths`

**Issue:** The Intent says "`paths.go`'s file comment gains that this one path is a repository document joined at the worktree root, not lyx state."
That comment is line 1 of `internal/orchcli/paths.go`, a free-standing header.
`lyx quarry resolve` puts `orchDirName` at lines 13–14, `PrimePaths` at 16–19 and `orchPaths` after them, so no member span holds the comment.
The card targets only the `orchPaths` member of that file, so the header edit has no target.
Adding the file self glyph beside the member trips `redundant-file-target`, so the fix replaces the member with the file path.

## Rejected

- Card 4's board note: a board write has no repository file to name as a target, and the plan writer stencil allows "a card that writes another board entry".
  Carrying it in the Intent of a `Create`/`Edit` card is correct typing.
  A `Custom` card would need a target and could not combine with the card's other groups (`card-custom-not-alone`).
  The write touches only the new note `reed-673-workaround-home`, never this task's entry nor `reed-add-parentless-placement`'s, so the board-lock bullet does not fire.
  The payload's labels `bug` and `reed` hold one type label (`bug`) and one plain label in `board.yaml`, so `validateLabels` accepts the note.
- Card 1 re-sign arrows: `RenderRoleFile` carries its arrow.
  `Watcher.renderRoleFile`, `ResumeContext`, `orchCLI.startCmd` and `orchPaths` keep their signatures, and `Paths` is a struct, which takes no arrow.
- Card 1 test economy: every planned row extends an existing test.
  The watcher, resume-context and start rows each reach a different render site that D6's failure scenarios name.
  The one new test, `TestPrimePaths_RootsRepositoryRulesAtTheWorktree`, pins the worktree-root join, which `newTestCLI` bypasses by setting the path by hand, and the `internal/orchcli` budget row (28, at its count today) rises with it.
  Cards 2 and 3 plan no assertion on static prose.
- Card order: card 1 adds the `repository_rules` marker, the `## This repository` section and the leading-comment sentence to `orch-template-role.md`.
  Card 2 rewrites other sections of that stencil and keeps card 1's additions, so it depends only on an earlier card.
- Reachability: the unreadable-file refusal is reached by a directory at the rules path in `TestRenderRoleFile_RefusalsWriteNoFile`, `TestResumeContext_FailureReturnsErrorAndWritesNoMark`, `TestWatcher_FailingRenderAbortsWithoutClear` and `TestStart_Refusals`.
  The start row reaches `RenderRoleFile` in the `StartRelaunch` branch: with no strand, no prior check refuses (`internal/orchcli/start.go` 200–282).
  The missing-marker refusal is reached by the `{{.command_index}}`-only override row.
  The `budget` line is brief prose with no Go branch.
  The watcher `role` row's new `wantReason` matches the marker-check error that `{{.nope}}` produces.
- Verify coverage: the section runs every targeted package.
  `internal/shedadapters` renders the bouncer brief, `contracts/stencils` and `cmd/lyx` hold the marker, role-skill and denied-command scans, and `internal/lyxcwd` scans `CLAUDE.md`.
  No other package's test reads the edited stencils' content or the real `CLAUDE.md`; `TestMerriamBase` uses a temp worktree.
  The order matches the plan template's.
- Fidelity: D1–D9 and every Constraint are carried.
  D9's board note is in card 4, the other landing order is left to Merge-Review as D9 assigns, and D6's three render sites each get a failure row.
  Card 2's reading of the "no `update-plugins.sh`" criterion as the stencil's own text follows D7, which puts that rule in this repository's `ORCHESTRATOR.md`.
