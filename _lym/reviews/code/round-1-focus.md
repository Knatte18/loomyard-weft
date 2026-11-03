---
round: 1
focus:
  - "Scope against the Goal: the diff is a merge of origin/main into the beam. Check that commits `d026a9a8` and `c896481c` (hibernate carrier, drift dir base) are conflict or build reconciliation and not new behaviour, per the Bounds."
  - "internal/hibernateengine and internal/hibernatecli (carrier.go, verbs.go, watch.go, request.go, deps.go, spawn.go): do main's hibernate changes and the beam's request/carrier design both still hold after resolution? Look for dropped or doubled logic and unhandled failure paths."
  - "internal/driftengine (config.go, seats.go, steprun.go, compare.go) and internal/loomcli/driftwiring.go: does the merged code keep what the beam's Drift/Codex work did, while using main's changed APIs correctly? Check the missing-Drift-dir failure path."
  - "internal/gateslot/config.go, internal/fabriccli (anchor.go, anchordeps.go) and internal/lymdirs/dirs.go: are main's changes under the beam's new code consistent, and does each new behaviour have a test?"
  - "Repository rules: `pattern/PATTERN-daemon-wakeups.md` and the doc.go files changed in the range stay in step with the code they describe; new or changed doc comments follow the repo's conventions."
  - "Test economy: new tests in carrier_test.go, hibernatedaemon_test.go, config_test.go and capture_test.go that duplicate an existing test's coverage."
  - "Commit messages in the range: any `Pattern-Departure:` trailer names a known entry that forbids the way taken."
---

Seed focus for a merge-reconciliation review. No decision-record sections beyond `## Goal` are filled, so the Goal and its Bounds are the measuring stick. The Sketch for P2 is the authority on what the beam's code must keep doing.
