```yaml
lens: repo-invariants
outcome: reported
```

**Lens: repo-invariants.** I checked test seams, test tiers, doc text the cards falsify, and whether each card can pass its own gate from the cards before it.

## Per card

- **Card 1**
  - Seams: held. No reflective `//lym:seam` covers any `gitrepo` struct.
  - Tier: at risk. It adds one tier0 test and one tier1 test and names no file for either.
  - Docs: held. `gitrepo/doc.go` is extended.
  - Gate (deadcode guard): broken until card 3. `RefNamespace.Branch`, `RefNamespace.Tag`, `RefNamespace.LocalRef` and `RemoteFetchRefspecs` have no caller reachable from a `main` package.
  - Vocabulary: held.
- **Card 2**
  - Seams and tier: held. The test is tier1 and says why it is not parallel.
  - Gate (deadcode guard): broken until card 3. Nothing in production reaches `originNamespaceAt` or `oneRepositoryNamespace`.
  - Docs: held.
- **Card 3**
  - Seams: held. `pushDestination` has no seam marker.
  - Docs: held. The doc comments sit on members the card edits, and it also edits `gitrepo/doc.go`.
  - Callers outside fabricengine and gitrepo: held. The injected `Push` dependencies are function values, not `gitrepo` calls.
  - Vocabulary: broken. Its planned `gitrepo` test rows spell the side token.
- **Card 4**
  - Seams, docs and signatures: held.
- **Card 5**
  - Seams: held. Nothing reflects over `CloneOptions` or `CloneResult`.
  - Re-signed callers: held. `checkSubpathBeforeReset` is called only in `clone.go:219`; `fetchWeftBranchTip` only in `recordsprobe.go:166`, through `probeMainWeftHasPath`.
  - Destructive-guard allowlist: held. `destroy.go` and `recordsprobe.go` are allowlisted.
  - Gate (deadcode guard): broken until card 6. Nothing in production reaches `PushCloneBoard`, `AbandonSeededClone` or `pushCloneSeed`.
- **Card 6**
  - The `runCloneWithReset` re-sign has one caller, `fabriccli/fabric.go:88` (the `Command` closure), which the card edits: held.
  - The new row goes into a tier0 table. That it spawns nothing is not verified here; guard-reach owns it.
- **Card 7**
  - The `cloneHub` re-sign has three callers (`hub.go:250`, `hub.go:271`, `template.go:223`), all in the card: held.
  - `Hub` field docs: held.
- **Card 8**
  - The `cleanup.go` header sentence is named in the Intent: held.
  - Order: held. It uses `NewOneRepoHub` from card 7, and card 7 comes first.
- **Card 9**
  - Order: held. It comes after cards 7 and 8.
  - Tier1 tests: held.
- **Card 10**
  - `PATTERN.md` and `pattern/` invariants: held. The `README.md` vocabulary belongs to the handler's review.

## Findings

1. **MEDIUM, design.** Location: cards 1, 2 and 5 (`internal/gitrepo#RefNamespace`, `plan:internal/fabricengine#originNamespaceAt`, `plan:internal/fabricengine#PushCloneBoard`).
   - These cards add production functions that no `main` package reaches until a later card: card 3 for cards 1 and 2, card 6 for card 5.
   - `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` is a tier1 `//lym:guard` test with no `//lym:watch`, so the batch gate runs it by name.
   - The batch gate is `batch: {tiers: [0, 1], scope: card}`. Any batch boundary after card 1, card 2 or card 5 and before its first caller therefore fails.
   - No card names the guard.
   - Fix: land each producer with its first caller (merge card 2 into card 3, card 5's verbs into card 6), or name the batching constraint.
   - Evidence: `cmd/lym/testonlycode_linux_test.go:77-79`, `_bolt/_lym/config/gate.yaml` (`gates.batch`), card 2 ("until then it is reached only by its test"), card 5 ImpactSummary ("no caller until card 6").

2. **BLOCKING, consistency.** Location: card 3 (`internal/gitrepo_test#TestDeleteRemoteBranch`).
   - The planned rows spell `refs/lym/heads/x-weft` and `refs/lym/heads/y/x-weft` in a `gitrepo` test file.
   - `gitrepo` is outside the fabric vocabulary owner set, and every `*_test.go` is in scope. `BareToken` matches the side token as a case-insensitive substring.
   - `internal/gitrepo` holds no side-token text today, so `TestEnforcement_FabricVocabulary` would fail. This breaks `PATTERN-fabric-boundary`.
   - Fix: use branch names without the side token, such as `x-records` and `y/x-records`.
   - Evidence: `pattern/PATTERN-fabric-boundary.md` (owner set and scope list), `internal/fabricvocab/fabricvocab.go:62-92`, `grep -rln weft internal/gitrepo` (no hits).

3. **LOW, consistency.** Location: card 1 (`TestRefNamespace` is tier0, `TestRemoteFetchRefspecs` is tier1).
   - The card names no file for either test.
   - Every test function in one file must name one tier, and a tier0 file may spawn no git (`cmd/lym/tiertags_test.go`, `cmd/lym/tierpurity_test.go`).
   - Fix: state that `TestRefNamespace` goes in a tier0-only file and `TestRemoteFetchRefspecs` in a tier1 file.
   - Evidence: `pattern/PATTERN-test-speed.md`, the "Four tiers" and "tier0 tests spawn nothing" sections.

## Tool calls

1. Grepped `//lym:seam` markers and reflective test files to find seam coverage of the changed structs; the manifest had no seam inventory.
2. Read the deadcode guard's tier and markers, the callers of the re-signed unexported functions, and the `fabricvocab` function list; none were in context.
3. Read the `fabricvocab` side-token predicate and located `gate.yaml`, to judge how the vocabulary scan matches.
4. Grepped `gate.yaml` for the card and batch keys; it matched nothing, because the file indents differently.
5. Read the head of `gate.yaml` for the batch key's tiers and scope.
6. Grepped `fabricengine/doc.go` and `gitrepo/doc.go` for text the cards' changes would falsify; no hits beyond the members the cards already edit.
