---
round: 1
focus:
  - "D3 lyxbin exemption: the Bound says lyxbin may spawn `go build` of `./cmd/lyx` and nothing else, while the scan exempts lyxbin from the spawn-import rule by name. Settle whether the Decision states what keeps lyxbin to that one spawn, or whether the bound is enforced only by the Tier Purity token."
  - "D3 Testkit Invariant rules versus its scan: the scan covers the importer, `*cli` and spawn-import rules, but not 'starts no process, tmux server or agent' or 'asserts nothing beyond setup Fatalf and the Call/RequireOutcome check'. Settle whether the Decision says which rules are scan-enforced and which are review discipline, the way D6 does for its clauses."
  - "Test and assertion removals (D7's CLI remedy-string assertions, D8's replaced nil-check tests, #325 §5's duplicate-behaviour deletions) against D9's 'every assertion they make survives': settle whether each removal states what it can now let through and whether D9's protected list bounds it, and whether D7 and D9 conflict for any CLI test that is itself protected."
  - "D1's cycle-forced local copies and D4's move of untagged gitkit callers to integration-tagged files are exceptions to the sharing rule and a shift of tests out of Tier 1: settle whether each Decision bounds its exception (a closed set of cases, and what coverage the default `go test ./...` run loses)."
  - "Scope-to-acceptance mapping: walk each Scope bullet and settle whether a Decision or an acceptance criterion makes it checkable, e.g. #325 §6's reedcli reap-cycle table and shared `*lyxcwd.Location` builder, and #324 §6's log capture and `slices` replacements. This is item-level coverage, not call-site enumeration."
  - "Acceptance criteria list structure: the Tier Purity / Hermetic Env sections bullet and the fabricengine `fabricFixture` bullet appear run together on one line; settle whether each criterion is a separate, independently checkable item."
  - "Relocation and exclusion in decision-record.md: D13's 'Kept unchanged, with the reason' list and its 'Notes for test-suite-derived-guards', D4's narrative of the two scans' current disagreement, and the survey tallies in Auto-mode assumptions and D4/D5 rationale. Apply the completeness-before-leanness test first: settle which parts carry a constraint the plan writer needs, and whether the rest belongs in support-log.md or outside Discussion as Plan-time facts the writer's stencil says not to gather."
  - "Reverse completeness: Plan-Write never reads support-log.md. Settle whether any requirement or constraint lives only there, e.g. interview item 1's 'pattern-init later moves CONSTRAINTS.md, so the Testkit Invariant lands there first'."
---

The decision record introduces several escape hatches and guard changes (the `lyxbin` spawn exemption, `intentionallyNil`, cycle-forced local copies, deleted or relocated tests), so the attack-surface question carries most of the focus.
D8's `intentionallyNil` already states a per-entry bound, so it is not singled out.
Notes for the plan writer and the absence of rejected alternatives in the record are excluded per the rubric's Do-not-flag list.
