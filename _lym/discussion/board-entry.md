# Board entry `test-logs-leak-into-bolt`: A gate test writes trace logs into the live hub and Bolt commits them

trace-*.log from a run gate test landed in _board/.lym/logs and board sync committed them

## Seen

On 2026-10-27, at 07:37Z and 07:45Z, a test that a run's gate ran wrote `trace-*.log` files into the live hub's records worktree, under `_board/.lym/logs/`, rather than into a temporary directory of its own.
Bolt's next `board sync` committed them (`12046c7ab`, `589327e6d`) because `_board/.gitignore` does not exclude `.lym/`.
They were removed later by hand.

## Idea

- Find the test that resolves the hub's records directory from the environment or the working directory instead of a test-owned temporary hub, and point it at its own fixture.
- Make Bolt's sync stage only the board's own files, or have `_board/.gitignore` exclude `.lym/`, so a stray machine-local file never reaches Bolt.
