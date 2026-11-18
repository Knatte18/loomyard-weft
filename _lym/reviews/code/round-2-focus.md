---
round: 2
focus:
  - "contracts/stencils/darn/darn-fragment-make-change-gated.md step 3: does the writer's measuring route name a gate key (`lym gate test --gate darn-write ...`) rather than a tier or target list, as PATTERN-test-speed requires, and do its no-test instructions agree with the gate's no-test report line in internal/gatecli/tierrun.go?"
  - "internal/gatecli/run.go runTest: in a hub whose `tiers` is empty, does a `--gate` call still run the key's declared lints and exit 1 on a lint failure, with a test row covering it in internal/gatecli/test_integration_test.go?"
  - "internal/gatecli/scope.go allScope: do the two fall-back-to-whole branches (no parent branch, unreadable merge base) have test rows in internal/gatecli/scopes_integration_test.go?"
  - "Do the prose sites that describe scope all or the no-gate writer rule (internal/gatecli/cli.go testCmd help, internal/gateslot/template.yaml comments, the darn fragment header, internal/darnengine/doc.go and spec.go, docs/dotnet-hub.md) agree with the inert-only empty-scope behavior and the writer's single measuring run?"
---

Targets the second round after the first round's fixes land.
