---
round: 3
focus:
  - "Card 14 (`_lym/plan/14-loom-harden-wiring.md`) `hardenLiveRun` and card 6 (`_lym/plan/06-live-gate.md`) `Run`: settle whether the way `hardenLiveRun` derives passed, output, failed and `Refused` from the spawned `lym gate test --wait` call matches what `internal/gatecli` actually emits (summary and failure report on stderr, exit code for the verdict, a JSON error envelope on stdout only on a refusal), so that an ordinary green or red run yields a live report rather than an error."
  - "Cards 6 and 14 together: settle whether the error `Run` wraps is reserved for a spawn or wait failure, and whether every branch card 6 prescribes for a refused call, a failed run and a passed run is reached by a named input and a named test row in `TestLiveGate` or `TestHardenLiveArgs`."
  - "Card 14's `TestHardenLiveArgs` rows: settle whether each row's input is parsed the way the card's own `hardenLiveRun` text says, including build output on stdout and a JSON line on stdout that lacks `ok` false or an error."
---

Round 2 left one BLOCKING design finding on the live-run reading path; round 3 checks the rewritten text for it and for fresh defects it introduced.
