---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/loomcli/overlay_test.go:159
    summary: readOverlay's syntax-error arms (invalid or truncated JSON, a scalar) have no test row, though only the array, second-value, duplicate-name and non-string-value arms are pinned
    fix: "In TestValidatePlanCmd_Overlay in internal/loomcli/overlay_test.go, add one table row beside the array row, named invalid JSON is malformed, with a truncated object as input (an opening brace, one quoted key and a colon, nothing after), wantExit 1, and wantOut holding malformed overlay and the way forward text pipe one JSON object. Add it as a row, not as a new top-level test, so test-budget.yaml stays unchanged."
---

### [LOW] Invalid and truncated JSON stdin is not covered

**Location:** `internal/loomcli/overlay_test.go:159`
**Issue:** `readOverlay` (`internal/loomcli/overlay.go`) has three `malformed("%v", err)` arms that wrap a decoder syntax error, at the opening token, the key token and the closing token.
`TestValidatePlanCmd_Overlay` pins the array, second-value, duplicate-name and non-string-value arms, but no row feeds input that is not JSON at all or an object cut off mid-way.
Card 5's Intent lists "invalid JSON" among the malformed inputs, so this behavior is part of the stated contract.
The code is correct on reading: a truncated object reaches the closing `dec.Token()` and returns `io.EOF`, which is wrapped as `errOverlayMalformed`.
The gap is coverage only, and the refusal is a clear error, never a silent pass.

## Verification notes

These are what I checked, not findings.

- **Range:** `_lyx/loom/rework/` does not exist, so the range is `git merge-base main HEAD..HEAD`, seven commits.
- **Tests:** `lyx gate test` is green for `planglyph`, `planparser`, `loomcli`, `websterengine`, `loomshed`, `lyxcwd`, `contracts/stencils` and `cmd/lyx`, and `--tags integration` is green for `planglyph`.
- **Cards 1–2:**
  - `delete-before-reference` finds a deleted member only through `memberReferences`, in `.go` files outside the directories `ignoredByGoTool` rejects.
  - `LaterDeleteReferences` passes a nil loader, so it is scan-only.
  - `planGatePass` gets the typed form through `callerCoverageFindings`.
  - `laterDeleteReferenceReasons` keeps only `SeverityBlocking`, and it is the one site both record-batch and recover-batch reach.
  - `deletedSymbol` is gone.
  - `validateFormatAfter` calls `dispatchFindings`, so with a done card each reference is reported once.
- **Cards 3–5:**
  - No remaining plan-file read in the check path bypasses `Plan.ReadFile`, `Plan.RewriteRefs` and `Plan.Reparse`.
  - `os.ReadDir(plan.Dir)` in `checkIndexFileConsistency` is a directory listing, which an overlay cannot change because it adds and removes no file.
  - `pendingCardsByID` and `cardsByID` copy the `Plan` by value, so the overlay survives the scoped views and `CanonicalizeHandles` still rewrites in memory.
  - `--overlay` refuses a terminal, empty or malformed stdin, a name that is not a plan file, a missing card file and a flag combination, each with its way forward, and none passes silently.
- **Card 6:** the rubric text matches the shipped `--overlay` behavior and the help text, and the `wordingClaims` rows pin it.
