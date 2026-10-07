---
round: 2
focus:
  - "In `_lyx/plan/00-overview.md`'s `## verify:` section, settle whether every package a card targets has its `tmux`-tagged tests run, in particular `internal/loomcli` and `internal/webstercli`, whose `tmux`-tagged test files a `-tags integration` run does not compile."
  - "For each card that changes observable behavior of a package or CLI contract (the reed skip rule, `lyx loom status` waiting, webster's verify gate, landing's verify), settle whether the package's `doc.go`, `docs/overview.md` and the contract spec that describe that behavior are edited by a card in the same plan."
  - "For card 5 (reed resume dropping completed strands), settle whether the tests it plans reach `Resume`'s state removal, the dropped count and the CLI envelope, and whether its drop rule matches the decision record's wording with no added condition."
  - "For card 16 (batten re-spawn), settle whether the Intent states how each `DriverStrand` value, including retiring, routes the running-and-unconfirmed arm, and what bounds the re-spawn."
---

Targets the round after a first review that found one design and several doc-coverage and test-coverage gaps.
