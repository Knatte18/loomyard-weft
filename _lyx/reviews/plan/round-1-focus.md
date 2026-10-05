---
round: 1
focus:
  - "Card 19 (reed-socket-removal) and card 17 (tmuxkit sweep): does each `**Intent:**` state the bound D9 sets on the removal (only the killed key's socket, only inside that teardown, only when it is a socket and no server answers, never a glob, identity re-check)?"
  - "Card 15 (llm-spawn-guard) and card 11 (llmkit): does `**Intent:**` state what the guard cannot see (a tier-3 test driving `lyx` into an LLM spawn without the helper) and the `exec`-identifier check on files under `llmkit`, per D7?"
  - "Cards 13 and 14 against the verify section: does every package whose files move to `tmux` or `llm` get run by `go test -tags tmux` or compiled by `go vet -tags llm`, and does any card leave a file under a tag neither a run nor a compile check covers?"
  - "Card 10 against D8: are all retired IDs (`PATTERN-test-tier-purity`, `-time-seam`, `-parallel`, `-hermetic-git-tests`, `-tmux-test-isolation`) renamed in code, stencils, specs and docs, are the background files deleted or created as D8 says, and do later cards place their rules in the right surviving entry?"
  - "Cards 3-9 against D4 and D5: does any card add, remove or retag a test file before card 9, and does any seam use a package-level `var` rather than a per-instance field or parameter?"
  - "Cards 1, 20 and 21 against D1-D3: does card 20 carry the D2 rules (subset test, covering tests by overlap, greedy removable set, separate no-coverage list for spawning tests with over-approximating scan, default tags), and does card 20 plan tests that duplicate coverage already given by card 1's tests?"
  - "Granularity across cards 3-5, 17-19 and 12-16: is any card split per symbol where one reviewable unit would do, or is any card a bundle of independently reviewable units?"
  - "Decision-record coverage: is every Decision D1-D10 and the after-Discussion cross-tier helper constraint carried by some card, and does any card's scope go beyond what the record licenses (for example gate selection or `cardGateCommand`)?"
---

Seed focus drawn from the overview and the decision record; each entry was checked against the `Do not flag` list and names a question, not a verdict.
