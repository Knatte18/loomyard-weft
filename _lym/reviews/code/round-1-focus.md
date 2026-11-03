---
round: 1
focus:
  - "internal/seatengine/table.go `Table.CheckMarkers`: does it report exactly the markers `Table.Compose` would leave unfilled (stencil and block markers; table, seat, optional and reserved values), with no false positives on markers filled only at compose time and no misses for block markers?"
  - "internal/preflightshed/preflight.go: does Preflight compose the Darn writer's table the same way the real Darn-Write seat run does (same config values, same stencil source), so the check cannot pass while the run fails or fail while the run succeeds? Is the refusal's message carrying seat name, markers and way forward, and are failure paths of composing the table handled?"
  - "internal/seatengine/prompt.go, engine.go, resume.go: does every seat error now name the seat by its strand role, and is the lead's role prefix used consistently on all error paths, including the resume path, with no remaining empty-name `seat \"\"` message?"
  - "Scope against the board entry's three asks (find why `subagent_model` was unfilled, reject unfillable seats before the run with the seat name, fix the empty seat name) and the operator note that the marker stays: does the diff drop the marker or hardcode a model anywhere, and do the edits in internal/shedrecipe, internal/loomcli/darnwiring.go and internal/testkit/envkit go beyond what those asks license?"
  - "Claims in `_lym/landing/summary.md` and the two commit messages (stale hub stencils refreshed by a newer binary mid-run as the cause): does the diff bear them out?"
  - "Test economy in internal/seatengine/table_test.go, prompt_test.go, internal/preflightshed/preflight_test.go and internal/loomcli/wiring_*_test.go: any new test duplicating an existing one's coverage, and any added behavior (CheckMarkers branches, Preflight refusal, role-named errors) with no coverage."
---

Seed focus from the decision record's board entry and the diff stat over `486bbf44..HEAD`; no round has been judged.
