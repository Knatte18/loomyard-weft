---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/shuttleengine/gate.go:25
    summary: New and changed Go comments across the diff are fixed-column wrapped mid-sentence, violating CONSTRAINTS.md's Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/shuttleengine/gate.go:1
    summary: The gate.go file doc says four exported types while listing five, and the package now exports seven gate types
  - id: F3
    severity: LOW
    location: internal/shedrecipe/entries_burler_test.go:546
    summary: GateAttemptsRejectedInsideProfileSubMap still plants the removed gate_attempts key, so it no longer guards against the row-level gates key being accepted inside the profile sub-map
  - id: F4
    severity: LOW
    location: internal/shedrecipe/entries_gate.go:147
    summary: An unknown gate name is reported as config key name with no gates key or element index, unlike every other per-element error
  - id: F5
    severity: NIT
    location: internal/loomrecipe/fixture_test.go:411
    summary: The fake RunGated/AttachGated doc still says the fake invokes the closure exactly once, though it now runs the list through evalGateList
---

Review range: `product.parent` is `main` (from `_lyx/shed/gate-list/status.json`), merge base `9ae5f9405`, so the range is commits `cdce74404` (Card 1) and `3f79defd0` (Card 2).

Verify block at HEAD: `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` all pass.
At commit 1 alone (extracted with `git archive` into the scratchpad): `go build ./...` and `go vet -tags smoke ./...` pass, and the tests of `shuttleengine`, `shedrecipe`, `burlerengine`, `shedadapters`, `shedbuild` and `loomrecipe` pass.
`websterengine`'s two `batchfail_test.go` failures there come from the extracted tree not being a git repository, not from the change.
Commit 1's `resolveGateSpec` reads `gate`/`gate_attempts` into a one-entry list with `Attempts` 3 as the default, and `AbsentGateAttemptsYieldsPackageDefault` asserts 3, as Card 1 requires.

Per-card mechanical check: Card 1's Create targets `shuttleengine#GateEntry` and `gate_list_test.go` exist, and Card 2's `entries_gate_test.go` exists.
No card carries a `Rename` or `Delete` group.

Focus verdicts, judged against Cards 1 and 2:

- Wait's gated-Done branch increments `gateFails`/`gateSent` only after a successful `Send`, resets the failure count of every entry reported passed on that arrival, finalizes on a non-`PassOnCap` entry whose count reached `Attempts` with the memoised failed outcome (whose `FindingsPath` is that entry's file), and a `PassOnCap` entry at its cap is reported let through without running. Matches Card 1.
- `evaluateGate`: a failing `PassOnCap` entry with budget left and nothing required after it yields `Passed` true with an empty `FindingsPath`, while the re-prompt and Warn line read `gateFindingsPath`; a required entry after it is reported not reached and makes `Passed` false; a list of only off entries yields a non-nil passed outcome. Matches Card 1.
- Attach: `reconstructAndWait` sets only `gate`, and `evaluateGate` sizes the count slices lazily, so an attached run counts exactly as a started one; `evaluateGate` never changes a count, so `finalize` after a deadline or lost session cannot change one. Not a defect.
- `burlerengine.Engine.Run` builds a fresh slice and wraps every entry, the caller's slice is untouched, and the new test recovers `gateSpec[1].Gate` and checks both round paths. Not a defect.
- `shedrecipe`: every rejection Card 2 lists is reached and names the row; the accessor errors also name the element index. The unknown-name case is F4. An `attempts`-0 entry resolves its closure and its `Env` requirements, and a test proves it.
- Every gate-capable entry's `configRejectUnknown` list drops `gate`/`gate_attempts` and accepts `gates`, and `TestGateCapableEntries_RejectLeftoverOldKeys` covers all six entries with both old keys.
- `loom-recipe.yaml`: all six gated rows carry a one-entry `gates` list with `attempts: 3` and no `pass_on_cap`, and no `gate:`/`gate_attempts:` key remains.

### [MEDIUM] New and changed Go comments are fixed-column wrapped

**Location:** `internal/shuttleengine/gate.go` (file doc, `GateEntry.Attempts`, `GateEntry.PassOnCap`, `GateSpec`, `GateOutcome.Passed`/`Attempts`/`FindingsPath`), `internal/shuttleengine/run.go` (`gate`, `gateVerdict`, `gateFailedAt`, `gateFindingsPath`, `gateFails` fields, `StartGated`, `RunGated` docs), `internal/shuttleengine/wait.go` (file doc's gated-Done sentence, `evaluateGate` doc, re-prompt branch inline comments), `internal/burlerengine/engine.go`, `internal/burlerengine/profile.go`, `internal/shedadapters/singlellm.go`, `internal/websterengine/runlevel.go`, `internal/shedrecipe/config.go` (`configMapList`), `internal/shedrecipe/entries_gate.go`, the gate-capable entries' doc comments, and the changed test-fake and test doc comments.
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks — one sentence per line, no column limit — for new and changed Go comments.
The diff wraps these comments at roughly column 100, breaking sentences mid-phrase (for example `GateEntry.Attempts`: "how many consecutive failures re-prompt the agent / before the entry gives up. 0 means the entry is off: it is skipped at every arrival, whatever / its PassOnCap.").
**Fix:** Rewrap every sentence the diff adds or changes to one line per sentence, breaking further only at an independent-clause boundary; leave untouched legacy lines alone.

### [LOW] gate.go file doc miscounts its exported types

**Location:** `internal/shuttleengine/gate.go:1`
**Issue:** The file doc says "the four exported types" and then lists five (Gate, GateEntry, GateSpec, GateResult, GateOutcome); the file also exports `GateEntryState` and `GateEntryOutcome`.
**Fix:** Drop the count and name the types the file declares.

### [LOW] The profile-sub-map guard test no longer guards anything

**Location:** `internal/shedrecipe/entries_burler_test.go:546` (`GateAttemptsRejectedInsideProfileSubMap`)
**Issue:** Its doc says it guards against the row-level and profile allowlists being confused, by planting the row-level gate key inside `profile:`.
After Card 2 `gate_attempts` is a row-level key nowhere, so the test passes for a reason unrelated to its purpose, and a regression that accepted `gates` inside the profile sub-map would go unnoticed.
**Fix:** Plant `gates` inside the profile sub-map and assert the error names it.

### [LOW] Unknown gate name error loses its element context

**Location:** `internal/shedrecipe/entries_gate.go:147` (`resolveGateClosure`'s default case)
**Issue:** Every other per-element error reads `shedrecipe: <entry>: config key "gates" element <i>: ...`, but an unknown name reads `shedrecipe: <entry>: config key "name" must be ...`, naming neither the `gates` key nor the element.
In a multi-entry list the author cannot tell which element is wrong.
**Fix:** Qualify the unknown-name error with the `gates` key and the element index, as the accessor errors are.

### [NIT] Stale fake doc in loomrecipe

**Location:** `internal/loomrecipe/fixture_test.go:411`
**Issue:** The `RunGated`/`AttachGated` doc still says the fake will "invoke the closure exactly once", though it now runs the entries in list order through `evalGateList`.
**Fix:** Describe the list evaluation, as the sibling fakes' docs do.
