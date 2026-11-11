---
round: 1
focus:
  - "internal/boardengine/ownerset.go: does the replace-set selection run after the write's own pull, touch only open beam projects and the prime whose owner is in the set, leave done entries and other owners alone, and ignore an empty former name?"
  - "internal/machinecli/set.go: does the order follow D5 (pull, read owners, hostname name as own, collision refusal, file write, sweep), including the already-set re-run, the bad-file repair and the undecodable-file replacement?"
  - "internal/machinecli/set.go: is the collision check exact: owners not among current, former and hostname-rule names refused, a pull-failed or fetch-skipped hub checked locally and named as such, and the last valid name of a hand-broken file kept in names?"
  - "internal/machinecli/set.go and hubs.go: does the sweep cover every hubregistry.Live hub plus the working directory's hub once, report a failed write, Lost entries and a nothing-to-sweep hub per D6, report a pushed-only re-run via the pre-write HasUnpushed read, and exit non-zero on failure?"
  - "internal/machinecli output: does it name the D6 window recoveries and advise lifting a pause only after the hub's sweep reports done (acceptance criterion marked Review)?"
  - "internal/machinecli/current.go and cli.go: no-argument show prints name, source, file path and former names, writes nothing, and prints the file's error and path over a bad file; no resolvable config directory or relative variable refuses a write naming the variables."
  - "internal/machinename/machinename.go, cmd/lym/main.go, tests: do the tests reach the machine file and hubs only through the deps seams (D8), and are the five tests the redundancy report lists judged against the test-economy bullets?"
  - "pattern/PATTERN-owner-machine.md, pattern/PATTERN-cli-cobra.md, docs/overview.md, boardengine/doc.go: does the declared PATTERN-owner-machine change land with its declared text, and do docs land for the new module?"
---

Seed focus drawn from the decision record's D5, D6 and D8 and its acceptance criteria.
