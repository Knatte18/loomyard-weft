```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the correctness lens on the fix's changed code and tests.

No findings.

I ran concrete inputs through each changed path and found no wrong result.

- **Split order:** with `bigCap = 340` and a 129-byte header, the pieces come out as header+hunk one (205 bytes), then header+hunk two plus the `@@` line and `c01..c11` (336), then header plus `c12..c50` (324). That matches the `wantContents` in `FileOverTheCapSplitsAtHunksThenLineEnds`. Cutting at arbitrary bytes would fail that row.
- **UTF-8 cut:** in `LineOverTheCapIsCutOnARuneBoundary`, a raw cut at byte 200 lands on a continuation byte, so the `utf8.Valid` check fails without `runeBoundaryAtOrBefore`.
- **Line cap:** the `ManyShortLinesOverTheLineCapSplit` and `PackageOverTheLineCap` rows fail without the line count.
- **Package from path:** `docs/foo b/bar.md` is read from the `+++` line and gives `docs/foo b`. Reading the `diff --git` header would give `.`.
- **Name cap:** `DeepPackageNameIsCapped` fails without the 80-byte cap, because the uncapped name exceeds 255 bytes and `WriteFile` errors.
- **Write failures:** the two blocked rows reach the `MkdirAll` and `WriteFile` error returns.
- **Committed diff:** `TestFabric_ReadCommittedDiff` would fail if the read used the working tree, since `lib.txt` and `untracked.txt` would then appear.
- **Callers:**
  - `writeDiffPieces`'s only caller, `review.go`, passes the new `pieceCap`.
  - `subjectDiff` and its users are gone, and `stage_diff` now requires `Env.CodeDiff`.
  - `CodeDiff` is wired in `loomcli`, defaulted in `envkit` and listed in `nilLegal`.
  - `CodeSubject` was only ever wired in `wiring.go`, so no other `Env` builder breaks.
- **`codeSubject` refactor:** `recordedTaskBase` keeps the not-found, error and found behaviour of the code it replaced.

Edge cases I judged out of scope:
- `pieceFileName`'s `for !utf8.RuneStart(name[0])` could index past the end only if all 80 kept bytes were UTF-8 continuation bytes. A path with an 80-byte run of such bytes is not a realistic input.
- `continuationHeader` assumes a file section does not begin with `@@ `, which git output never does.

No further tool calls: my context already held every changed file and the gate results.
