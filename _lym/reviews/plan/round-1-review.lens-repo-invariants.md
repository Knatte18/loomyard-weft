```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked test seams, test-file tiers, doc text the change falsifies, and card order.

**Per-card invariant list**

- **Card 1** (board-view-projection)
  - Seam, held: `Outputs.Labels` is pinned by `TestOutputs`, which the card edits. No `//lym:seam` and no reflective field-set test covers `boardengine#Outputs`.
  - Tier, held: no new test file.
  - `boardengine/doc.go` `# README`, held: the card extends it, and nothing in it is exclusive.
  - `render.go` file header, **broken**: lines 1–2 list Render's outputs as README.md, the project files and the design files. After this card Render also returns `board-data.js`, and no card names the header. See finding 2.
  - Order, held: nothing pins the whole rendered file set. `render_test.go:861` checks project files only, and `render_test.go:1187` loops over README and project-root only.
- **Card 2** (entry-revision-check)
  - Seam, held: `Board` gains `expected`, and no seam or reflective test covers `boardengine#Board`. `Board` holds no lock value, so the shallow copy passes vet.
  - Doc, held: the card extends `doc.go`'s `# Concurrency and sync`.
  - Order, held: it builds on card 1's `migrateLoaded`.
- **Card 3** (viewer-page-asset)
  - Doc, held: `doc.go` `# Replay` (line 800) is listed as an Edit.
  - Doc, **broken**: the same `render.go` header omits `board.html`. Folded into finding 2.
  - Order, held: it edits card 1's `renderFiles` and card 1's manifest row.
- **Card 4** (board-server-routes)
  - Tier, held: `testmain_test.go` is a support file with no tier. `server_test.go` is tier0 and spawns nothing.
  - Doc, held: the new package's `doc.go` is created here.
- **Card 5** (board-server-lifecycle)
  - Tier, held: `run_test.go` is tier0 with no exec, hubforge or lymbin, and its clock is injected. `owneronly_windows_test.go` is tier0.
  - Doc, held: `lymdirs/doc.go` says each constant's own doc comment places its directory (lines 8, 11), so it needs no change. The `PATTERN-daemon-wakeups` background and `unboundedWaitFiles` are both listed.
  - Order, held: `boardserve/doc.go` exists from card 4.
- **Card 6** (board-serve-open-verbs)
  - Tier, held: `open_test.go` and `serveopen_test.go` are tier0. `runCLI` runs in process through `clihelp.Execute` (`cli_unit_test.go:19-24`). `serve_linux_integration_test.go` is tier1 with lymbin and hubforge.
  - `runproc/doc.go`, **broken**: the card lists it under Extends, but line 9 closes the list of HubOwner roles. See finding 1.
  - Doc, held: `boardcli/doc.go` gains a paragraph.
  - Seam, held: `runproc`'s leaf import set is unchanged, since only a constant is added.
- **Card 7** (vscode-tasks-and-keys)
  - Order, held: every caller of the deleted `assertAttachTaskOnly` is listed. They are `checkTaskSlugOpensBareFolder`, `TestSpawnAnchoredScenario`, `checkTaskPairStaysClean` and `checkOverwritesTasksKeepsSettings`.
  - Doc, held: the headers of `config.go` and `keybindings.go` and the overview's VS Code passage are all named.
- **Card 8** (workspace-content-seam)
  - Doc, held: the `workspace.go` header ("builds the content…, owns VS Code format knowledge only") stays true. `fabricengine/doc.go` `# The hub workspace file` is extended. The fabric-boundary background (line 124) is untouched by the change.
  - Order, held: `BuildWorkspace`'s callers are all on this card.
- **Card 9** (beam-start-workspace-task)
  - Tier, held: `ideengine` and `vscode` already have tier0 files (`look_test.go`, `workspace_test.go`) to hold the two new tier0 tests.
  - Seam, held: `TestPersistentPreRunE_InstallsSeams`' want list is edited.
  - Order, held: it builds on card 8's installer and card 7's label.

**Findings**

1. **LOW** · consistency · card 6, `internal/runproc/doc.go`
   - Summary: Card 6 lists `runproc/doc.go` under Extends, but line 9 is a closed list of the HubOwner roles: "Their roles are `RoleSync` and `RoleSlackRelay`." Adding `RoleBoardServer` under HubOwner means rewriting that sentence, which Extends does not allow. Appending a sentence would leave line 9 false.
   - Fix: move `internal/runproc/doc.go` from Extends to Edit on card 6.
   - Evidence: `internal/runproc/doc.go:8-9`.
2. **LOW** · consistency · card 1 (and card 3), `internal/boardengine/render.go` file header
   - Summary: The header says Render's outputs are "README.md, a project-&lt;slug&gt;.md for each project and the root, and a design-*.md for each entry with a body". Cards 1 and 3 make Render also return `board-data.js` and `board.html`, and no card names the header.
   - Fix: card 1's Intent says the header gains `board-data.js`, and card 3's says it gains `board.html`. The fix has to be Intent text only, because a `render.go` file target beside the card's `render.go` members is refused by `redundant-file-target`.
   - Evidence: `internal/boardengine/render.go:1-2`.

**Tool calls**

1. Grepped `//lym:seam` markers, reflective tests, and boardcli test files and TestMains. The pre-read did not cover the seam and tier conventions.
2. Read the `PATTERN-test-speed` tier0 file rules and the `boardengine/doc.go` headings. I needed to know how a file's tier is read and where the cards' doc edits land.
3. Read the remaining `boardengine/doc.go` headings, the vscode and fabric docs, the fabric-boundary background and `runproc/doc.go` roles. These are doc texts the cards may falsify and were not pre-read.
4. Read the `runCLI` definition, the `workspace.go` header, the ideengine and vscode tier0 files, and the `runproc/doc.go` head. I needed the tier0 legality of `TestServeOpenRefusals` and the exact doc wording.
5. Grepped tests that pin the exact rendered file set. This settles whether card 1 passes its own gate.
