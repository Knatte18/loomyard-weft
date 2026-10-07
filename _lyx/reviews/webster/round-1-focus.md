---
round: 1
focus:
  - "Cards 1-3 and 12-13: does every typed text-plus-submit sequence go through the paced, confirmed path, with no remaining raw text+Enter send in `internal/shuttleengine/claudeengine` or the orch watcher, and does `sendVerified` stop after its bound of extra Enters?"
  - "Cards 10-13 and `PATTERN-orch-pane-single-writer`: does all orch pane typing, including `/reload-plugins` and skill retyping, stay in the watcher and idle-gated, and does only a `/clear` reload retype skills?"
  - "Cards 5-7: does each strand removal cover every terminal outcome and every live recovery strand the card names, and does it leave a strand a different run still needs?"
  - "Cards 8-9: does a step that errors after persisting still record the handoff voucher, and does the crash-resume note name the lock and voucher readings and list all history as the card says?"
  - "Cards 14-16: does any path in `lyx loom start` or `lyx orch start` still attach or switch a tmux client, and is the wait for a stopping watcher bounded with the stopping state written before the signal's effect?"
  - "Cards 17-18: does every Burler round gate its review on `ParseReview` with a re-prompt, and does the `validate-review` verb use the same parse the gate does?"
  - "Test economy across all cards: for each new test, check whether the pruned suites' existing test, scenario or table row already covers it (cards 4, 6, 7, 14, 15, 16, 17 named in the plan's pruned-suites decision), and whether each added behavior has a regression test entering through the CLI wiring."
  - "Per-card mechanical checks: confirm each card's groups' type-specific checks (`Rename`, `Delete` and any others) ran against every target of the group, not only the first."
  - "Doc comments: new and changed comments follow the repository's `CLAUDE.md` and `scribe` conventions (semantic line breaks, no pinned tallies), and docs land in the same commit where a card changes observable CLI behavior or adds a module or invariant."
---

Seed focus drawn from the plan's Card Index and Shared Decisions.
The review range is `git diff $(git merge-base main HEAD)..HEAD`, since no rework round exists.
