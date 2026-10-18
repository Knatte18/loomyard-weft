# Card 1 — rule-files-render

**Edit:**
- `internal/orchengine#Paths`
- `internal/orchengine#RenderRoleFile` -> `func RenderRoleFile(stencilsDir, path, index, hubRulesPath, ownRulesPath string) error`
- `internal/orchengine#Watcher.renderRoleFile`
- `internal/orchengine#Watcher.isRequestedNote`
- `internal/orchengine#ResumeContext`
- `internal/orchcli#orchPaths`
- `internal/orchcli#orchCLI.startCmd`
- `//internal/orchengine/doc.go`
- `//internal/orchcli/doc.go`
- `//contracts/stencils/orch/orch-template-role.md`
- `internal/orchengine#TestRenderFiles_WriteWithoutLeadingComment`
- `internal/orchengine#renderRole` -> `func renderRole(dir, path, hubRulesPath, ownRulesPath string) error`
- `internal/orchengine#TestRenderRoleFile_RefusalsWriteNoFile`
- `internal/orchengine#TestResumeContext_FailureReturnsErrorAndWritesNoMark`
- `internal/orchengine#testPaths`
- `internal/orchengine#TestWatcher_FailingRenderAbortsWithoutClear`
- `internal/orchengine#TestWatcher_NoteReplyGate`
- `internal/orchcli#newTestCLI`
- `internal/orchcli#TestStart_Refusals`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Delete:**
- `internal/orchengine#repositoryRulesMarker`
- `internal/orchengine#noRepositoryRules`
- `internal/orchengine#readRepositoryRules`
- `internal/orchcli#TestPrimePaths_RootsRepositoryRulesAtTheWorktree`

**Create:**
- `plan:internal/orchengine#hubRulesMarker` -> `const hubRulesMarker = "hub_rules"`
- `plan:internal/orchengine#orchRulesMarker` -> `const orchRulesMarker = "orch_rules"`
- `plan:internal/orchengine#noHubRules` -> `const noHubRules = "This hub has no hub-wide orch rules; they go in %s."`
- `plan:internal/orchengine#noOwnRules` -> `const noOwnRules = "This orch has no rules of its own; they go in %s."`
- `plan:internal/orchengine#readRules` -> `func readRules(path, placeholder string) (string, error)`
- `plan:internal/orchcli#rulesRel` -> `func rulesRel() string`
- `plan:internal/orchcli#TestPrimePaths_JoinsRuleFilesAndPrimeRoot` -> `func TestPrimePaths_JoinsRuleFilesAndPrimeRoot(t *testing.T)`

**Uses:**
- `internal/fabricengine#BoardDir`
- `internal/lyxdirs#LyxDirName`
- `internal/orchengine#watchEnv.reply`
- `internal/orchengine#watchEnv.writeHandoff`
- `internal/orchengine#watchEnv.injectHandoff`
- `internal/orchengine#newWatchEnv`
- `internal/orchcli#newStartHarness`

**Intent:** Replace the one repository rules file with D1's two told rule files and D2's two role-file sections, in one card because removing `RepoRulesPath` and the `repository_rules` marker breaks every caller and the shipped stencil at once.

