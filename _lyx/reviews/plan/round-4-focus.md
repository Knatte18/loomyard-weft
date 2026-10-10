---
round: 4
focus:
  - "Card 10's socket check: does its rule under `allowed` false fail any tier-1 or tier-2 package whose tests run a tmux client probe (`has-session`, `kill-session`, `list-sessions`) through production such as `pairteardown`, with no server ever started, and does every package the rule fails have a card in scope and a Verify line that runs it?"
  - "Every path that runs a card's or the plan's verify command as a shell command (`lyx gate test`, `verifytree.Verify`, webster's card-verify rerun): does each strip the prebuilt-binary variable, and is the card 13 bound that a verify command never sees it true on all of them?"
  - "Reed servers a registered test boots again after its own `down` or `kill-server`: does the plan close the config-less boot test-side or cite a decision-record entry that accepts it, so D4's never-read-the-operator's-config rule is neither narrowed nor silently broken?"
  - "Card 31's tmux-tier evaluation of build constraints: does a file constrained by a platform tag together with `integration` count as tier 2 on every platform the module builds on, not only on the host's GOOS?"
---

Round 3 closed with a BLOCKING design finding, so round 4 re-checks the cards those findings touched after the fixes.
