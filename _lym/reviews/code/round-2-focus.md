---
round: 2
focus:
  - "Once the scope is closed (a run at or past its first Code-Review round, including one sent back to Darn-Write or Webster), does every refusal, hold and answer route still name a way forward that the closed state accepts? Look at internal/loomshed/redfirstgate.go, internal/loomshed/darnwrite.go, internal/shedadapters/webster.go, internal/websterengine/recordbatch.go, internal/loomcli/wiring.go, internal/loomcli/parentreview.go, internal/landingshed/fixerrun.go and internal/shedadapters/bouncerdecision.go (including at the hard cap), and check that a test reaches each changed way forward."
  - "Does scopeClosedAt in internal/loomcli/decision.go keep the scope closed after the Code-Review run directory is archived (a new generation or an entry clear, including a generation that writes no round), and do tests drive it over a real directory?"
  - "Do the role files under contracts/orchrole and the darn, sampler and loom Code-Review stencils under contracts/stencils tell the parent or an agent anything that the closed scope refuses?"
  - "Do the tests fail if the scope_closed key is dropped from a shipped Code-Bouncer recipe row or from its pass-through in internal/shedrecipe/entries_bouncer.go?"
  - "Is the category given to plan_tier0_budget_sec in internal/gateslot/config.go the one PATTERN-config-layers requires for a key of that effect?"
---
