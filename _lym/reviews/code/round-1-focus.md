---
round: 1
focus:
  - "internal/gatecli/scope.go `allScope`: does a scope-all gate key with only inert changes really run no tier at any gate key, Finalize included, while an explicit --all and a missing parent branch or unreadable merge base stay whole?"
  - "internal/gateslot/tiers.go `Config.Gate`: with no tiers declared, does it return a GateKey that every caller (shed build, fixer gate-call render, runTest) handles safely, and is the partial-tier case (tiers declared, no tier0) still refused?"
  - "internal/gatecli/run.go `runTest`: the early return 0 for --gate with no tiers; does it skip lints, reports or other gate duties a gate key still owes, and does its stderr message reach the caller?"
  - "contracts/stencils/darn/darn-fragment-make-change-gated.md: does the new exception name a permitted way to run one test class, and does it stay consistent with the surrounding no-`lym gate test` rule?"
  - "Issue #135 asks that a refused or neutered `dotnet test` say so with a message and nonzero exit; does the diff make that change anywhere, or only add the darn prompt wording, and is any gap a scope finding?"
  - "New tests in scopes_integration_test.go, test_integration_test.go and config_test.go: do they cover the tierless gate, the inert-only scope-all gate and the fall-back-to-whole branches, and do any duplicate one another?"
  - "Docs landing in the same diff: gatecli and gateslot `doc.go` against the changed behavior."
---

The decision record holds the board entry alone, with no amendments.
The two parts are issue #135 (silent `dotnet test`) and issue #133 (tierless hub, Finalize ignoring `inert_paths`).
