---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: ruling
    reason: "Parent ruling: keep ReferenceEngine and ComparedEngine as constants and record a Pattern-Departure naming PATTERN-code-conventions. Carried out as the empty commit shuttleparity: fix F4 with that trailer."
  - id: F5
    disposition: fixed-differently
    reason: "The proposal asked for a removal assertion on every TestReport row, which the report's directory cannot give on an unknown-seat row (no engine run directory to stat). A separate serial test globs the process's lymtemp root for manifest-report-* before and after an unknown-seat refusal and a failing Prepare, which pins removal on error paths."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: deferred
    reason: "Added the no-shortname refusal test. The positive row that pins NewWithSeams(UnrunHooks, ConfiguredSandbox) needs a full hub fixture that runs both real engines' Prepare over real seat stand-ins, which this fixer cannot build alone this round; the seams stay unpinned by any test."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Fixer report, round 1

## Changes

Each fix is its own commit, except where one file carried several findings.

- F1, F2: `internal/heddleengine/target.go` declares `ClaudeLocalInstructionsFile`, `CodexOverrideInstructionsFile` and the unexported `claudeEngine` and `codexEngine`, and the table rows use them.
  `claudeengine` drops `localInstructionsFileName` for the heddle constant, `claudeengine/capabilities.go` uses it, and `codexengine`'s `agentsOverrideFile` is the heddle constant.
- F3: `codexengine/manifest.go` lists an omitted hook with no plugin keyed by its name.
  The Codex manifest test has a row for it that asserts the item and its verdict entry.
- F5: see the disposition reason.
- F6: `TestManifestCmd` has an `unrun` report holding an omitted plugin context and a verdict, and asserts the unrun entry, a null verdict and the missing list as `channel key` strings.
- F7: `TestManifestReportOf_RefusesAHubWithNoShortname`; the wiring row is deferred.
- F8: `readHeddleStanding` in `cmd/lym/passagereader.go`, wired from `main.go`, with `TestReadHeddleStanding` over a heddle store holding one prompt and one file entry.
  The file entry is an `agents-file` entry, so its engine key is empty; heddle normalizes a `claude-file` entry to `agents-file` on read.
- F9: the doc comment and log text on the Claude resolution failure now state what the manifest lacks and that a Codex launch refuses on the same source error.
- F10: Claude manifest rows for an unparseable hooks file, a failed setup resolution and an anchor with no repository; the verdict check now compares keys.
- F11: both manifest tests have a row planting a directory at `manifest.json` that asserts the Prepare error names the way forward.
  The Claude write error now carries `way forward: make the run directory writable`, as the Codex one did.
- F12: `//testtiming:keep` above `TestShowAndPassagesReadTheShippedLayerFirst`.
- F13: `docs/overview.md` verb list names `manifest`.

Gate repair: the gate refused three new test comments for a wrapped break or a line of 120 characters or more.
The comments on `TestReadHeddleStanding`, `TestManifestReportOf_RefusesAHubWithNoShortname` and `TestReport_RemovesTheTemporaryDirectoryOnARefusal` now break at sentence boundaries and stay under the limit.
No code changed.

Gate: `lym gate test` over the touched packages and `cmd/lym` passed before the repair.

## Disputed

None.

## Deferred

- F7, the positive wiring row: see the disposition reason.

## Rulings

- F4 (`decision` class): ruling from the parent, recorded as "The parity check names its two engines as constants": keep `ReferenceEngine` and `ComparedEngine` and record a Pattern-Departure.
  Carried out as an empty commit `shuttleparity: fix F4` with a `Pattern-Departure: PATTERN-code-conventions` trailer giving the reason.
  The commit that added the constants is already in history, so the trailer sits on this commit.
  The parent was not told, since the ruling was carried out.

## Self-check

- fix-code-correctness, MEDIUM: F7 refusal-only test leaves the `NewWithSeams` seams unpinned. Disposition: deferred, as under F7.
- fix-code-correctness, LOW: F8 file entry has an empty engine key, so no file-entry engine key is pinned. Disposition: fixed as proposed by no change; the prompt row pins the engine key and the entry's empty key is heddle's behaviour for an `agents-file` entry.
- fix-proposal, LOW: F7 only the refusal row exists. Disposition: deferred, as under F7.
- fix-proposal, LOW: F5 differs from the proposed change. Disposition: fixed-differently, as under F5.
- fix-proposal, LOW: the Claude write error's way-forward suffix may duplicate one from `state.WriteJSON`. Disposition: no change; the fork's grep of `internal/state` found no such text.
- fix-consistency: no findings.
- fix-affected-set: no findings.
