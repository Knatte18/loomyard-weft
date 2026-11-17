# Card 4 — codex-file-carries-both

**Create:**
- `internal/heddleengine#importRecordFile`
- `internal/heddleengine#importRecord`
- `internal/heddleengine#recordImports`
- `internal/heddleengine#recordedImports`

**Edit:**
- `internal/heddleengine#RenderRequest`
- `internal/heddleengine#requestOf`
- `internal/heddleengine#fileText` -> `func fileText(target Target, anchor string, entries []LayeredEntry) (text string, imports []string, held bool, err error)`
- `internal/heddleengine#Renderer.Render`
- `internal/heddleengine#Renderer.Show`
- `internal/heddleengine#Seam.TookIn`
- `internal/heddleengine#TestRender`
- `internal/heddleengine#TestSeam`
- `internal/heddleengine#renderFixture.request`
- `internal/heddleengine_test#TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile`
- `internal/heddleengine/doc.go`
- `internal/heddlecli#heddleCLI.renderInto`

**Uses:**
- `internal/heddlecli/doc.go`
- `internal/claudeimports#Expand`
- `internal/heddleengine#newRenderFixture`
- `internal/heddleengine#renderFixture.writeAnchor`
- `internal/heddleengine#renderFixture.readAnchor`
- `internal/heddleengine#newSeamFixture`
- `internal/heddleengine#seamFixture.tookInAfterClearing`

**Intent:**
`AGENTS.override.md` carries the repository's `AGENTS.md` and `CLAUDE.md` both, with the CLAUDE.md imports expanded,
and an edit to a file inside the worktree that the CLAUDE.md imports re-renders it (D9).

`fileText` for `codex-file` opens with the anchor's `AGENTS.md` text when it exists, then the anchor's `CLAUDE.md` expanded through `claudeimports.Expand` when it exists, each its own paragraph;
today it takes only the first of the two that exists.
It returns the expanded import files, absolute, as `imports`; every other target returns none.
`Render` and `Show` are its two callers and take the new result; `Show` ignores `imports`.

`RenderRequest` gains `Root`, the worktree root, which every builder sets:
`requestOf` copies `ev.Root`, `heddlecli`'s `renderInto` copies `event.Root`,
the `TestRender` fixture's `f.request()` sets the fixture's anchor, and `TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile` sets `event.Root`.
When `Render` renders `codex-file`, `recordImports` writes `importRecord{Paths}` to `<ScratchDir>/heddle/codex-file-imports.json`
through `internal/state.WriteJSON`, with a lock file beside it (`PATTERN-tolerant-records`).
`Paths` are the slash-separated, root-relative paths of the `imports` inside `Root`, sorted;
an import outside the worktree raises no worktree event, so it is left out, and an edit to it reaches Codex at the next render (the record's bound).
The record is rewritten at each such render, empty when the target holds no text or its CLAUDE.md imports nothing, so a dropped import stops matching.
A failed record write is a render warning naming the record file and `lym heddle render` as the way forward, and leaves the rendered files as written.

`Seam.TookIn` selects `codex-file` also when one of `ev.Paths` equals a path `recordedImports(ev.ScratchDir)` returns.
`recordedImports` reads the record with `state.ReadJSON`; an absent record gives none,
and an unreadable one gives none with a Warn log naming the file and `lym heddle render`, which rewrites it.

Tests:
- `TestRender`'s `CodexFileOpensWithRootAgents` table changes its `AgentsWinsOverClaudeMd` row, renamed `BothFiles`, to expect `AGENTS.md`'s text then `CLAUDE.md`'s;
  `RootAgents`, `ClaudeMdWhenNoAgents` and `NeitherFile` stay.
  It gains `ClaudeMdImport`: a `CLAUDE.md` importing `@docs/rules.md` gives the imported text in its place.
- `TestRender` gains a `CodexFileRecordsItsImports` subtest whose rows build the request from `f.request()`, whose `Root` is the fixture's anchor:
  `InsideTheWorktree`: the record names `docs/rules.md`;
  `OutsideTheWorktree`: an import of a file outside the anchor leaves it out of the record;
  `DroppedImport`: a second render after the import line is removed leaves the record empty;
  `RecordUnwritable`: a directory standing at the record's path makes the render report a warning naming `lym heddle render`, and `AGENTS.override.md` is still written.
- `TestSeam` gains a subtest, `TookInRendersTheCodexFileOnAChangedImport`, over `newSeamFixture` at anchors `.` and `src/app`.
  The anchor's `CLAUDE.md` imports `@docs/rules.md`; after `Made`, `tookInAfterClearing` over that import's root-relative path re-renders `AGENTS.override.md` and not `CLAUDE.local.md`.
  A row whose record file holds malformed JSON re-renders neither on that path.
- `TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile` sets `Root` to `event.Root` in its request.

`heddleengine/doc.go`'s codex-file sentence, "opens with the anchor's AGENTS.md, else its CLAUDE.md", is rewritten:
the target carries both files with CLAUDE.md's imports expanded, and the render records the imports a take-in re-renders on.

**ImpactSummary:** `fileText`'s two callers sit in render.go and change with it; `RenderRequest` gains one field its two production builders and two test builders set; `TookIn` gains one record read; the claude-file render is unchanged.
