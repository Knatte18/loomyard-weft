# Card 2 toolfloor audit halted: allow-set-title is above tmux 3.4

Card 2 says to halt, naming any audited feature other than the two known forms whose confirmed version is above the floor.
tmux's upstream CHANGES file lists "Add an option allow-set-title to forbid applications from changing the pane title" under "CHANGES FROM 3.4 TO 3.5", so the option needs tmux 3.5, not a version at or below the floor.
`internal/reedengine/spawn.go` sets it with `set-option -p allow-set-title off` and only logs a Warn when the option is absent, so 3.4 degrades gracefully; the integration test already skips its check there.

Other tmux versions read from CHANGES: pane options (`set-option -p`) 3.0, `window-size latest` 3.1, `list-*` `-f` filter and `break-pane -b` 3.2, `#{E:}`/`#{T:}` and status-format list/range 2.9, user ranges 3.4.
The git versions on the card matched this host's RelNotes where the notes state them (show-current 2.22, cherry-pick --skip 2.23, path-format 2.31, merge-tree write-tree 2.38, worktree --orphan 2.42, cherry-pick --empty=drop 2.45); worktreepath, autoSetupRemote, maintenance.auto and merge-tree --no-messages are not stated in the notes text, so they were left as the card gives them.

Card 2 was not started in the tree; card 1 is committed.
Way forward: the plan decides whether `allow-set-title` is a third recorded form (found above the floor, tolerated by a Warn fallback) or is replaced.
