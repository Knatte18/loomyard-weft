---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/02-config-load-fills.md (Edit list, Tests)
    summary: card 2 misses internal/websterengine/config_test.go, whose TestLoadConfig_MissingNumericKnobNamesTheKey fails under D1 and lies in no card's targets
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/01-yaml-fill-missing.md and _lyx/plan/02-config-load-fills.md (Tests)
    summary: no card names a test for the acceptance criterion that unparseable YAML is still refused, and configengine has none today
  - id: F3
    severity: LOW
    location: _lyx/plan/02-config-load-fills.md (Tests, configengine bullet)
    summary: card 2 flips three configengine refusal tests but omits TestLoadOrTemplate_PresentCommentsOnly, which also fails under D1
  - id: F4
    severity: LOW
    location: _lyx/plan/06-ide-spawn-clean.md (Tests, tracked tasks.json bullet)
    summary: the tracked-tasks.json integration case never exercises the prime, so the D4 clause that a tracked skip still writes the hub workspace file and launches it goes untested
  - id: F5
    severity: LOW
    location: _lyx/plan/06-ide-spawn-clean.md (Delete spawn_test.go)
    summary: deleting spawn_test.go drops the only test of Spawn's prime-resolution degrade warning, and the card neither moves that case nor says it is dropped
  - id: F6
    severity: LOW
    location: _lyx/plan/05-reed-add-unless-flag.md (Intent)
    summary: card 5 does not state that --unless-name is checked before --if-absent, so a live match also suppresses the relaunch of a dead --name strand, nor that the flag needs neither --name nor --if-absent
  - id: F7
    severity: NIT
    location: _lyx/plan/02-config-load-fills.md (Edit list)
    summary: test-helper comments in internal/loomcli/wiring_test.go and internal/webstercli/wiring_test.go say configengine.Load is strict on missing keys, which D1 makes false, and no card targets them
  - id: F8
    severity: NIT
    location: _lyx/plan/02-config-load-fills.md (Intent, step 3)
    summary: step 3 says the MissingKeys backstop only reports a key missing inside a present list element, but MissingKeys also reports element keys of a shortened or emptied list of mappings
---

The plan carries every decision (D1–D10) and every constraint of the decision record.
No rework round exists, so the decision record alone is the answer key.
To check card 2's blast radius, I applied a stand-in for D1 to a scratch copy of the tree: the missing-key refusal in `configengine`'s `load` was swapped for a merge.
Then I ran `go test ./...`.
The failures beyond the scratch copy's own missing `.git` are `configengine` (`TestLoad_MissingKey`, `TestLoadOrTemplate_PresentMissingKey`, `TestLoadOrTemplate_PresentEmpty`, `TestLoadOrTemplate_PresentCommentsOnly`), `loggerconfig` (`TestLoad_MissingKeyErrors`), `loomengine` (`TestLoadConfig_MissingFrictionKeys`) and `websterengine` (`TestLoadConfig_MissingNumericKnobNamesTheKey`).

### [MEDIUM] Card 2 misses the webster config test D1 breaks

**Location:** `_lyx/plan/02-config-load-fills.md`, Edit list and Tests.
**Issue:** `internal/websterengine/config_test.go`'s `TestLoadConfig_MissingNumericKnobNamesTheKey` writes a `webster.yaml` lacking numeric knobs and expects `LoadConfig` to refuse, naming the key.
Under D1 those keys fill from the template (`self_fix_cap: 2`, `recovery_timeout_min: 60`), so its `every numeric knob absent` and `recovery_timeout_min absent` cases load and fail; the scratch-copy run confirms both.
The `poll_wait_s explicitly zero` case still refuses through webster's own positive-integer check.
Card 2's catch-all ("fix any other test `go test ./...` surfaces") reaches it only by editing a file no card targets, and the ImpactSummary hides that a webster test flips.
**Fix:** add `internal/websterengine/config_test.go` to card 2's Edit list, with a test bullet: the two absent-key cases become positive cases loading at the template defaults, and the explicit-zero case keeps its refusal.

### [MEDIUM] No test for the unparseable-YAML refusal

**Location:** `_lyx/plan/01-yaml-fill-missing.md` and `_lyx/plan/02-config-load-fills.md`, Tests.
**Issue:** the acceptance criteria list unparseable YAML among the still-refused cases.
Card 1 states the parse-error rule, but its enumerated `fill_test.go` cases leave it out; card 2's configengine cases leave it out too, and `internal/configengine/config_test.go` has no parse-error test today.
The refusal moves from `MissingKeys` to `FillMissing` in this task, so this is the moment it can regress unnoticed.
**Fix:** name a parse-error case in card 1's test list (either side unparseable), and a configengine case in card 2 asserting `Load` on an unparseable present file refuses with the file path named.

### [LOW] Card 2 omits `TestLoadOrTemplate_PresentCommentsOnly`

**Location:** `_lyx/plan/02-config-load-fills.md`, Tests, configengine bullet.
**Issue:** the bullet flips `TestLoad_MissingKey`, `TestLoadOrTemplate_PresentMissingKey` and `TestLoadOrTemplate_PresentEmpty`, but `TestLoadOrTemplate_PresentCommentsOnly` also asserts a refusal and fails under D1 (card 1 returns the template for a comments-only file).
**Fix:** add it to the flipped list: a comments-only present file loads as the template.

