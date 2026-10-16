# Card 1 — role-file-repository-rules

**Edit:**
- `internal/orchengine#Paths`
- `internal/orchengine#RenderRoleFile` -> `func RenderRoleFile(stencilsDir, path, index, rulesPath string) error`
- `internal/orchengine#Watcher.renderRoleFile`
- `internal/orchengine#ResumeContext`
- `internal/orchengine/doc.go`
- `internal/orchengine#testPaths`
- `internal/orchengine#TestRenderFiles_WriteWithoutLeadingComment`
- `internal/orchengine#TestResumeContext_FailureReturnsErrorAndWritesNoMark`
- `internal/orchengine#TestWatcher_FailingRenderAbortsWithoutClear`
- `internal/orchcli/paths.go`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli/doc.go`
- `internal/orchcli#newTestCLI`
- `internal/orchcli#TestStart_Refusals`
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/orchengine#repositoryRulesMarker`
- `internal/orchengine#noRepositoryRules`
- `internal/orchengine#readRepositoryRules`
- `internal/orchcli#TestPrimePaths_RootsRepositoryRulesAtTheWorktree`

**Rename:**
- `internal/orchengine#TestRenderRoleFile_RefusesStencilWithoutIndexMarker` -> `internal/orchengine#TestRenderRoleFile_RefusalsWriteNoFile`

**Uses:**
- `internal/orchengine#seedStencils`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#errBoom`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#PrimePaths`
- `internal/testkit/locationkit#Location`

**Intent:** Render each repository's orch-only rules file, `ORCHESTRATOR.md`, into the orch's role file at every role-file render (D6).

`Paths` gains the field `RepoRulesPath string`, commented as the repository's `ORCHESTRATOR.md`, read at every role-file render.
`orchPaths` sets it once to `filepath.Join(location.WorktreePath(), "ORCHESTRATOR.md")`: the prime worktree's root beside `CLAUDE.md`, never `AnchorPath()`, which may be a subpath.
`paths.go`'s file comment gains that this one path is a repository document joined at the worktree root, not lyx state.
`orchengine` derives no path (`PATTERN-told-geometry`).

`readRepositoryRules` reads the file at `path`.
A missing file (`errors.Is(err, fs.ErrNotExist)`), or one holding only whitespace, returns `noRepositoryRules`.
Any other read error, a directory at that path included, returns `orch: read repository rules <path>: <err>; fix the file or remove it`.
Otherwise it returns the content as it stands, untrimmed (D6).

Bound (D6): the render reads one fixed path in the prime worktree, `Paths.RepoRulesPath` set from `WorktreePath()`, never a task pair's copy, and inlines the file with no size cap; it adds no verb or flag.
The file can carry any text into the orch's role file at every delivery, and it reaches the prime through any landing, including one the alignment agent lands with no orch turn.
The only guard against a rule that contradicts the role is the `## This repository` section's own text, which tells the orch to follow the role and raise the conflict to the operator.

`RenderRoleFile` takes `rulesPath`.
It refuses a role stencil whose top-level markers lack either `command_index` or `repository_rules`, with the existing refusal's wording naming the stencil and the missing marker: restore the marker, or delete the override so the shipped stencil applies.
It then calls `readRepositoryRules(rulesPath)`, returns its error unwrapped and writes no file on it, and fills `repository_rules` with the result beside `command_index`.
The markers check loops over the two required markers, `command_index` first and then `repository_rules`, in place of the one `slices.Contains` call, so a stencil missing both is refused naming `command_index`.
Its doc comment is rewritten to match: two markers, the index and the rules file read from `rulesPath`, each missing marker refused naming the stencil, and a rules read failure returned with no file written.
The callers pass the told path: `Watcher.renderRoleFile` passes `w.paths.RepoRulesPath`, `ResumeContext` passes `p.RepoRulesPath`, and `orchCLI.startCmd`'s relaunch branch passes `c.paths.RepoRulesPath`.
So `start` refuses on an unreadable file before it starts a session, the watcher's cycle returns to idle with the reason, and `resume-context` fails and writes no resume mark, each through its existing render-failure branch.
`Watcher.renderRoleFile` rewords its render wrap from `role stencil %s failed to render: %w` to `role file failed to render: %w`, since the failing source may now be the rules file; every role-stencil error it wraps already names `orch-template-role`.
Its doc comment names the index, the role stencil or `ORCHESTRATOR.md` as the failing source.

