---
round: 1
focus:
  - "D7 `Outside a hub it runs unslotted`: can an agent put `lyx gate test` outside a hub (via `-C` or cwd) and thereby skip the slot, and does the Decision state what bounds that?"
  - "D11 and D12: the Publish failure record lives under the worktree's `.lyx/`, which the pair controls; what can a pair skip by deleting or editing the record, and is that bound stated beside the tier 3 and Burler-range guards it feeds?"
  - "D4 against the Constraints on `verifyrun` and the plan-writer note: D4 says a shell command run through `verifyrun` gets the `GOFLAGS` cap, the Constraint says the slot is acquired in callers or through a told seam; do the two agree on where acquisition and the environment composition happen?"
  - "D1, D2 and D7 together: a card or batch gate step is `lyx gate test`, which exits with a distinct slot-busy code; is it settled which party (fork, Master, or Go) treats that code as non-failure, and do D2's `FAILED` report rule and the self-fix cap agree with D8?"
  - "D10 retiring per-pair `landing` overrides: the Bound names the new route for a landing change; is every guard that `landing` carried (for example `require_pr_to_base`, `squash`) still enforced at one site, and does the migration for a started pair in Open risks have a matching Decision or acceptance criterion?"
  - "Relocation check: do `Auto-mode assumptions` and `Open risks` repeat content already in a Decision (defaults in D5, the tier 3 condition in D11, raw package-scoped runs in D7)? Before any move, confirm each carries no requirement missing from Decisions or Constraints."
  - "Acceptance criteria against Decisions: is each Decision with an observable behavior covered by a criterion (D6 status output, D9 refusal, D13 lint rules, D14), and does any criterion state something no Decision introduces?"
---

Seed focus only; the reviewer still reads both artifacts in full.
Entries avoid the rubric's `Do not flag` items: no entry asks for signature enumeration, call-site lists, or rejected alternatives in `decision-record.md`.
