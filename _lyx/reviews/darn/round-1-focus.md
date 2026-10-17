---
round: 1
focus:
  - "internal/gatecli/run.go: does the busy path print an ok envelope with slot_free false and the holders, exit SlotBusyExit and run no go test, and does Command pass shuttleengine.DefaultAwaitCap itself rather than a copy?"
  - "Retirement of cli_wait_sec (gateslot config, Limits, template.yaml, the gate spec's RetiredKeys as CategoryCrossPair): is any reader, test fixture, doc or hub-geometry helper still referencing the key, and does a hub file that still sets it keep loading?"
  - "internal/shellguard/rows.go and the webster implementer and verify-fix stencils: do they tell agents to re-run on the no-slot-free answer in the await help's words, and does the diff drop, weaken or contradict any other stencil rule?"
  - "internal/shuttleengine/claudeengine/testdata/settings-golden.json: is the 256-line diff only the gate rule wording, or does it carry unrelated churn?"
  - "cmd/testtiming and internal/shedverbs/loopwait_test.go: does the testtiming slot wait still end on a stuck holder, and does the shedverbs grace change match what the summary claims without scope drift beyond the board entry?"
  - "Docs and test economy: do doc.go files and docs/overview.md match the new behavior, and does each added behavior (slot_free false envelope, exit 75, retired key) have coverage without duplicating an existing test?"
---

Focus is set from the board entry's operator decision and the change description's claims; each entry names a place to check the diff against them.
