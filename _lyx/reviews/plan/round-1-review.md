---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lyx/plan/03-config-cli-machine-layer.md (Intent, the --layer help paragraph); _lyx/plan/00-overview.md (verify section)
    summary: Card 3 adds machine and a gloss to the config command's Long text without budgeting the 2000-byte cap cmd/lyx/clitree_test.go enforces, and the verify section runs no cmd/lyx test, so a breach surfaces only at landing
    fix: >-
      In card 3's Intent, state that buildConfigLong gains only the machine token in its two lists, as "--layer default|worktree|machine|local" and "layer: template, default, worktree, machine or local", and that the gloss (a never-tracked override under the board dir's .lyx, written from the prime, of a hub-wide module's machine keys) goes into the --layer flag help in Command and into configcli/doc.go, never into Long, which today measures about 1978 of its 2000 bytes. In 00-overview.md's verify block, append ./cmd/lyx to the lyx gate test line, so it ends "./internal/configcli ./internal/hubreconcile ./cmd/lyx" (checked with validate-plan --overlay; no new finding).
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/01-config-machine-layer.md (Intent, the ValidateMachine refusal list)
    summary: The prescribed refusal for a covered unit of another category renders an empty category for a CategoryNone key (key "name" is  and cannot be set), names only the hub file where D4 asks for the layers that take the key, and leads a pre-write --set refusal with a removal step that has nothing to remove
    fix: >-
      In card 1's Intent, word the covered-unit refusal per category. For CategoryNone, write key %q is not a machine key; way forward: remove it from the machine override and set it in the hub file through "lyx config <module>" from the prime, or in the local override through "lyx config <module> --layer local". For any other category, keep the existing text naming the category. State that "remove it from the machine override" applies to a file already holding the key, while a --set refusal writes nothing. Make the TestValidateLocal row for name assert the CategoryNone text naming both --layer local and lyx config mod.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lyx/plan/01-config-machine-layer.md (Intent)
    summary: Card 1 introduces the machine category and layer, a new route by which a config value reaches every gate run on a machine without passing Bolt, but its Intent states no bound on what that route lets through, unlike D2's Bound paragraph
    fix: "Add a Bound paragraph to card 1's Intent after the ValidateLocal sentence: the category widens nothing for the local layer; what the machine layer lets through is a value of a machine key written to <BoardDir>/.lyx/config/<module>.machine.yaml, which configcli's --layer machine write guard confines to the prime (card 3) and which a direct file write bypasses, but which never reaches Bolt or any record; a loaded value still passes the module's own loader, so gate's slots and go_parallel stay at least 1, with no upper bound, as on the hub file."
  - id: F4
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/02-gate-machine-keys.md (Intent, the withConfigWayForward forms; Tests, the local-layer row)
    summary: A gate load failure caused by a local override file gets a way forward that cannot fix it; the two-file form claims the hub or machine file holds the value, and the role clause sends the orch to lyx config gate, which leaves the pair's refused gate.local.yaml in place
    fix: >-
      In card 2's Intent, have ConfigFiles also return the local override file (configengine.LayerFile of LayerLocal) when it applies and os.Stat succeeds. Have withConfigWayForward name it with the step "remove <local>, which takes no gate key", and reword the two-file form's "holds the value" to "or the file the error names holds the value". Extend the TestLoadConfig local-layer row to assert the local file's removal step.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/01-config-machine-layer.md (Intent, the pattern/PATTERN-config-layers.md edit)
    summary: Card 1 adds Machine to the category list but leaves pattern/PATTERN-config-layers.md's counted prose, "one of four categories is closed to the local override layer" and "A key meeting none of the four", which become wrong
    fix: "In card 1's PATTERN paragraph, add: the opening paragraph's \"A key meeting one of four categories is closed to the local override layer\" and the \"A key meeting none of the four is CategoryNone\" line are reworded without a count, so every category but None closes a key to the local layer and Machine alone opens it to the machine layer."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/03-config-cli-machine-layer.md (Intent, setMachine and editMachine)
    summary: A refused machine --set or editor save wraps the gate loader's two-file text, whose steps include removing the machine file, inside newLoadFailure's way forward, after rollBack may already have removed that file
    fix: >-
      In card 3's Intent, state that setMachine and editMachine report a load-check refusal through newLoadFailure, whose trailing "way forward: --set the key to a value it takes" is the operative step. Have the TestDispatchLayers rows for --set go_parallel=0 and the go_parallel 0 editor save assert that trailing clause, so the embedded loader steps read as the cause and not as the next step.
