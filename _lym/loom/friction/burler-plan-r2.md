# Burler fix, plan round 2: `lym shuttle await-forks` refused from the plan directory

In the middle of the round, the harness moved the session's working directory to the weft's `_lym/plan`.
`lym shuttle await-forks` then failed with "cwd is outside the recorded fabric anchor subtree", because it requires the anchored directory itself.
The self-check fell back to taking each fork's result as it arrived, across four turns.
