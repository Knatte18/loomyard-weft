```yaml
lens: fix-affected-set
outcome: reported
```

Unchecked findings: none. I re-ran each set-premise search and checked the fix against every member.

- **F1** (tests of unexported helpers, and repeated rows):
  - Card 2's `TestFromThisHost` targets an exported function, and card 3's tests target `newCommand` through `runName`. Card 4 had the only tests aimed at an unexported helper.
  - The fix puts all eight cut `TestNextFile` rows on `TestName_Set`. Four become new rows, and four are carried by asserting the written `names` on the matching rows.
  - Both `TestHubOfDir` rows become `CwdContainerDirectoryIsSwept` and `PlainCwdAddsNoHub`.
  - No live card still names `TestNextFile` or `TestHubOfDir`.
- **F3** (doc sites that say an entry another machine owns is left alone without `Takeover`):
  - In the Go code the search finds exactly `internal/boardengine/ownerset.go:43`, `ownerset.go:72` and `internal/boardengine/doc.go:183`, and the fix covers all three.
  - The fourth site, `pattern/PATTERN-owner-machine.md:27`, is already rewritten by card 1.
  - The other matches are bench data under `docs/research/`, outside the plan's scope.
- **F4** (per-hub seams that can error): `hubs` and `readOwners` refuse the verb, `pull` marks the hub checked locally, `setOwners` makes it `failed`, and `unpushed` now has an outcome. `cwdHub` returns no error. Every member is covered.
- **F5** (sites that depend on an error naming its own way forward): step 7 and the `FailedHubIsReportedAndOthersSweep` row were the only two, and both are rewritten. The match in card 3 is the show's `hostName` error, which is a different path.
- **F6** (per-hub seams): `deps` defines `pull`, `unpushed`, `readOwners` and `setOwners`. The rewritten `fakeHubs` text names exactly these four.

no findings

Calls:
1. grep for doc claims that only `Takeover` takes another machine's owner, across internal, pattern, contracts and docs. My context had only the three sites the review named.
2. grep of the test-speed pattern and of test targets across cards 2 and 3. My context did not hold those cards.
3. grep for leftover references to `own way forward`, the seam count and the cut tests across the live cards. This checked the F1, F5 and F6 sets beyond card 4's edited spans.
