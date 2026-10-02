---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/configreg/fill_test.go:122
    summary: The fill-line key-path assertions match anywhere in the log, and the temp file path in the same line already holds the key-path for the reed subtests, so the card's one-fill-line-naming-the-key-path postcondition is asserted tautologically
  - id: F2
    severity: LOW
    location: internal/ideengine/spawn.go:65
    summary: SpawnDriven's doc comment still names gitignore.Ensure, a function of the package card 7 deletes; card 7's grep covered imports and docs but not Go comments
  - id: F3
    severity: LOW
    location: internal/configengine/config.go:68
    summary: New and changed comments across the diff use fixed-column wrapping where CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line
  - id: F4
    severity: LOW
    location: internal/reedengine/strand_test.go:607
    summary: TestAddStrandUnless_LiveNamedSkips checks only the persisted strand table, so a SaveState on the skip path, which re-stamps socket, session and pane generation, would pass unnoticed
  - id: F5
    severity: NIT
    location: internal/configreg/fill_test.go:144
    summary: TestFill_ReedNestedAndWholeMapping and TestFill_EmptyFileLoadsAsTemplate drop configreg.Template's ok result, so an unregistered module panics on a nil func instead of failing with a message
---

Review range: no `_lyx/loom/rework/` round exists and `_lyx/shed/operator-surface-fixes/status.json` records `product.parent: main`, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (`812600b3a..8780eafce`, the eight card commits).

Evidence gathered: `go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go vet -tags integration ./...`, `go test ./...` and `go test -tags integration ./...` all pass on HEAD.
The three `TestUnlessName_*` real-tmux tests pass when run verbose.
Card 6's Delete group: `assertSpawnedTasksStampTheReedChain`, the one helper the deleted `spawn_test.go` declared, has no remaining caller in any build tag.
Card 7's Delete group: no Go import of `internal/gitignore` remains and no `.md`/`.yaml` outside `_lyx/` names it; the one surviving reference is the comment in F2 (`internal/gitrepo`'s `gitignore.Pattern` is go-git's package, unrelated).

Each card's intent is implemented: `FillMissing` follows card 1's rules, `load` fills, logs, backstops and resolves as card 2 describes, the reconcile hints are rewritten, `AddStrandUnless` and `--unless-name` behave as cards 4 and 5 specify, `Spawn` shares `writeVSCodeConfig` with `SpawnDriven`, and the docs land in their cards' commits.
The findings are test-strength and comment-hygiene gaps, none a behavior defect.

### [MEDIUM] Fill-line key-path assertions are tautological

**Location:** `internal/configreg/fill_test.go:122` (`assertFilled`), and `internal/configengine/config_test.go:121` (`assertOneFillLine`)

**Issue:** Both helpers check `strings.Contains(log, keyPath)` against the whole captured log.
The fill line also carries `file=<cfgPath>`, and `t.TempDir()` names its directory after the test, so for `TestFill_ReedNestedAndWholeMapping/selvage.height_rows` and `/status_line` the path itself contains the key-path and the assertion passes whatever `keys=` holds.
In `configengine`, `home` matches any temp path under a home directory.
Card 2 requires each test to assert one fill line naming the key-path, and as written that postcondition is not pinned.

**Fix:** Assert on the `keys=` attribute: `strings.Contains(log, "keys="+keyPath)` in both helpers (each dropped key-path here is the only filled key, so the value is exactly the path).

### [LOW] Stale `gitignore.Ensure` reference in `SpawnDriven`'s doc

**Location:** `internal/ideengine/spawn.go:65`

**Issue:** "gitignore.Ensure is never called: the child's loom run commits in that worktree." names a function of the package card 7 deletes.
Since card 6, neither entry point calls it, so the line no longer distinguishes `SpawnDriven` from `Spawn` and points at a symbol that no longer exists.
Card 7 asks for a grep confirming no reference to `internal/gitignore` remains; the grep missed Go comments.

**Fix:** Replace the line with the reason the exclude matters for a driven pair, without naming the deleted function, e.g. "The child's loom run commits in that worktree, so .vscode/ must stay out of its commits."

### [LOW] Fixed-column wrapping in new and changed comments

