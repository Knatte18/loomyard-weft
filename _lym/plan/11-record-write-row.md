# Card 11 — record-write-row

**Create:**
- `internal/loomshed#NameRecordWrite`
- `internal/loomshed#RecordWriteDeps`
- `internal/loomshed#NewRecordWrite`
- `internal/loomshed#BoardRecord`
- `internal/loomshed#TestRecordWrite_Call`
- `internal/shedrecipe#recordWriteEntry`
- `internal/shedrecipe#TestRecordWriteEntry`
- `internal/loomcli#recordWriteDeps`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe/doc.go`
- `internal/testkit/envkit#FullEnv`
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed/doc.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/loomcli#TestRenderChecklist`
- `internal/battencli#intentionallyNil`
- `internal/loomrecipe#TestRecipeKeys`
- `internal/reportengine#darnPhases`
- `internal/reportengine/doc.go`
- `internal/loomrecipe#New`
- `contracts/recipes/darn-recipe.yaml`

**Uses:**
- `internal/driftengine#TestDriftBaseline`
- `internal/driftengine#TestDriftAwait`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/loomrecipe/doc.go`
- `internal/loomcli#commitDiscussion`
- `internal/discussionparser#Validate`

**Intent:** A Go row, `Record-Write` with engine `RecordWrite`, writes darn's decision record and support log from the board entry, with no agent (D4), and darn declares the loom contract.

`BoardRecord(body)` renders the record: `## Goal` holding the board entry's body verbatim in one fenced `markdown` block whose backtick fence is max(3, one longer than the longest backtick run in the body) backticks long, then `## Scope`, `## Decisions`, `## Constraints`, `## Auto-mode assumptions`, `## Open risks` and `## Acceptance criteria`, each present and empty. Under card 1's fence rule no body line, a `## Decisions` or `## Open risks` heading, an inner code block or an unclosed inner fence included, starts a record section, so `decision add`, the carry-over and the PATTERN parse act only on the record's own sections; an empty body writes an empty fenced block.
The support log carries its four sections in order: `## Interview` saying the record was written from the board entry with no interview, `## Rejected alternatives` empty, `## Review rounds` seeded with `_No rounds yet._`, and `## Question ledger` empty.

`RecordWriteDeps` carries `RecordPath`, `SupportLogPath` and `DiscussionDir` (absolute), `ReadBody func() (body string, found bool, err error)` and `Commit func() error`.
`NewRecordWrite`'s `Call`, after the entry context check:
1. removes every `.tmp-` file directly in `DiscussionDir` a crash left, so the discussion commit, which stages the whole directory, never takes one;
2. writes whichever of the two files is missing through `fsx.AtomicWriteBytes`, reading the body only when the record is missing, and never overwrites a file that exists, so a resume or a goto back to the row keeps every `Added after Discussion` entry;
3. calls `Commit`, which commits the pair when it holds uncommitted changes and is a no-op on a clean directory, so a resume after a crash between the writes or before the commit completes the pair;
4. returns Done with the record as its pointer.
A board that holds no entry for the run (`found` false) is Stuck, reason `the board holds no entry for this run; way forward: restore the entry with "lym board upsert", then "lym loom resume"`. A read, write or commit error is a returned error, and a cancelled context wins.

`recordWriteEntry` takes no config key (unknown keys refused like every entry), requires the three paths absolute and the two seams non-nil through `requireAbsRoot`/`requireSeam`, and builds the producer from `Env.RecordWrite`, a new whole-struct passthrough field `RecordWrite loomshed.RecordWriteDeps` like `Env.Darn`. It is registered as `"RecordWrite"`, the engine `shedrecipe.LoomRecordEngines` already names.
`wire` fills `env.RecordWrite` with `recordWriteDeps(location)` for every recipe: the record, support log and discussion paths from loomengine's accessors, `ReadBody` opening the board through `boardengine.OpenHub` and reading the run's task by its slug (never through an agent), and `Commit` over `commitDiscussion`. It writes no relevant list.
`envkit.FullEnv` and shedrecipe's `newTestEnv` fill the field; `InterruptPolicies` gives the row the reinvoke policy; `darnPhases` places it in setup, and `internal/reportengine/doc.go`'s darn setup sentence names Preflight, Record-Write, Drift-Baseline and Drift-Await.

`darn-recipe.yaml` gains `contract: loom` and `verify_source: tree_scope` at the top level and the row `Record-Write` (engine `RecordWrite`) between Preflight and Drift-Baseline, Preflight's `on_done` naming it; the header names the row. The darn writer still finds a record that validates and leaves it, until card 15 stops it writing one.

With both recipes declaring the keys, `New` refuses a recipe that does not declare `contract: loom`, naming `NewPawl` or `lym shed` as the way forward, and its `DarnVerifyAttempts` check and stamp switch from a recipe whose build row's engine is `DarnWrite` to a recipe whose `verify_source` is `tree_scope`, onto the same rows. Until card 19 refuses a non-loom recipe in `armAt`, a batten seed armed through `lym loom` reaches this refusal, since `newShed` branches off only pawl.

Tests:
- `TestRecordWrite_Call` (tier0, over a temp directory and fake seams): a fresh directory gets both files and one commit, the record holding the body verbatim in the fence and `discussionparser.Validate` reporting no finding. A body with `## Decisions` and `## Open risks` headings, one with no backticks, one with inline code only, one holding its own fenced block with a `## Decisions` line inside and one with an unclosed fence each validate, and an `AppendDecision` lands under the record's own `## Decisions`. An existing pair is left byte for byte, and `Commit` is still called once, so a pair a crash left written but uncommitted is committed on resume. A record with no support log gets the support log and a commit, the record unchanged. A torn `.tmp-` file is removed and the written file is whole. A missing entry is Stuck naming `lym board upsert`.
- `TestRecordWriteEntry`: the entry refuses a relative path and a nil seam, each by field name, and builds with a filled Env.
- `TestNewDarn_Routing` gains a row walking `on_done` from Record-Write: Drift-Baseline, Drift-Await, then Darn-Write, the route `lym loom goto --to Record-Write` then `lym loom resume` takes a darn run parked past Record-Write with no record; that Drift-Baseline then passes again keeping its scope record and capture with no commit past the branch point, and takes no capture with commits and `awaited_at`, while Drift-Await is Done, stays pinned by `internal/driftengine#TestDriftBaseline` and `internal/driftengine#TestDriftAwait`.
- `battencli`'s `intentionallyNil` lists `RecordWrite.ReadBody` and `RecordWrite.Commit`, since batten wires no loom record.
- `TestWire_DarnRecipe` asserts `env.RecordWrite`'s paths; `TestWire_EverySeamFilled` sees it filled. `wantDarnProducerTable` adds Record-Write. With Record-Write at number 2, every later darn entry number rises by one: `TestEntryNumbers`' darn map gives Drift-Baseline 3 through the Code-Bouncer entries 7, and `TestRenderChecklist`'s darn rows read `[ ] 2. Record-Write`, then `[ ] 3. Drift-Baseline` through `[ ] 11. Friction-Reflect`. `TestRecipeKeys` now reads darn as loom with record engine RecordWrite and verify source tree_scope.
- `TestNewDarn_Routing` gains a row in which `New` with the batten recipe's name is refused naming `NewPawl` and `lym shed`, and the existing darn rows build under the new keys.

**ImpactSummary:** Adds one producer, one registry entry, one Env passthrough and one darn row; no existing producer changes, darn becomes a declared loom recipe, and `New` refuses a recipe without the contract and keys its verify-attempts stamp on `verify_source`.
