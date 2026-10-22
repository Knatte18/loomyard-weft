# Card 4 — halt-note-drops-parent-status

**Delete:**
- `internal/loomcli#renderParentFabricStatus`
- `internal/loomcli#loomCLI.parentDirtyPathsFromFabric`
- `internal/fabricengine#IsWorktreeDirtyReason`

**Edit:**
- `internal/loomcli#haltNote`
- `internal/loomcli#writeHaltNote`
- `internal/loomcli#loomCLI.reflectHalt`
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/haltnote#Note`
- `internal/haltnote#Render`
- `internal/haltnote#Parse`
- `internal/haltnote#parentHeader`
- `internal/haltnote#TestRenderParse`
- `internal/loomcli#TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory`
- `internal/loomcli#TestLoomPostRun_HaltNotes`
- `internal/fabricengine#TestMergeVocabulary_ErrorsAreSideFree`
- `contracts/stencils/friction/friction-template-reflection.md`

**Intent:**
D3: the halt note's fenced reason now holds the guard's own path list (card 3), so the second, later read across both checkouts that showed the run record as dirt goes.

- `haltnote.Note` loses `ParentStatus`, and `Render` writes no `parent fabric status:` section.
  `Parse` still recognizes the `parent fabric status:` header and skips every line under it, so a note an older binary wrote parses with every other field intact (`PATTERN-tolerant-records`); the `parentHeader` constant stays for that, its comment saying so.
- `loomcli`'s `haltNote` loses `ParentFabricStatus`, `writeHaltNote` stops copying it, and `reflectHalt` no longer reads the parent: it writes the note as told.
  `renderParentFabricStatus`, `loomCLI.parentDirtyPathsFromFabric`, the `parentDirtyPaths` field of `loomCLI` and its assignment in `newLoomCLI` go; `halt.go` drops its `fabricengine` and any other import left unused.
- `fabricengine.IsWorktreeDirtyReason` goes with its one production caller, `reflectHalt`; `TestMergeVocabulary_ErrorsAreSideFree` loses its two assertions on it.
- The friction reflection stencil's sentence about a `parent fabric status:` section goes, since no new note carries one; a dirty-parent halt's reason names the parent's code checkout itself.

Tests:
- `TestRenderParse` drops its `parent status` row and gains a parse row: bytes holding a full note followed by a `parent fabric status:` section of two path lines parse to the `Note` with every other field equal and no error, and re-render with no section.
- `TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory` drops the `parentStatus` field and its `ParentPaths`, `ParentReadError` and `ParentClean` rows, and every remaining row asserts the note holds no `parent fabric status` text.
- `TestLoomPostRun_HaltNotes` drops its `DirtyReasonCarriesTheParentFabricStatusAndAnotherReasonDoesNot` subtest, which drove the removed seam, and gains no replacement: with the seam gone `reflectHalt` takes no branch on the reason, and the absence of the section is the `TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory` and `TestRenderParse` assertions above.

**ImpactSummary:** the halt note format and its one writer in `loomcli`, one fabricengine helper with that single caller, and a stencil sentence; no other package reads `Note.ParentStatus`, and old notes still parse.
