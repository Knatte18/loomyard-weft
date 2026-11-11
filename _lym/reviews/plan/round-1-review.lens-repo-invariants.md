```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card, I checked reflective seam tests and test-file tiers, doc text the card falsifies, and whether the card can pass its gate in order.

**Per card, invariants checked**

- **Card 1 (owner-replace-set)**
  - Reflective seam test covering `OwnerRequest` or `OwnerWrite`: none exists, so the new `Replace` field needs no `//lym:seam` coverage. Held.
  - Tier: the new rows and subtests go in `TestSetOwners`, which is tier 1 (`ownerset_integration_test.go:157-158`). Held.
  - `PATTERN.md` overview line: the declared text equals the current line (`PATTERN.md:24`). Held.
  - `pattern/PATTERN-owner-machine.md:27` clause: the card lists and rewrites it. Held.
  - `internal/boardengine/doc.go` commit line (`:174`): the card lists and rewrites it. Held.
  - Doc text the card falsifies: `OwnerRequest.Takeover` field doc (`ownerset.go:43`), `SetOwners` doc (`ownerset.go:72-73`), and `doc.go:183-184`. **Broken** (finding 1).
  - Order: the card stands alone. Held.
- **Card 2 (hostname-name)**
  - Tier: `TestFromThisHost` is tier 0, the tier `machinename_test.go` already uses (`:17`, `:49`, `:125`). Held.
  - `internal/machinename/doc.go:9` ("the only production reader of the hostname and of the machine file") stays true. Held.
  - Sole-source scan: the new test file sits inside the package and is not production code. Held.
  - Order. Held.
- **Card 3 (machine-name-show)**
  - Seams: the new `deps` struct is private and no reflective test covers it. Held.
  - `RunCLI` and `RunCLIIn` go into both `seamsignature_test.go` lists. Held.
  - `excludedModules` gains `machine` in the same card that registers `machine` in `newRoot`, so the stale-entry check (`sandbox_coverage_test.go:130-133`) passes. Held.
  - `registration_test.go` walks `internal/` for `Command()`, and `newRoot` mounts it in this card. Held.
  - `TestMain` matches the TestMain form `PATTERN-test-isolation` requires (`gitkit.HermeticGitEnv`, then `tmuxkit.Main`). Held.
  - Tier: `TestName_Show` is tier 0 and spawns nothing. Held.
  - Doc text: the card writes `internal/machinecli/doc.go` itself and extends `docs/overview.md` and `pattern/PATTERN-cli-cobra.md`. Held.
  - Order: card 3 uses `FromThisHost`, which card 2 creates. Held.
- **Card 4 (machine-name-set)**
  - Order: card 4 uses `Replace` (card 1) and `deps`, `readCurrent`, `runName` (card 3). Held.
  - `TestHubOfDir` at tier 0: `lymcwd.ResolveWorktree` resolves the root in-process through `dotgit.FindRoot` (`lymcwd.go:152-162`), and `WriteShortname` spawns nothing. Held.
  - Test-file tiers: the card plans tier-0 tests (`TestNextFile`, `TestHubOfDir`, `TestName_Set`) and a tier-1 test (`TestName_SweepsHubs`) but never says they go in separate files. **At risk** (finding 2).
  - Doc text: the card lists `machinecli/doc.go` and `docs/overview.md`. Held.

**Findings**

1. **MEDIUM, consistency, card 1** (`internal/boardengine#SetOwners`, `#OwnerRequest`, `internal/boardengine/doc.go`)
   - The replace set writes over an owner that reads as another machine while `Takeover` is unset. Three existing doc sentences say that cannot happen:
     - the `Takeover` field doc: "without it such an entry is left alone and reported in OwnerWrite.Lost" (`ownerset.go:43`);
     - the `SetOwners` doc: "Without req.Takeover, an entry whose effective owner is another machine is neither written nor claimed" (`ownerset.go:72-73`);
     - `doc.go:183`: "Without Takeover, SetOwners leaves an entry whose effective owner is another machine untouched."
   - Card 1 lists all three symbols or files as Edit targets. Its Intent directs only the new `Replace` doc, the `writeOwners` doc, the `Recorded` doc, the doc.go commit line and the new replace-set paragraph. It never says to reword these three sentences, which stay false after the card.
   - Fix: the Intent names each sentence and its new wording, for example "without Takeover or a matching replace set".
2. **LOW, consistency, card 4** (`plan:internal/machinecli#TestName_SweepsHubs`)
   - `PATTERN-test-speed` requires every test function in one file to name the same tier, and `cmd/lym/tiertags_test.go` and `tierpurity_test.go` enforce it.
   - The card plans the tier-1 `TestName_SweepsHubs`, which uses `hubforge` and `gitkit`, beside three tier-0 tests, and names no test file for any of them. Putting `TestName_SweepsHubs` in the same file as `TestName_Set` fails the tier-purity scan.
   - Fix: the Intent says `TestName_SweepsHubs` goes in its own tier-1 test file.

Tool calls:
1. Searched for `//lym:seam` markers on `OwnerRequest`/`OwnerWrite`, doc text claiming only adopt takes over owners, and `machinename_test.go`'s tier. The manifest had no reflective-seam or doc-claim search.
2. Read `internal/machinename/doc.go` and other packages' takeover claims, to see whether cards 2–4 falsify them. Not in the manifest.
3. Searched `cmd/lym` scans for rules on `NewBolt`, `Pull` and `Takeover` callers, to see whether card 4's new callers break a pinned list. Not in the manifest.
4. Read the pinned-method list in `cmd/lym/gitrepoboundary_test.go` that the previous search found, to judge whether it covers card 4's callers. Not in the manifest.
5. Read the header of `cmd/lym/gitrepoboundary_test.go` to confirm it scans only `internal/gitrepo`. Not in the manifest.