---

### [MEDIUM] Long text cap is not budgeted, and cmd/lyx is not verified

**Location:** `_lyx/plan/03-config-cli-machine-layer.md`, the Intent paragraph on `--layer` help; `_lyx/plan/00-overview.md`, `## verify:`

**Issue:** `cmd/lyx/clitree_test.go:164` sets `maxLongBytes = 2000`, and `longProblems` (line 272) fails any command over it.
`lyx config --help` up to `Usage:` is 1980 bytes, so `buildConfigLong` returns about 1978 bytes (no code `cmd/lyx` reads changed between `origin/prod` and HEAD).
Card 3 has `Command` and `buildConfigLong` name `machine` with the parenthetical gloss.
Adding the two `machine` tokens alone takes about 17 bytes, so the gloss in Long breaches the cap.
The verify section only builds `./cmd/lyx` and runs none of its tests, so the card and batch gates do not catch the breach.

### [MEDIUM] ValidateMachine's covered-unit refusal: empty category and missing local layer

**Location:** `_lyx/plan/01-config-machine-layer.md`, Intent, the first `ValidateMachine` bullet

**Issue:** The template `key %q is <category> and cannot be set in a machine override` interpolates `CategoryNone`, whose value is `""`, as nothing.
The card's own `name` row reaches this.
D4 says a refused `--set` names "the key and the layers that take it".
A `CategoryNone` key of a hub-wide module is taken by the hub file and also by the local layer, which `ValidateLocal` admits, but the message names only the hub file.
On the `--set` path, `setSparse` refuses before writing, so "remove it from the machine override" names a step with nothing to act on.

### [MEDIUM] Card 1 states no bound for the machine route

**Location:** `_lyx/plan/01-config-machine-layer.md`, Intent

**Issue:** The rubric's attack-surface bullet requires the card that introduces a routing edge to state, in its `**Intent:**`, what the edge lets through and what bounds it.
Card 1 adds the layer through which a machine-local value reaches every gate run of the hub on that machine.
Its Intent says only that `ValidateLocal` keeps refusing a machine key.
It does not carry D2's Bound: the guard confines writes to the prime but not direct writes, values never reach Bolt, and the loader floor applies with no ceiling.
Card 3 carries the matching bound for its write path; card 1 has none for the layer itself.

### [LOW] A local-file gate failure gets a way forward that does not fix it

**Location:** `_lyx/plan/02-gate-machine-keys.md`, Intent (the `withConfigWayForward` forms) and the `TestLoadConfig` local-layer row

**Issue:** `ValidateLocal` refuses every gate key, so a `gate.local.yaml` holding one fails the load.
Card 2 then appends either "fix it with lyx config gate" or the two-file text claiming the hub or machine file "holds the value".
Neither touches the local file the error names.
The new role clause sends a pair to report FAILED so that the orch runs `lyx config gate`, which leaves the failure in place.
The defect exists today, but card 2 rewrites this message and widens its claim.
Such a file arises only from a direct write or an older build, because `--set` and editor saves are refused or put back.

### [LOW] PATTERN-config-layers keeps its "four categories" counts

**Location:** `_lyx/plan/01-config-machine-layer.md`, the `pattern/PATTERN-config-layers.md` edit

