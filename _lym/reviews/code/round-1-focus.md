---
round: 1
focus:
  - "internal/runfsm/state.go and state_test.go: confirm `Order` has no caller left in any package or main, and that removing it with its test changes no behaviour a user can call."
  - "The deleted test helpers (battenshed, burlerengine, fabricengine, loomshed, orchcli, reedengine, reportengine, shedadapters, websterengine, cmd/lym): check each for a remaining caller in any build tag or platform (e.g. the linux-only test file), and that no deleted helper was reached through reflection, a registry or linkname."
  - "Scope: the answer key says code only tests reach is deleted unless it is a test helper, yet the diff removes test helpers. Settle whether each removed helper was uncalled by every test (so removal is licensed) or a helper that tests still need."
  - "internal/burlerengine/engine_test.go (largest deletion) and cmd/lym/testonlycode_linux_test.go: check that no remaining test loses coverage of a behaviour it names, and that no assertion in a surviving test was loosened."
  - "internal/shuttleengine/startup_test.go: the only edit with additions; check the doc comment rewrite follows the repository's comment conventions and the function's behaviour is unchanged."
  - "Commit messages and the change description (_lym/landing/summary.md, if present): check the claim about what was removed and kept matches the diff, and that the answer key's required report (removed lines per package, hits kept with reasons) is not contradicted."
---

Seed focus from the diff stat and the decision record, whose Goal is the board entry alone.
