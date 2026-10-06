# Run tabs overflow a narrow terminal once several runs are open

`reed-tabs-narrow` · Note · enhancement, reed

## Seen 2026-10-06

With five runs and the prime open, the session row of the status line (the run tabs from `reed-status-tabs`) is wider than a narrow terminal, and the rightmost runs fall off the edge with no sign that they exist.
The row prints every session's full name with padding, and run slugs such as `test-suite-prune-core` are long.

## Options

1. Current session full, others truncated, overflow scrolls (recommended): the session the client is in shows its full name, every other session is cut to about 12 characters (`#{=12:session_name}`), and the row uses tmux's list mode (`list=on`, `list=focus`, left and right markers), so a row that still overflows scrolls with `<` and `>` markers and keeps the current session visible.
2. Index only for other sessions, the current one by full name: the most compact, but the operator has to remember which number is which run.
3. List-mode scrolling alone, full names kept: the least change, with the fewest sessions visible at once.
4. Wrap the session row onto further status lines: costs terminal height, and tmux has no automatic wrap, so reed would compute the split itself.

The run tabs' design is in the note `reed-status-tabs`.
