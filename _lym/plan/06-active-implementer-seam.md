# Card 6 — active-implementer-seam

**Create:**
- `internal/weaverengine#Implementer`
- `internal/weaverengine#ImplementerWebster`
- `internal/weaverengine#ImplementerWeaver`
- `internal/weaverengine#BatchStrandRole`
- `internal/weaverengine#ErrImplementerConflict`
- `internal/weaverengine#ErrActiveRefused`
- `internal/weaverengine#StateVersion`
- `internal/weaverengine#ActiveDeps`
- `internal/weaverengine#Active`
- `internal/weaverengine#ActiveOrSetting`
- `internal/weaverengine#RunDirs`
- `internal/weaverengine#restorableVersion`
- `internal/weaverengine_test#TestActive`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/websterengine#LoadState`
- `internal/websterengine#Dir`
- `internal/websterengine#ScratchDir`

**Intent:** The one seam every loom reader of implementer state goes through, and the D1 guard's verdict (D1, D13).

- `StateVersion` is one committed version of Weaver's state file: the weft commit it was read from and its bytes.
- `ActiveDeps` carries `AnchorRoot`; `Setting`, which reads `tapestry.implementer`; and the three reads the Weaver repair advice takes:
  `StateHistory func() ([]StateVersion, error)` (the state file's versions on the records branch's first-parent line, newest first; loom fills it from `fabricengine.AnchoredFileFirstParentHistory` over `StateRel()`, whose versions carry their commit), `TaskHead` (the task branch's HEAD) and `LiveStrands` (the names `lym reed status` lists live).
  A nil read makes the advice name `lym weaver reset` alone.
- `Active` loads Webster's state (`websterengine.LoadState` over its `Dir` and `ScratchDir`) and Weaver's (`LoadState` over `Dir` and `ScratchDir`).
  A missing file counts as no begun batches.
  - A state file that fails to load refuses with an error wrapping `ErrActiveRefused` and naming the file:
    for Webster the way forward is repairing `_lym/webster/state.json`, or restoring its committed version with git;
    for Weaver it is restoring the version `restorableVersion` picks, naming its commit, when it picks one, and `lym weaver reset` otherwise.
    A corrupt file never counts as missing.
  - Both holding begun batches refuses with an error wrapping `ErrImplementerConflict` and `ErrActiveRefused` that names the resets alone, since a setting cannot clear it: `lym webster reset --to start` for Webster's run, or `lym weaver reset` for Weaver's.
  - One holding begun batches is the active implementer.
  - Neither: the setting's value, `webster` or `weaver`; a nil `Setting` answers `webster`, the setting's default, since with neither begun there is no state to read.
    A setting error or another value refuses, wrapping `ErrActiveRefused` and naming `lym config tapestry --set implementer=<webster|weaver>`.
  It returns the active implementer's `runplan.State`, nil when its file is missing.
- `restorableVersion` picks the newest version that parses, only when its `LastEnd` equals `TaskHead`'s answer and `LiveStrands` lists no strand whose role matches `BatchStrandRole` or `gatefixer.RoleVerifyFixer`; on any read error it picks none.
  The refusal says why a restored state holds: the next begin captures HEAD as the start, so no attempt may have committed since and none may be in flight.
- `ActiveOrSetting` is the advisory readers' form (the decision-notice guard, merge-in's warning, the arbiter's note, batten's fixer timeout): on any refusal of `Active` it falls back to the setting's implementer, loads that implementer's state when it loads, and returns the refusal's text as the reason the caller logs.
- `RunDirs` returns an implementer's run and scratch directories, so a reader names the active one's without a switch of its own.

`TestActive` is tier0, a table over temp anchors with planted state files:
neither file falls back to each setting value, and to `webster` with a nil setting; a setting error and an unknown value refuse naming `lym config tapestry`;
a begun Webster or Weaver state wins over the other setting;
both begun refuses naming both resets and no setting step; a corrupt Webster file refuses naming it and its repair, also with Weaver begun;
a corrupt Weaver file names the restore and its commit when the newest parsing version's `LastEnd` equals HEAD and no strand is live, and `lym weaver reset` when HEAD moved, when a `weaver` or `verify-fixer` strand is live, when no version parses and when the history read fails;
`ActiveOrSetting` falls back to the setting on each refusal and returns its reason.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