`orchengine.Paths` loses `RepoRulesPath` and gains three told fields: `HubRulesPath` (the hub-wide rules file on Bolt), `OwnRulesPath` (this orch thread's own rules file in its worktree's weft) and `PrimeRoot` (the prime's worktree root), each with a one-line field comment.
`rulesRel` returns `filepath.Join(lyxdirs.LyxDirName, orchDirName, "rules.md")`, the one join of the rules file's relative path, which card 2's verb also commits by.
`orchPaths` sets `HubRulesPath` to `filepath.Join(fabricengine.BoardDir(location.HubPath), rulesRel())`, `OwnRulesPath` to `filepath.Join(location.AnchorPath(), rulesRel())` and `PrimeRoot` to `location.WorktreePath()`.
`paths.go`'s header comment drops the `ORCHESTRATOR.md` exception and says the two rules files are durable `_lyx/orch/` files, the hub-wide one under the board dir and the own one under the anchor, and that `PrimeRoot` is the worktree root a relative note path resolves against.

`RenderRoleFile` takes `hubRulesPath` and `ownRulesPath` in place of `rulesPath`.
It checks the markers in the order `command_index`, `hub_rules`, `orch_rules`, `decision_note_title`, each refused as today with the stencil and the missing marker named.
It reads the hub-wide file with `readRules(hubRulesPath, noHubRules)` before the own file with `readRules(ownRulesPath, noOwnRules)` and fills `hub_rules` and `orch_rules` with them.
`readRules` is `readRepositoryRules` with the placeholder passed in: a missing file, or one holding only whitespace, yields `fmt.Sprintf(placeholder, path)`, so the placeholder names the file's absolute path;
any other read error returns `orch: read rules %s: %w; fix the file or remove it`, naming the file, and no role file is written; the text of a readable file is returned verbatim.
`RenderRoleFile`'s doc comment names the two files and the four markers.
`Watcher.renderRoleFile` and `ResumeContext` pass `paths.HubRulesPath, paths.OwnRulesPath`, and so does `startCmd`'s render call; `Watcher.renderRoleFile`'s doc comment names "a rules file" in place of `ORCHESTRATOR.md`.
`Watcher.isRequestedNote` joins a relative path onto `w.paths.PrimeRoot` in place of `filepath.Dir(w.paths.RepoRulesPath)`.

The role stencil's closing `## This repository` section and its `{{.repository_rules}}` marker are replaced by a preamble and two closing sections, in this order.
The preamble is its own short section, `## Rules files`, directly above the two: the two rules files below add to this role and override none of it; a rule in them that contradicts this role is raised to the operator, and this role is followed meanwhile; a rules file's own headings start at level three (`###`), since it is inlined under a level-two section.
Then `## Hub-wide rules`, saying it holds the hub's rules for every orch, followed by `{{.hub_rules}}`, and `## This orch's rules`, saying it holds this orch thread's own rules, followed by `{{.orch_rules}}`, which stays the stencil's last line.
The stencil's leading comment names the four markers, `hub_rules` filled with the hub-wide rules file and `orch_rules` with this orch's own rules file, each or a one-line placeholder naming its path.
`TestStencils_TopLevelMarkers`'s `orch-template-role` row lists `command_index`, `decision_note_title`, `hub_rules` and `orch_rules`.
`internal/orchengine/doc.go`'s role-file paragraph says the role file's last two sections, `## Hub-wide rules` and `## This orch's rules`, carry the files at `Paths.HubRulesPath` and `Paths.OwnRulesPath`, read at every role-file render, with each absent or whitespace-only file rendering a placeholder line naming its path, any other read failure failing the render naming the file, and a stencil without `hub_rules` or `orch_rules` failing naming the stencil.
`internal/orchcli/doc.go`'s `## This repository` sentence becomes one naming the role file's `## Hub-wide rules` and `## This orch's rules` sections, holding the hub-wide rules file on Bolt and this orch thread's own rules file under its worktree's `_lyx/orch/`, re-read at every delivery, so this card's commit ships no doc describing the removed section.

Tests, all in the existing functions:
- `testPaths` sets `PrimeRoot` to `filepath.Dir(dir)`, `HubRulesPath` to `filepath.Join(filepath.Dir(dir), "board", "rules", "hub.md")` and `OwnRulesPath` to `filepath.Join(filepath.Dir(dir), "own", "rules.md")`, so neither rules file's directory is the prime root; `newTestCLI` sets the same three fields in the same shape.
- `TestRenderFiles_WriteWithoutLeadingComment`'s `render` takes both paths, `renderRole` passes both, and the cases seed `hubRules` and `ownRules` separately and replace `wantRules` with an ordered `wantOrder func(hubPath, ownPath string) []string` that each must appear after the previous one.
  The two rules files are seeded at `filepath.Join(root, "board", "hub-rules.md")` and `filepath.Join(root, "own", "rules.md")`, and `TestRenderRoleFile_RefusalsWriteNoFile` joins its two rules paths the same way, so no test keeps the name `ORCHESTRATOR.md` that card 3's Verify greps for.
  Rows: both present (`## Hub-wide rules`, the hub text, `## This orch's rules`, the own text); only the hub-wide file present (hub text then `fmt.Sprintf(noOwnRules, ownPath)`); only the own file present (`fmt.Sprintf(noHubRules, hubPath)` then the own text); the hub-wide file whitespace-only and the own file present (hub placeholder then the own text); and the existing rewritten-rules row on the own file.
  A swapped argument or section order fails the order assertion, and a placeholder that does not name its path fails the placeholder rows.
- `TestRenderRoleFile_RefusalsWriteNoFile`: the `no rules marker` row becomes `no hub rules marker` (override carrying only `command_index`, naming `hub_rules`), a new `no orch rules marker` row overrides with `command_index` and `hub_rules` and wants `orch_rules` named, the decision-note row's override carries all three other markers, and `rulesDir` becomes a choice of which path gets a directory, with rows `unreadable hub rules` and `unreadable own rules` each wanting that path named and no role file written.
- `TestResumeContext_FailureReturnsErrorAndWritesNoMark`, `TestWatcher_FailingRenderAbortsWithoutClear` and `TestStart_Refusals` each replace their one unreadable-rules row with two, a directory at `HubRulesPath` and one at `OwnRulesPath`, each wanting that path in the error, abort reason or refusal; this pins that `ResumeContext`, the watcher and `start` each render both files.
- `TestWatcher_NoteReplyGate` gains a subtest beside `a reply that passes the gate moves on to clearing`: the note is written with `writeHandoff`, the reply names it relative to `e.paths.PrimeRoot` (`filepath.Rel`), and the phase moves to clearing with one clear sent; resolving against any other directory leaves the gate closed.
- `TestPrimePaths_JoinsRuleFilesAndPrimeRoot` replaces `TestPrimePaths_RootsRepositoryRulesAtTheWorktree`: over `locationkit.Location(hub, "main", "sub/dir")`, whose anchor differs from its worktree root, it wants `HubRulesPath` = `filepath.Join(fabricengine.BoardDir(hub), lyxdirs.LyxDirName, "orch", "rules.md")`, `OwnRulesPath` = `filepath.Join(hub, "main", "sub", "dir", lyxdirs.LyxDirName, "orch", "rules.md")` and `PrimeRoot` = `filepath.Join(hub, "main")`, spelled out rather than through `rulesRel`.

**ImpactSummary:** Re-signs `RenderRoleFile`, whose three callers (`Watcher.renderRoleFile`, `ResumeContext`, `startCmd`) and test helper change here; `Paths` loses a field read only in orchengine, orchcli and their tests; the shipped role stencil and its marker row change together; other modules' `PrimePaths` callers read no rules field.
