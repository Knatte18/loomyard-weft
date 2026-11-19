---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

## Changes

- F1 (`7af8ea84`, `internal/burlerengine/verdict.go`, `verdict_test.go`): `malformedQuotedSummary` now matches within one line (space-or-tab classes) and its fragment skips backslash-escaped quotes.
  `TestParseReview` gained a fully quoted summary with an unquoted colon in `fix` (colon hint), a quoted fragment followed by prose holding a colon (fragment hint), and an `errAbsent` assertion on the bad yaml row.
- F1 follow-up from the self-check (`internal/burlerengine/verdict.go`, `verdict_test.go`): the pattern's trailing class is `[^\s#]`, so a fully quoted summary with a trailing YAML comment no longer matches; one test row pins it.
- F2 (`internal/haltnote/haltnote.go`): the `routedPrefix` comment and the second line of the `Row.RoutedTo` doc name a routed Stuck or a reroute.

- Gate repair: the new `malformedQuotedSummary` and `routedPrefix` comment lines that reached 120 characters are split at sentence boundaries, with no change to the code.

Verify: `lym gate test ./internal/burlerengine ./internal/haltnote` passed after each round of edits.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW, a fully quoted summary with a trailing `# comment` still matched the fragment regex; fixed as proposed (the follow-up above).
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
