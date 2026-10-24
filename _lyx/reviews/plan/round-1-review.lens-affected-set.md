```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: what each card's change forces outside the card, and which cards cover it.

## Per change: the set found and the cards covering it

- **Card 23, the batten, adopt and cleanup guard and its refusal text.**
  - Guard sites, all covered by card 23: `armAt` (arm.go:366), `cancelCmd` (cancelcmd.go:104), `lockStore` (lock.go:74), `Cleanup` (cleanup.go:96), `overviewCmd` (overview.go:468), `adoptcli` `Command` (cli.go:63), `cleanupcli` `Command` (cli.go:74).
  - Shed seed (shedcli/table.go:62) keeps `RefuseUnlessPrime`, as the card says.
  - Tests that assert the refusal text:
    - covered: `TestCleanupVerb_…` (cleanup_integration_test.go:158), `TestArmAt_StartGuardRefusesBeforeAnyWrite`, `TestCancelCmd_RefusesBeforeAnything`;
    - **missed:** `stepNonPrimeRefusal` (lifecycle_integration_test.go:1482) and `stepRecordsPrimeRefusal` (:1509), both run from `TestBattenIntegration_Rows` (:326-327).
  - Docs: adopt and cleanup `doc.go` and `docs/overview.md` are covered.
- **Card 19, orch's `refuseNonPrime` widened and reworded.**
  - Covered: `TestRefuseNonPrime` (orchcli/cli_test.go:148-155) and `TestStart_Refusals`.
  - **Missed:** `refusalCase` (orchcli/refusal_integration_test.go:118-128), used by `TestOrchIntegration_Refusals` (:23). It asserts the text `prime worktree only`.
- **Cards 15 and 16, `fabricengine.ParentSlug` replaced and then deleted.** Every caller is covered by card 15:
  - boardcli/cli.go:141 (`Command`), pairteardown/originseams.go:45 (`HubSeamOptions`), loomcli/landingdeps.go:516 (`parentOwner`), adoptcli/deps.go:60 (`hubDepsAs`), battencli/cancelcmd.go:187 (`hubCancelDeps`), battencli/owner.go:45 (`hubOwnerSeams`), tendercli/deps.go:157 (`HubDeps`).
  - pairteardown/doc.go:128 is covered by card 15.
  - The fabricengine doc mention is covered by card 16.
- **Card 25, `orchStrandRecorded` deleted.** All four callers are covered by card 25: wire.go:549 (`noticesReachDriversParent`), :576 (`rowNotices`), :906 (`battenCLI.wire`), drive.go:199 (`driverDepsFor`).
- **Card 21, tender's four orch seams replaced by `Orchs`.**
  - Declarer `Deps` (report.go:83-90), user `orchCase` (pause.go:434), user `reviveOrch` (resume.go:483), filler `HubDeps` (tendercli/deps.go:117-175) and test filler `env.deps` (pause_test.go:310-341) are all covered by card 21.
  - The `ReviveReport` `orch` key (resume.go:391) has no reader in `plugins/`, `contracts/` or `docs/`.
- **Card 3, `AddOptions` alias becomes a struct.** All uses go through composite literals, plus one pass to `CommitRecordsPaths` (add.go:454), which card 3 covers.
  - Composite literals: fabriccli/fabric.go:1193, adoptcli/deps.go:71, battencli/wire.go:851, hubforge/hub.go:426.
- **Card 6, `runAdd` and `addOptionsFromEnv` re-signed.** The one caller (fabric.go:529) is in `runAdd`, which card 6 covers.
- **Card 9, the new `kind` value.** Every kind reader is covered by card 9 or card 11:
  - `ownersOf` (owners.go:59), `ListTasksBrief` (store.go:665), `Store.Promote` (store.go:700), `writeOwners` (ownerset.go:164), `handleLetter` (handles.go:63), `validateLabels` (labels.go:31-37), `normalizeTask` (task.go:258).
  - `migrateEntries` (migrate.go:87-120) keeps any stored kind, so it is not affected.

## Findings

1. **MEDIUM, consistency, card 23 (`23-batten-guard.md`).** Card 23 replaces the prime-only batten refusal with `ResolveBattenPrime`'s new text, which names the nearest beam or the prime. Two integration steps that assert the old refusal are not among its targets:
   - `internal/battencli#stepNonPrimeRefusal` asserts that the refusal names the task worktree and the prime (lifecycle_integration_test.go:1482-1502).
   - `internal/battencli#stepRecordsPrimeRefusal` asserts the wording `prime worktree only` for the records-sibling refusal, which `RefuseUnlessPrime` wraps today (lifecycle_integration_test.go:1509-1525; refusal.go:43).

   The writer stencil's rule 4 puts every test that asserts a reworded message on the rewording card.
   Fix: add both glyphs under card 23's `**Edit:**` and state the asserted text in the Intent.
   The glyph spellings are not yet confirmed with quarry.
2. **MEDIUM, consistency, card 19 (`19-orch-in-beam.md`).** Card 19 rewords orch's `refuseNonPrime` to admit a beam and to name `lyx beam create` and `lyx beam promote`. The integration helper `internal/orchcli#refusalCase`, which `TestOrchIntegration_Refusals` drives from a non-prime worktree, asserts `prime worktree only` (orchcli/refusal_integration_test.go:118-128) and is not on the card.
   Fix: add `internal/orchcli#refusalCase` to card 19's `**Edit:**`, or have the Intent keep that phrase in the new text.

## Tool calls

1. Read battencli/refusal.go and orchcli/refusal.go, and grepped tests: the exact old refusal texts and their asserting tests were not in context.
2. Grepped for tests asserting `prime worktree only`: to find every test that asserts it.
3. Read lifecycle_integration_test.go:1500-1530 and orchcli/refusal_integration_test.go:105-130: the asserting helpers' bodies were not in context.
4. Read lifecycle_integration_test.go:1470-1499, the step registrations and the adopt and cleanup assertions: the non-prime step and its runner were not in context.
5. Grepped where tender's orch seams are used and filled, and where the revive `orch` key is read: card 21's affected set was not in context.
6. Grepped the seam uses in report.go, pause_test.go and tendercli/deps.go: to map them to members, which context lacked.
