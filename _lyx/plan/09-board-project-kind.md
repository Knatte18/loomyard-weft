# Card 9 — board-project-kind

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#normalizeTask`
- `internal/boardengine#handleLetter`
- `internal/boardengine#validateTypeLetters`
- `internal/boardengine#ComputeLayers`
- `internal/boardengine#RenderOrder`
- `internal/boardengine#Store.ListTasksBrief`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#scopeChanged`
- `internal/boardengine#replayFields`
- `internal/boardengine#writeOwners`
- `internal/boardengine#kindName`
- `internal/boardengine#validateLabels`
- `internal/boardengine#TestStoreValidatesLabelsWithVocabulary`
- `internal/boardengine_test#TestKindDependencyRules`
- `internal/boardengine_test#TestHandles_AssignedLowestFreeAndStable`
- `internal/boardengine_test#TestLoadConfig_LabelsRefused`
- `internal/boardengine_test#TestComputeLayers`
- `internal/boardengine_test#TestListTasksBriefLayerAndProposal`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine#Store.Promote`
- `internal/boardengine_test#TestPromote`
- `internal/boardengine_test#TestPrimeOwnerReadAndRendered`
- `internal/boardengine/doc.go`

**Create:**
- `plan:internal/boardengine_test#newProjectBoard` -> `func newProjectBoard(t *testing.T, configure func(cfg *boardengine.Config)) *boardengine.Board`
- `plan:internal/boardengine#KindProject` -> `const KindProject = "project"`
- `plan:internal/boardengine#projectHandleLetter` -> `const projectHandleLetter = "P"`
- `plan:internal/boardengine#projectLayer` -> `const projectLayer = "Project"`
- `plan:internal/boardengine#checkProjectRules` -> `func checkProjectRules(loaded, after []Task, pairExists func(slug string) bool) error`
- `plan:internal/boardengine_test#TestProjectRules` -> `func TestProjectRules(t *testing.T)`

**Intent:**
D1: `kind: project` joins task and note.
`Task` gains `Project string` (`project`, omitempty), `Order []string` (`order`, omitempty) and `Beam bool` (`beam`, omitempty); an older binary keeps them as unknown keys (PATTERN-tolerant-records), and the record's open risk on the kind value stands.
No upsert key is added here: cards 10 and 11 admit `project`, `order` and `beam` with their rules.

`normalizeTask` accepts `KindProject`, and refuses a project carrying `depends_on`, `recipe`, `profile`, `run_settings` or `priority`, naming the entry and the field to drop; it refuses `order` or `beam` on a task or a note, naming the field, since only a project carries them; a project keeps its `owner_machine` as a task does.
A project carries one or more type labels, as a task does: `validateLabels` refuses a project with none, worded as its task refusal with `project`, and `TestStoreValidatesLabelsWithVocabulary` gains that row.
`handleLetter` gives a project `projectHandleLetter`; `validateTypeLetters` reserves `P` as it reserves `T`, refusing a type label whose letter is `P` with the text naming the label and renaming it in `board.yaml`.

`checkProjectRules` runs in `boardCriticalSection` beside `checkRunLocks`, over the loaded and the after entries (a rename judged under the new slug, as the run lock is), with the store's pair-exists seam (card 10 fills its use, nil here).
This card adds its first rule: an entry whose `depends_on` the write sets or changes may not name a project, refused naming the entry, the project and the project's task members (entries whose `project` is it) as what to depend on, or `lyx board set-deps` to drop it; an unchanged `depends_on` is never judged, so a done task naming a promoted project takes a later write.
The input that reaches it: an upsert or `set-deps` giving a task `depends_on: [<project>]`; `TestProjectRules` drives it through a `Board` from `newProjectBoard`, which writes a `board.yaml` with type labels and the README output under `t.TempDir()`, loads it with `LoadConfig`, and calls `configure` on the config when it is non-nil before `New`, so later cards set their seams (`PairExists`, `ValidateSlug`, `ParentOf`) through it; the shape `newRenameBoard` builds.
`Store.Promote` refuses a project entry, naming it and that a project is made a beam with `lyx beam create <slug>`, never a task, since only a task is claimable bar a beam's landing (D1); a note and a task keep today's behaviour.

`ComputeLayers` puts an open project in `projectLayer`, never Ready or a letter, a run-held project in Running, and a done one in the done layer; a project counts as no dependency's depth.
`RenderOrder` orders open tasks, then open projects by handle, then open notes, then done entries (a done project between done tasks and done notes); `kindName` names a project `Project`.
`ListTasksBrief` resolves a project's effective owner as it does a task's.
`scopeChanged` compares `Project`, `Order` and `Beam`, so the run lock refuses a change to them on a run-held entry (D2's closing rule).
`replayFields` gains `project`, `order` and `beam`.
`writeOwners` records an owner on a project as on a task, and writes a claim only on a task or on a project whose `Beam` is true (D1's one claimable project), leaving any other project's status alone.

Test rows: `TestKindDependencyRules` refuses each forbidden project field by name and `order`/`beam` on a task and a note; `TestHandles_AssignedLowestFreeAndStable` gives two projects `P1`, `P2`; `TestLoadConfig_LabelsRefused` refuses a type label `plan` naming it and `board.yaml`; `TestComputeLayers` lists an open project with no members under `Project`, a run-held one under `Running` and a task beside them unchanged; `TestListTasksBriefLayerAndProposal` carries the project's layer; `TestPrimeOwnerReadAndRendered`, over a board whose `board.json` records a `prime_owner`, lists an unowned project with the prime owner as its `owner_machine`; `TestFabricReplay` replays a changed `project` and `order`; `TestSetOwners` records an owner on a project, and a claim asked on a project whose `beam` is false writes no status on it while the owner is still recorded;
`TestPromote` refuses a project naming `lyx beam create`;
`TestProjectRules` refuses a task's `depends_on` set to a project naming its members, and lets a done task whose unchanged `depends_on` names a project take a body edit.
The package doc's Entries, Handles, Owners and README sections are rewritten where they say every entry is a task or a note and only a task is claimable, to name the project kind.

**ImpactSummary:** The board's kind set, handle letters, layers and run-lock scope grow by the project kind; every caller reads the same Task type and no signature changes.
