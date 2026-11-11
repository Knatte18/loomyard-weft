---
round: 2
focus:
  - "internal/shedengine step and appendOrFold tests: does every entry path that sets StartedAt (done, stuck, reroute, reroute-capped) and the fold that keeps the first call's StartedAt have an assertion, so dropping the field fails a test?"
  - "internal/shedverbs idle gap reporting: do the IdleGap comment and package doc state what happens to idle time inside a folded budget-exempt stuck entry, and does the code match that statement?"
  - "internal/shuttleengine capped skill-load hold: is Result.APIError, the cap-naming parent notice text, and the all-outputs-exist return covered by a test, and do the OutcomeAPIError comment in engine.go and the api-error text in docs/overview.md name the capped hold?"
  - "internal/shedverbs status envelope: is a non-empty idle_gaps with its JSON field names asserted in a status command test?"
---

Targets the areas where round 1 found gaps; the reviewer judges the whole diff afresh.
