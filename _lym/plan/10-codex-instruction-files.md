# Card 10 — codex-instruction-files

**Create:**
- `plan:internal/shuttleengine/codexengine#defaultProjectDocMaxBytes` -> `const defaultProjectDocMaxBytes = 32 * 1024`
- `plan:internal/shuttleengine/codexengine#projectDocMaxBytes` -> `func projectDocMaxBytes(found checkout, workDir string) (int, error)`
- `plan:internal/shuttleengine/codexengine#Codex.unreadClaudeFiles` -> `func (c *Codex) unreadClaudeFiles(found checkout, workDir string) []string`
- `plan:internal/shuttleengine/codexengine#instructionPointers` -> `func instructionPointers(paths []string) string`
- `plan:internal/shuttleengine/codexengine#TestPrepare_InstructionFiles` -> `func TestPrepare_InstructionFiles(t *testing.T)`

**Edit:**
- `internal/shuttleengine/codexengine#launchLine`
- `internal/shuttleengine/codexengine#codexCommand`
- `internal/shuttleengine/codexengine#Codex.Prepare`
- `internal/shuttleengine/codexengine#TestBuildLaunchCmd`
- `internal/shuttleengine/codexengine#TestPrepare_Launch`
- `internal/shuttleengine/codexengine#TestPrepare_SandboxMode`
- `internal/shuttleengine/codexengine#TestPrepare_ExpandsHomeInWritableRoots`
- `internal/shuttleengine/codexengine#TestPrepare_InstallsGuardsFromItsInputs`
- `plan:internal/shuttleengine/codexengine#TestPrepare_SubagentTerms`

**Extends:**
- `internal/shuttleengine/codexengine/doc.go`

**Uses:**
- `internal/claudeimports#Expand`
- `internal/heddleengine#Marker`
- `plan:internal/shuttleengine/codexengine#Seams`
- `internal/shuttleengine/codexengine#instructionFallbackFile`
- `internal/shuttleengine/codexengine#repoFixture.addWorktree`
- `internal/shuttleengine/codexengine#repoFixture.writeCodexLayer`
- `internal/shuttleengine/codexengine#writeWritableRoots`
- `internal/shuttleengine/codexengine#warnLog`
- `internal/shuttleengine/codexengine#newEngine`

**Intent:**
Codex reads every instruction file Claude reads, whole (D9).

The session's working directory is the engine's anchor, where the pane starts and the sandbox probe runs.
`projectDocMaxBytes` sums, over each directory from the checkout root down to the working directory, the size of the one file Codex reads there:
the first of `AGENTS.override.md`, `AGENTS.md` and `instructionFallbackFile` that exists.
It returns that sum, or `defaultProjectDocMaxBytes`, Codex's own default, when the sum is smaller, so the cap is at least the size of the files Codex reads.
A stat error other than a missing file refuses the launch with the `codexengine:` prefix, naming the file, before `prompt.md` is written; it is a propagated raw I/O error.
`launchLine` gains `projectDocMaxBytes`, and `codexCommand` adds `-c project_doc_max_bytes=<n>` beside the fallback-file setting, before the hook settings, when it is above zero;
`Prepare` sets it, and `OpenLaunch` leaves it zero, so an opened line is unchanged.

`unreadClaudeFiles` walks the whole checkout from its root without following links, skipping `.git` directories.
It returns, sorted, the absolute path of each `CLAUDE.md` in the worktree Codex does not read itself:
one in a directory below the working directory;
one anywhere in the worktree beside an `AGENTS.md`, except at the working directory when the `AGENTS.override.md` there opens with `heddleengine.Marker`, compared as a prefix of the file's text as `Render` does, since heddle's render carries both files in it (card 4);
an override without the marker is the repository's own, which `Render` leaves untouched, so the CLAUDE.md beside it stays named;
and one on the path from the root to the working directory that Codex reads itself but whose `@` imports `claudeimports.Expand` resolves, since Codex reads it unexpanded.
A directory the walk cannot read is skipped and named in a warning through `c.warn`, so one unreadable directory never blocks a launch; the pointers are advisory.
`instructionPointers` returns "" for none, else a passage naming each file with the rule to read it in full, and the files its `@` lines import, before working in its directory, as Claude loads a nested CLAUDE.md there.
`Prepare` adds it as the developer-instructions item named `the instruction-file pointers`, after the user setup's instruction files and before the skill listing, D5's order;
its way forward is to move the rules into fewer CLAUDE.md files.

Tests:
- `TestPrepare_InstructionFiles` (tier0, parallel table over the fixture worktree with the anchor in a subdirectory `src/app`;
  each row calls `repo.writeCodexLayer(t, subAnchor, "")` and `writeWritableRoots(t, subAnchor, "")`, since the engine reads codex.yaml at its anchor):
  root and anchor files totalling over 32 KiB give a cap equal to their sum, and a small set gives 32768;
  an `AGENTS.override.md` at the anchor of a size other than the anchor's `AGENTS.md`, with the total over 32 KiB, counts instead of it;
  a CLAUDE.md below the anchor is named;
  a CLAUDE.md beside an `AGENTS.md` at the root, and one beside an `AGENTS.md` in a sibling directory `src/lib`, are named;
  the anchor's CLAUDE.md beside an `AGENTS.override.md` opening with `heddleengine.Marker` is not named, and a CLAUDE.md alone at the root with no import is not named;
  an unmarked `AGENTS.override.md` at the anchor beside `AGENTS.md` and CLAUDE.md leaves the CLAUDE.md named;
  a CLAUDE.md alone at the root that imports `@docs/rules.md` is named;
  a directory made unreadable with mode 0 is skipped and named in a warning (the row skips itself when the test runs as root, which reads every directory);
  `PointersBetweenFilesAndListing` builds the engine with a fake `Seams.UserSetup` returning one instruction file and a skill listing, and asserts the developer instructions hold the file, then the pointers, then the listing.
- `TestBuildLaunchCmd`'s `EveryInput` row sets `projectDocMaxBytes` and its want carries the setting; the opened-line assertion carries none.
- The four full-line Prepare tests add `projectDocMaxBytes: defaultProjectDocMaxBytes` to their want line, since their fixtures hold no instruction file.
  `TestPrepare_SubagentTerms`'s want lines add it too, since its fixture holds no instruction file either.

`codexengine/doc.go` gains a paragraph on the cap and the pointers.

**ImpactSummary:** every Prepare launch line gains the doc cap, and launches in worktrees with unread CLAUDE.md files gain one developer-instructions item; OpenLaunch is unchanged.
