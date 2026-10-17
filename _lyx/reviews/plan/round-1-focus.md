---
round: 1
focus:
  - "Fidelity: check that every Decision (D1 to D9) and every Scope In bullet of the decision record is carried by some card, in particular the stencil and package-doc updates, the CLI changes listed under Scope (`inbox` flag, `log`, `orch status` help, `tender revive` report), and the two PATTERN changes."
  - "Card order: cards 3, 5, 6, 7 and 9 to 11 build on creel and client changes from earlier cards; settle whether any card uses a symbol that only a later card creates."
  - "Attack-surface: the command inject (program sender only, orch strand only), the Go-only withdraw verb, the hold on an orch strand and the outside-pane refusal code each need a stated bound in the introducing card's Intent."
  - "Reachability: each refusal the cards prescribe (command inject from a non-program sender, withdraw from the CLI, hold from a non-program sender, failed first withdraw in `lyx orch start`) names an input and a test that reach it, and no upstream check rejects that input first."
  - "Verify coverage: every package a card targets, including `internal/creelcli`, `internal/shedrun`, `internal/battenshed` and any package card 8 or card 6 touches in a caller, has its tests run by the plan's `## verify:` section."
  - "Test economy and granularity: cards 6 and 8 move many callers; check that they plan no per-caller or per-symbol tests where one public-surface test covers the behavior, and that no symbol on a public surface is buried in another card."
  - "Edit cards: member glyphs instead of bare Go file paths where the Intent names symbols, and the re-sign arrow where the Intent changes a signature (for example `NotifyPrime`, the `Delivery` seam, `Event`, the client call)."
---

Seed focus from the plan overview and the decision record head.
No prior round exists; the entries name where to look, not verdicts.
