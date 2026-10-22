# Card 11 — shellguard-gate-test-case

**Edit:**
- `internal/shellguard#TestRows_PatternsDenyAndAllow`

**Intent:** D6: the shell guard lets the darn writer's tmux-tier reproduce command through every row, and a test pins it so a future guard pattern cannot silently take it away; `internal/shellguard`'s rows are unchanged.

`TestRows_PatternsDenyAndAllow` gains one case: the command `lyx gate test --wait --tier 2 github.com/Knatte18/loomyard/internal/x -- -run '^TestX$'`, the form `publishFailureSteps` renders for a failing tmux-tier test, checked against every row the test renders with a refuse list (the `rows` map), each through `payloadDenied`, and wanted allowed by each.
It runs as its own subtest loop over the rows beside the table loop, since the table's cases name one row each.

**ImpactSummary:** test-only; one allow case over the existing rows of `internal/shellguard`.