**Location:** `internal/configengine/config.go:68` (`Load` doc), `:92` (`load` doc addition), `:123` (inline comment above the `MissingKeys` backstop); `internal/configengine/config_test.go` (`captureLog`, `writeConfig`, `assertOneFillLine`, `TestLoad_MissingKey`, `TestLoad_FillKeepsExtraAndEmptyValues`, `TestLoad_FilledEnvMarkerUnsetRefuses`, `TestLoad_FillRefusals`, `TestLoadOrTemplate_PresentMissingKey`, `TestLoadOrTemplate_PresentEmpty`, `TestLoadOrTemplate_PresentCommentsOnly` docs); `internal/configreg/fill_test.go` (`loadFilled`, `assertFilled`, `TestFill_DroppedTopLevelKeyLoadsAtDefault` docs); `internal/loomengine/config_test.go` (`TestLoadConfig_MissingFrictionKeys` doc); `internal/websterengine/config_test.go` (`TestLoadConfig_MissingNumericKnobsLoadTemplateDefaults` and `TestLoadConfig_ExplicitZeroKnobNamesTheKey` docs); the changed sentences in `internal/loomcli/wiring_test.go` (`seedLoomConfigWithInteractive`, `seedLoomConfigWithFriction`) and `internal/webstercli/wiring_test.go` (`seedLoomConfigWithFriction`).

**Issue:** These comments break sentences mid-phrase at a fixed column (for example "resolves to its template default and is logged; the file is" / "never written.").
CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line for new and changed Go comments, with legacy wrapping not a convention to match.
The comments in `fill.go`, `spawn.go`, `strand.go` and the new integration tests already follow it.

**Fix:** Rewrite each listed comment one sentence per line, breaking further only at an independent-clause boundary; in the two wiring files, re-break only the changed sentences.

### [LOW] Skip test does not pin the state file

**Location:** `internal/reedengine/strand_test.go:607` (`TestAddStrandUnless_LiveNamedSkips`)

**Issue:** Card 4 requires the skip to leave persisted state unchanged.
The test compares only `loaded.Strands` with the seeded orch strand.
The seeded `ReedState` carries no socket, session or pane-generation stamp, and `loadOrInitStateLocked` stamps all three in memory, so a regression that called `SaveState` on the skip path would change the file while this assertion still passed.
`kill-pane` is also absent from the forbidden-command list, though the reconcile a skip must not run issues it.

**Fix:** Read `reed.json` bytes before the call and assert they are byte-identical afterwards, and add `kill-pane` to the forbidden commands.

### [NIT] Dropped `ok` from `configreg.Template`

**Location:** `internal/configreg/fill_test.go:144`, `:157`

**Issue:** `fn, _ := configreg.Template(...)` turns a missing registry entry into a nil-func panic, while `TestFill_DroppedTopLevelKeyLoadsAtDefault` in the same file checks `ok` and fails with the module named.

**Fix:** Check `ok` and `t.Fatalf` as the first test does.

## Focus verdicts

- Comment line breaks in `config.go`: yes, fixed-column; F3, which also lists the rest of the diff's wrapped comments.
- Card 7 grep and `SpawnDriven`'s `gitignore.Ensure`: stale; F2.
- Card 6 Delete check: `assertSpawnedTasksStampTheReedChain` has no caller under any tag; the chain-stamping assertion survives as `assertInteractiveChain` (absolute commands, `--unless-name orch`, the four labels; `runOn: folderOpen` stays pinned in `internal/vscode/config_test.go`), and the degrade case survives as `TestSpawnPrimeNameFailureOpensBareFolder`. Not a defect.
- `fillMapping` with a null template scalar: no real template holds a null-valued key (every `key:` with no inline value is a mapping), and a file mapping against a null template value refused before this diff too, through `MissingKeys`. `deepCopyNode` on an alias: no real template uses anchors or aliases. Not a defect for the shipped templates.
- Resolution on re-serialized bytes: `yamlengine.Resolve` itself parses into a node tree and re-marshals, and yaml.v3 keeps each scalar's tag and style across the round trip, so `${env:...}` expansion and the values the file held resolve the same as from the original bytes. Not a defect.
- `AddStrandUnless` skip path: `ensureSessionLocked` returns early on a usable session and `loadOrInitStateLocked` mutates only the in-memory state, so the doc comment's claim holds; the engine test asserts no `split-window`/`select-layout`/`select-pane`, with the state-file gap in F4.
- Card 5's integration test: compares `list-panes` id, height and active flag before and after, and covers both the killed-orch-pane and no-orch cases. Not a defect.
- Card 2 tests: byte-identical file, a single fill line, the loaded value and `t.Cleanup` restore are all asserted; the key-path half is tautological in places, F1.
