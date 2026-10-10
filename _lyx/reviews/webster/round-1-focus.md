---
round: 1
focus:
  - "Cards 14, 15, 16: does `websterengine` still import no `fabricengine`, with the read-only classifier reaching the engine only as an injected `func(cmd string) bool` supplied by `webstercli`, per the `classifier-by-injection` shared decision?"
  - "Cards 3, 4, 5: does every rebaseline refusal name the step its caller passes, so that no auto-path text tells an agent to run a verb by hand, and does the auto path accept exactly what `--card` accepts?"
  - "Cards 24, 25: is the recovery count and the cap at two enforced on every path that spawns a recovery, and is the retry of a dead recovery that committed reachable only when it committed, including on resume?"
  - "Cards 18, 20, 17: does the sandboxed-role guard refuse every mutating fabric verb and no reader verb, and does each refusal's way forward name the orch?"
  - "Cards 6, 7: does the done-card-edited detection compare against the batch's start and so flag only done cards edited since their batch began, with the plan gate refusing exactly those?"
  - "Cards 10, 11, 12: is the single verify chain splitter used by both verify checks, splitting at `||` and `|` and not inside quoted text, and does each new refusal carry its `contracts/specs/refusal-spec.md` row and `doc.go` passage in the same commit?"
  - "Cards 21, 22, 23: does the shell-wait rule treat both shell signals alike with every ended shell recorded, and does the Publish-failure re-run name the top-level test only?"
  - "Test economy: for each new test across the cards, does an existing test already cover its behavior, and does any added behavior lack coverage; also check the package rows in `cmd/lyx/testdata/test-budget.yaml`."
---

Seed focus for round 1, drawn from the plan's shared decisions and the cards whose behavior is easiest to get subtly wrong.
Nothing here limits the review to these entries.
