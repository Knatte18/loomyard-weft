---
round: 2
focus:
  - "internal/gatecli red-first tests: does a test assert each refusal `lym gate red-first` still emits through the shared diff-base and Go-module helpers (no parent branch to diff from, --base naming no commit, a hub declaring neither tier0 nor tier1), including the way forward each message names?"
  - "Tests deleted along with the removed verbs and report steps: did any of them hold the only assertion on behaviour that survives the diff, in packages other than the red-first verb as well?"
  - "_lym/landing/summary.md, sections Merge danger and Hub config: do the door call, the blast radius and the named manual steps match what the diff does, including how an older lym binary reads a gate.yaml entry this binary rewrites and what stored data the diff leaves on disk?"
  - "Prose describing gate-slot holders, cmd/testtiming and the removed re-run mark (pattern/PATTERN-test-speed.md, pattern/PATTERN-gate-slot.md, internal/gateslot/doc.go, internal/gatefixer/rowgate.go): does each statement agree with the code at HEAD?"
  - "internal/gateslot loading: is a hub gate.yaml that still sets a retired key accepted, and does a test pin that?"
---

The entries cover the areas where removal can leave surviving behaviour untested or prose stale.
