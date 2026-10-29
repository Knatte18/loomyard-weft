---
round: 1
focus:
  - "internal/websterengine/recoverbatch_test.go, `uncheckableRefusal`: settle whether every row reusing a setup/check pair now reads its record only from its own `recoverFixture`, so no closure variable is written from more than one parallel subtest."
  - "Rows built from `atStartSetup`, `committedSetup`, `longBesidePathSetup` and `noStartCheck`/`pauseRefusalCheck` wrappers: settle whether each wrapper still runs the base check against the fixture whose setup seeded the record, and whether `committedBaseSetup`'s discarded check leaves a path where `fx.Seeded` is unset or stale when a check reads it."
  - "The pointer-identity assertion `fx.Deps.State.Batches[1] != rec` and the Terminal/Status/StrandGUID assertions: settle whether the refusal rows keep every assertion they had before the diff, including that the refused spawn leaves the same record in place."
  - "`Seeded` on `recoverFixture` in recoverbatch_helper_test.go: settle whether the field's doc comment follows the repo's comment conventions and whether other tests building a `recoverFixture` are unaffected."
  - "Scope: settle that the diff stays inside the test files of internal/websterengine and adds no new test, as the decision record's constraints require."
---

Seed focus from the committed diff (one commit, two test files) read against the board entry and decision record D1.
