---
round: 1
focus:
  - "Decision-record fidelity, acceptance criteria by acceptance criterion: for each criterion in _lyx/discussion/decision-record.md, name the card and the test that carries it. Settle in particular whether three are carried: a failed llm-arm removal still reaches the driver spawn at the start level (Card 1 pins only the helper), a failed reed Up fails start on either arm and fails step with the bootstrap refusal kind, and an llm-seeded start over a strand-free session adds no status strand while the driver strand still spawns."
  - "Card 3's smoke_test.go edit: TestSmokeBootstrap_BringsUpSessionStrandAndDriver and TestSmokeBootstrap_SecondInvocationDoesNotSpawnASecondDriver run on an unseeded fixture, which resolveSeedDriver seeds llm. Settle whether the card's premise holds that they assert go-driver properties, and whether seeding go before the first start leaves the llm path's own start-level coverage intact or silently narrows it."
  - "Card 2's TestSmokeStep_AddsNoStatusStrandOnEitherDriver crosses go- and llm-seeded fixtures with lyx loom step and lyx shed step. Settle whether each of the four cases can reach and pass its assertions on the providerless shuttle fixture (compare smoke_bootstrapwiring_test.go, which already steps fixtures), and whether the file doc comment's zero-real-LLM-subprocess claim holds."
  - "Card 3's dropping of c.reed.Up() from ensureStatusStrand: settle whether every remaining caller brings reed up first once Card 2 has moved step off it, and whether the card's prose sweep reaches every comment that ties start to the status strand unconditionally. Candidates the card does not list: the header comment of integration_driverbootstrap_test.go (line 22 says start spawns the status strand's watcher pane) and start.go's child-died comment near line 234. Decide whether each is on the go-only path or now stale."
---

The plan is three cards over `internal/loomcli`, with comment-only edits in `internal/shedcli`.
The verify section runs both packages plain and under `-tags integration`, and vets the `smoke` tier.
The driver-choice tripwire test lives in `internal/loomcli/bootstrap_test.go`, so that criterion is inside verify's reach and is not a focus.
Card 1 is a standalone helper with no caller until Card 3.
Under the granularity rule an independently testable symbol gets its own card, so the split is not a focus.
