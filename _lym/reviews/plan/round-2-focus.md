---
round: 2
focus:
  - "Cards 12 and 13 (`_lym/plan/12-run-loop.md`, `_lym/plan/13-run-reentry-and-resume.md`): does every function the cards call from `weaverengine.Run` sit in some card's target list, and is each prescribed refusal or Stuck branch reached by a named test row?"
  - "Card 13 re-entry path: when a stopped batch is recorded done with no spawn, do the card and batch gates run at HEAD first, and is any field recording the last stop cleared once the re-entry succeeds?"
  - "Card 2 (`_lym/plan/02-weaver-config.md`) and the weaver.yaml keys: is any key that names the gate behind Done classified so a local override can weaken it, per `PATTERN-config-layers`?"
  - "Cards 10, 11 and 21 around Weaver's state file: can a save from `Run`'s in-memory State overwrite a field another process wrote under the state-mutation lease, such as delivered decision headings?"
  - "Card 14 (`_lym/plan/14-weaver-archive-and-reset.md`) reset: does each branch that commits or archives state state its bound in `Intent`, and does a test reach the refusal?"
  - "Cards 9, 10 and 11 tests: does any card plan a per-helper test for an unexported function whose behavior a `TestRun` row at the exported surface could hold, per `PATTERN-test-speed`?"
  - "Every card adding a production function: does a `main` package reach it, or is it only called from a test-only allowlisted oracle, per `PATTERN-test-only-code`?"
---

Focus entries point at areas, not at settled findings.
