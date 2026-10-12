---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/loomengine/seedownership_test.go:71
    summary: Fixtures that use an unknown status field as the undecodable file no longer reach any decode-failure branch, and the decode-failure behaviour they name is uncovered
    fix: "Card 2 made an unknown field decodable, but five fixtures still use one as 'undecodable' and still pass. Replace the body with malformed JSON such as `{not json` at each site: (1) `internal/loomengine/seedownership_test.go` subtest `DecodeFailurePasses` (line 84 body, and its comment at lines 71-76 that calls an unknown top-level field a malformed file); (2) `internal/loomcli/start_midmerge_test.go` const `unknownFieldStatusBody` at lines 46-47, rename it `malformedStatusBody` with a comment saying it does not parse, and rename the row at line 333 from 'status file carrying an unknown field' to 'malformed status file'; (3) its use at `start_midmerge_test.go:152` and `internal/loomcli/start_driver_test.go:648`. (4) In `internal/loomengine/seed_test.go` delete the row 'unknown top-level field' at line 96, because the 'malformed JSON' row above it covers the incoherent-file path, and drop '(malformed JSON, an unknown top-level field)' from the `TestCheckSeed` doc at line 60 so it names malformed JSON alone. Re-run `go test ./internal/loomengine ./internal/loomcli` and confirm each changed row still asserts what its name says."
  - id: F2
    severity: LOW
    class: consistency
    location: internal/battencli/overview_test.go:253
    summary: Comment says a seed with an unknown key fails the strict decode, but card 5 made seed.json decode tolerantly
    fix: "In `internal/battencli/overview_test.go` reword the comment at line 253 and the `strict` slug and seed file name around it (lines 254-256, 265-270, 276, 282): the seed with an unknown key now decodes and is listed like any other. Say that, rename the slug to something like `future-key`, and keep the expected line only if the `-` columns still come from the seed lacking `started_at`; otherwise fix the expected line."
  - id: F3
    severity: NIT
    class: consistency
    location: internal/websterengine/state_test.go:114
    summary: Missing space after a comma in the TestLoadState_UnusualFiles doc comment
    fix: "In the doc comment of `TestLoadState_UnusualFiles` change 'as unknown keys,the reserved' to 'as unknown keys, the reserved'."
---

### [MEDIUM] Fixtures that use an unknown field as the undecodable file are vacuous

**Location:** `internal/loomengine/seedownership_test.go:71`, `internal/loomcli/start_midmerge_test.go:46`, `internal/loomcli/start_driver_test.go:648`, `internal/loomengine/seed_test.go:96`
**Issue:** After card 2, `state.ReadJSONNoMkdir` accepts unknown fields.
Four test sites still treat a status file with an unknown field as an undecodable file: `DecodeFailurePasses`, the const `unknownFieldStatusBody` (used at `start_midmerge_test.go:152`, `:333` and `start_driver_test.go:648`) and the `TestCheckSeed` row 'unknown top-level field'.
They pass, but none reaches a decode-failure branch any more, so the decode-failure behaviour of `VerifySeedOwnership` and of the loom start paths has no coverage from them, and their names and comments are now false.
The plan's card 2 names the rigs and tests that relied on the refusal, but not these.
The rubric's test-economy dimension flags a behaviour with no coverage, and a mislabeled test is the same defect.

### [LOW] Stale "strict decode" comment in the batten overview test

**Location:** `internal/battencli/overview_test.go:253`
**Issue:** The comment says a seed with an unknown key "fails the strict decode".
`shedrun.ReadSeed` now accepts it, so the comment and the slug name `strict` describe a behaviour D4 removed.

### [NIT] Missing space in a test doc comment

**Location:** `internal/websterengine/state_test.go:114`
**Issue:** "as unknown keys,the reserved" lacks a space.

## Checked and found sound

- `ReadJSONStrict` has no remaining reference in Go, markdown, YAML or contract text outside `_lyx/`; the rename group's grep is clean, and `GOOS=windows` builds `proc`, `gatecli`, `gateslot` and `state`.
- `keepUnknown` matches D2 and D3: opaque types, case-insensitive declared-key lookup, promoted fields, index-wise slice merge only on equal re-encoding, maps by key, an absent or unparsable old file adding nothing, all under the write lock `writeJSONUnlocked` is already called with.
- `board.json`: `storeFile` and `Task` Marshalers stay opaque through `isOpaque`, and `TestLoad` pins the `Save` bytes through `saveExact`.
- `seedWire` is gone, `WriteSeed` still leaves an agreeing seed untouched, and batten cleanup's judge decodes tolerantly with a covering row.
- The watchdog: sampling, ancestry exemption, `-bench` and `-fuzz` exemption, one SIGQUIT per pid, and the context cancel followed by `watchdogDone.Wait()` leave no goroutine behind; `LYX_GATE_TEST_PID` is replaced before each child spawn.
- Webster, loom and landing read the line only through `gateslot.FindWatchdogQuit` and `BusyQuitWayForward`; webster's gate classifies the first run and the rerun and never reruns a hung first run.
- `lyx gate test` for the touched packages and `cmd/lyx` (test budget, tolerant-records scan, daemon-wakeups scan) is green.