### [LOW] The tracked-skip test never covers the prime

**Location:** `_lyx/plan/06-ide-spawn-clean.md`, Tests, last integration bullet.
**Issue:** D4 says a tracked `tasks.json` still launches, "the prime's hub workspace file included", and card 6's Intent carries that sentence.
The test bullet names no fixture and asserts only "still launches", so the prime branch (workspace file written, workspace path launched after the skip) is untested.
**Fix:** run the tracked case on the prime as well, asserting the hub workspace file is written and is the launched path.

### [LOW] Deleting `spawn_test.go` drops the degrade-path test silently

**Location:** `_lyx/plan/06-ide-spawn-clean.md`, Delete `internal/ideengine/spawn_test.go`.
**Issue:** `TestSpawnPrimeSlugDegradesWhenDetectionFails` is the only test of `resolveSpawnTarget`'s warning and bare-folder fallback when `fabricengine.PrimeName` fails.
The card moves the chain-stamping assertion into the new integration file but says nothing about this case.
The path stays reachable: `PrimeName` reads the caller's `AnchorPath`, not the spawned worktree.
**Fix:** move the case into `spawn_clean_integration_test.go`: a `Location` whose worktree is not a git checkout spawning a real pair logs the `resolve prime name` warning and launches the pair's bare folder.

### [LOW] `--unless-name`'s interaction with `--if-absent` and `--name` is unstated

**Location:** `_lyx/plan/05-reed-add-unless-flag.md`, Intent.
**Issue:** card 4 places the check before the `IfAbsent` branch, so with both flags a live `orch` also suppresses `--if-absent`'s relaunch of a dead `claude` strand.
That is one more thing the flag can skip, and card 5, which introduces the CLI surface, does not state it.
It also does not say whether `--unless-name` requires `--name` or `--if-absent`, while the acceptance test combines all three flags.
**Fix:** state in card 5's Intent and help paragraph that `--unless-name` is evaluated first, before `--if-absent`'s match, that a skip also suppresses an `--if-absent` relaunch, and that the flag stands alone without `--name` or `--if-absent`.

### [NIT] Stale "Load is strict on missing keys" comments

**Location:** `_lyx/plan/02-config-load-fills.md`, Edit list.
**Issue:** the doc comments on `seedLoomConfigWithInteractive` (`internal/loomcli/wiring_test.go`) and `seedLoomConfigWithFriction` (`internal/webstercli/wiring_test.go`) justify their full-key literals by saying `configengine.Load` is strict on missing keys; after D1 that is false.
**Fix:** add both files to card 2's Edit list and reword the comments: the literal pins every value explicitly rather than relying on template fills.

### [NIT] The backstop's description is narrower than `MissingKeys`

**Location:** `_lyx/plan/02-config-load-fills.md`, Intent step 3.
**Issue:** `sequenceBasePath` recognises only a trailing index, so `MissingKeys` also reports `list[N].key` paths when a list of mappings is shorter than the template's, emptied included.
No non-seed template holds a list of mappings today, so behaviour is unaffected; the step's wording and the doc it seeds are inaccurate.
**Fix:** say the backstop reports template keys inside list elements of mappings that the file's list lacks, whether the element is present or the list is shorter.

## Focus verdicts

- `spawn_hub_integration_test.go` under D4/D5: not a defect.
  hubforge primes are real git worktrees with no tracked `.vscode/`, so `PathTracked` and `ExcludeAnchoredDir` succeed at both anchors.
  Its assertions read only the launch argument, the workspace folders and the workspace bytes, which splice `settings.json` and never `tasks.json`, so none change and card 6 need not name the file.
  `idecli`'s `TestRunCLISpawnDispatch` already tolerates a spawn failure, and the menu tests spawn real git worktrees.
- Card 6's tracked branch: the Intent carries D4's whole sentence; the tests do not (F4).
  The not-a-git-checkout refusal tightens rather than bypasses, and the Intent bounds it ("as `SpawnDriven` already does"), so it is not a finding.
- Card 3 and D8: the grep over `internal/`, `docs/` and `contracts/` finds no adopt-a-new-key hint beyond card 3's named targets and the `configengine` error string and doc that card 2 owns.
  The leave-alone list matches D8's kept absent-file hint and the Out scope's seed-only modules; `burlerengine`'s no-fans error is a materialize-absent-file hint.
- Docs constraint: `docs/overview.md` lists reed verbs but not `add`'s flags, so no reed edit is due there.
  Card 4 carries the reed package doc, card 5 the CLI help.
  Card 3 edits the landing sentence and card 6 the **ide** bullet: separate paragraphs, consistent wording.
- Card 2 against D1: every refusal and positive case has a named test except unparseable YAML (F2).
  The read-error branch is unchanged code and needs none.
  Cards 1 and 2 agree on the list-element case, modulo F8.
- Granularity: cards 1, 2, 4 and 5 are each one reviewable, testable unit.
  Card 2's extra test files test its own behaviour change, and `liveStrandNamed` is a private helper belonging to card 4.
- Card 5's attack surface: the skip and its bound are stated; the flag interaction is not (F6).
