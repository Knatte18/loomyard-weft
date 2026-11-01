# Delete the one-shot boltrename program

`tools/boltrename` moved an existing hub's Bolt checkout from `_board` to `_bolt`, rewriting the git worktree links, process records and workspace paths that named the old directory.
Every hub that needed it has moved: Neve is migrated, and OSL-1046 is re-cloned rather than migrated.
The program has no hub left to run on, so it is deleted together with its tests.

No other file in the repository names the program or the `_board` directory, so nothing else changes.