The role stencil `contracts/stencils/orch/orch-template-role.md` gains, as its last section, `## This repository`, ending with the marker line `{{.repository_rules}}`.
The section's text says, one sentence per line: this section holds the repository's `ORCHESTRATOR.md`, its orch-only rules; it adds to this role and overrides none of it; a rule there that contradicts this role is raised to the operator, and this role is followed meanwhile.
The leading comment's marker sentence names two markers: `command_index`, filled with the operator command index, and `repository_rules`, filled with the prime's `ORCHESTRATOR.md` or a one-line placeholder, and RenderRoleFile refuses to render without either.

Docs in the same commit: `internal/orchengine/doc.go`'s role-file paragraph (the one on `## Commands` and `command_index`) gains that the role file's last section, `## This repository`, carries the `ORCHESTRATOR.md` at `Paths.RepoRulesPath` read at every role-file render, an absent or whitespace-only file renders the placeholder line, any other read failure fails the render naming the file, and a stencil without the `repository_rules` marker fails naming the stencil.
`internal/orchcli/doc.go`'s `# The role file and the note` section gains one sentence: the role file's `## This repository` section holds the prime worktree's `ORCHESTRATOR.md`, the repository's orch-only rules, re-read at every delivery.

Tests, all as rows or assertions of existing tests but for the one new `orchcli` test:
- `testPaths` sets `RepoRulesPath` to `ORCHESTRATOR.md` in the parent of its `orch` directory, a directory that exists and holds no such file.
- `TestRenderFiles_WriteWithoutLeadingComment`: the role row passes a rules path that is absent and wants `noRepositoryRules` after `## This repository`.
  New role rows: a rules file holding `Only the orch deploys.\n` wants that text in the section after `## This repository`, and the same row then rewrites the file to `Only the operator deploys.\n`, renders again to the same path and wants the new text and not the old; a whitespace-only file (`"  \n\t\n"`) wants `noRepositoryRules`.
  Every role row also wants the rendered file, with trailing whitespace trimmed, to end with the rules text it expects, which pins the section as the file's last.
  The case struct gains the rules content to seed (none for absent) and the content of the second render; the render closure takes the rules path, and the note-template row wraps `RenderNoteTemplateFile` in a closure that ignores it.
- `TestRenderRoleFile_RefusesStencilWithoutIndexMarker` is renamed `TestRenderRoleFile_RefusalsWriteNoFile` and becomes a table: the existing row (no marker at all) wants the stencil and `command_index` named; a new row whose override is `# Hub orchestrator\n\n{{.command_index}}\n` wants the stencil and `repository_rules` named; a new row keeps the shipped stencil, creates a directory at the rules path and wants the rules path named.
  Every row asserts no role file is written.
- `TestResumeContext_FailureReturnsErrorAndWritesNoMark` gains the row "unreadable repository rules": idle state saved and a directory created at `p.RepoRulesPath`; the case gains an optional substring the error must carry, here `p.RepoRulesPath`.
  The existing assertions (an error and no mark) then cover `resume-context` failing on an unreadable file.
- `TestWatcher_FailingRenderAbortsWithoutClear`: the `role` row's `wantReason` becomes `"role file failed to render: orch: " + roleStencilName`, since its `{{.nope}}` override fails the marker check before any fill.
  A new row, "repository rules", creates a directory at `e.paths.RepoRulesPath` before the tick and wants the reason to start with `role file failed to render: ` and contain `e.paths.RepoRulesPath`; the case gains a flag for the directory and the path check.
- `newTestCLI` sets `RepoRulesPath` to `ORCHESTRATOR.md` in the parent of its `orch` directory.
- `TestStart_Refusals` gains the row "unreadable repository rules": with no strands and no args, its seed creates a directory at `h.cli.paths.RepoRulesPath`, and the refusal must contain that path; the case gains a flag that adds the path to the wanted substrings.
  The row reaches the render in the relaunch branch, since no strand means `StartRelaunch`, and the existing assertions check no session started, no watcher spawned and the state file unchanged.
- `TestPrimePaths_RootsRepositoryRulesAtTheWorktree` builds a location with `locationkit.Location(hub, "main", "sub/dir")` and asserts `PrimePaths(location).RepoRulesPath` equals `filepath.Join(hub, "main", "ORCHESTRATOR.md")`, through the exported accessor other modules call.
  No existing test can take the assertion, since the one that calls `PrimePaths` anchors at `.` and cannot tell the anchor from the worktree root.
  `cmd/lyx/testdata/test-budget.yaml`'s `internal/orchcli` row rises by one for it.
- `TestStencils_TopLevelMarkers`: the `orch-template-role` row wants `command_index` and `repository_rules`, a forced edit of its exact-set comparison.

**ImpactSummary:** One re-signed engine function with three production callers (watcher, resume-context, `orchcli` start) and two tests, all on this card; one new told path set only in `orchcli`; the role stencil gains one marker, so a hand-edited role override without it refuses to render.
