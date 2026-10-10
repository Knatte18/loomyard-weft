---
round: 1
focus:
  - "Card 7 (`07-hermetic-reed-servers.md`) against D7 and the Constraints in `_lyx/discussion/decision-record.md`: does each Decision and Constraint have a carrying card, and does the card's scope (the `PATTERN-test-isolation.md` edit, the kit `ConfigText` export, the attach pre-flight change) stay within what D7 licenses?"
  - "Card 7 attack surface: `-f /dev/null` on every spawn, the shell pins and the Windows empty-string bypass each remove or downgrade a guard or reach; does the `**Intent:**` state what each lets through and what bounds it?"
  - "Card 7 granularity: it creates three unexported helpers (`serverSpawnArgv`, `resolveShellPath`, `shellOptionArgvs`) plus a kit export and edits ten targets; are any helpers private supporting symbols that belong in another symbol's card, and is the card one independently reviewable unit?"
  - "Test economy across cards 6, 7 and 8: do `TestServerSpawnArgv`, `TestBoot_ReadsNoOperatorConfigAndStartsNonLoginPanes` and `KitConfigSharesReedsShellPins` duplicate behavior an existing test such as `TestUp_BootValidation`, `TestPinGeometryOptionsLocked` or `TestAttachArgv_ChainGate` already covers? Name the existing test."
  - "Member glyphs and the re-sign arrow on `Edit` cards: where an `Intent` changes a member's parameters, results or receiver (for example `pinGeometryOptionsLocked`, `AttachArgv`, `ensureServerAndSessionLocked`, and the card 4 and card 5 members), does the target carry the arrow, and do `Edit` cards with only Go file targets name specific symbols in `Intent` without member glyphs?"
  - "`ImpactSummary` on `Edit` and `Delete` cards: is each a one-line blast-radius conclusion rather than a restatement of `Intent`, and does every `Edit` card that needs one carry it?"
  - "Card order and the `## verify:` section in `00-overview.md`: does any card depend on a later one (cards 6 to 8 share `ensureServerAndSessionLocked` and `pinBootOptionsLocked`), and does `verify:` run the tests of every package a card targets, including `internal/testkit/tmuxkit`, `internal/logger`, `internal/reedcli` and `cmd/lyx`, in the template's section order?"
  - "Card 9 (`Prosa`, `docs/benchmarks/daemon-idle-cpu.md`) against D8: is the type correct for a manual measurement card, and is it ordered after cards 1 to 5 whose behavior it measures?"
  - "Shared Decisions in `00-overview.md`: does each stay consistent with the cards that cite it, notably the `internal/reedengine` test-budget rows 243, 244, 246 and 247 against the tests each card adds, and does any card introduce scope the decision record does not license?"
---

Seed focus for round 1.
Entries point at the largest and most cross-cutting card (7), the fidelity and attack-surface rubric items, and the cross-card ordering and budget claims.
No entry restates a check the gate already enforces.