**Issue:** `pattern/PATTERN-config-layers.md` says "A key meeting one of four categories is closed to the local override layer" and "A key meeting none of the four is `CategoryNone`".
Card 1 adds Machine to the list but does not touch these counts.
After the change they name the wrong number, and Machine differs in kind from the four guard-type categories, since it opens a key to a layer.

### [LOW] A machine load-check refusal carries a stale removal step

**Location:** `_lyx/plan/03-config-cli-machine-layer.md`, Intent, `setMachine` and `editMachine`

**Issue:** `setChecked` wraps the after-load error in `newLoadFailure` (`internal/configcli/configcli.go:265`), then calls `rollBack`.
With card 2, that after-load error names "removing `<machine>`" and the role clause.
When the refused write created the file, the rollback has already removed it.
The message then lists a removal of a file that is gone, ahead of the operative "--set the key to a value it takes".

## Rejected

- guard-reach, the `selectedLayer` refusals are pre-empted on the no-terminal editor path: `dispatch` refuses `!terminal` first, and `noTerminalRefusal`'s way forward, `lyx config loom --layer machine --set key=value`, then reaches the `--layer local` refusal, so the path progresses in two hops. Card 3 claims only that the refusals precede the hub-write guard, which holds.
- Handler, the overview's Shared Decision keeping `ErrConfigAbsent` single-file with a machine file present: an absent strict base is not a value either file holds, no machine-file step progresses from it, and D3's rationale concerns a value below 1 after the merge. This is a reading of D3, not a departure from it.
- Handler, granularity of `MachineFile`, `MachineKeys` and `ValidateMachine`: each is a small part of the machine-layer unit card 1 builds, beside `LocalFile`, `LocalKeys` and `ValidateLocal`. Splitting them would give one card per literal symbol, which the granularity bullet rules out.

## Focus settlements

- D3: card 2 carries the role clause in both `withConfigWayForward` forms. The `GateSlots` wrapper carries it through the wrapped `LoadConfig` error, and `runTest`'s new way forward defers to it. The `Dirs{Board}` and no-machine-file cases take the one-file form naming the hub file's step only. Carried, apart from F4's local-file case.
- D4: card 3 carries the pre-write refusal (through card 1's `setSparse`/`setEntries` check), the editor-save put-back (card 1's `Edit` plus card 3's `editMachine` snapshot and `rollBack`), both `--layer machine` refusals with their ways forward, `--resolved` (card 1's `Provenance`) and the menu mark. Carried, apart from F2's wording.
- D1/D5 and the notes: card 1 judges the Anchor-and-Board rule on the told `Dirs` through `readLayers(..., anchorLayers)`. Card 2's `TestModules_EveryTemplateKeyClassified` rejects a per-worktree `CategoryMachine` key, with a synthetic row seen failing. The three PATTERN background files are edited by card 1 and both `doc.go` files by card 2.
- Reachability: every refusal a card prescribes names a reaching input and test row, and no upstream check rejects that input first. The template-lacks-key branch is pre-empted on `Set` by `yamlengine.SetSparse`, but both refuse before writing, and `TestValidateLocal`/`TestLayers` reach it directly.
- Test economy: no `hubreconcile` test pins a `.lyx` file through `Ensure` (`TestEnsure_*` in `ensure_integration_test.go`), so card 4 is new coverage. Cards 1 to 3 add rows to existing tests, not per-symbol tests.
- Granularity and Edit glyphs: see Rejected. Every `Edit` card lists member glyphs, and the two signature changes, `withConfigWayForward` and `noTerminalRefusal`, carry the re-sign arrow.
- Verify coverage: every package a card targets is in the gate test line. The one gap is `cmd/lyx`, which reads `buildConfigLong` (F1).
- CLI shape: `--layer machine` is a new value of an existing flag on the operator-audience `config` command, and the audience bullet of `pattern/PATTERN-cli-cobra.md` admits it.
- Attack surface: card 3 states its bound; card 1 does not (F3).
