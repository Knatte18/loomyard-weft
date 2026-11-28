# Burler plan fix, round 3: the plan validator stalled under machine load

The first `lym loom validate-plan` run passed in seconds.
The second, after the self-check edits, ran past 600 seconds and then failed three runs capped at 120 to 300 seconds, so I bisected the Uses lines through `--overlay`.
None of the edits was the cause: the machine's load average stood near 30, and one uncapped foreground run passed.
The validator prints nothing while it runs, so I could not tell a slow run from a hung one without reading `/proc`.
