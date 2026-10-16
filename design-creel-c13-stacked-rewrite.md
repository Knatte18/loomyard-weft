# Rewrite C13's stacked half for window-per-strand

`creel-c13-stacked-rewrite` · Note · enhancement, creel

The board entry `reed-add-parentless-placement` removes the stacked layout and `collapsed_rows`, refuses `below-parent` and `top`, and moves a strand pane left in the shell window into its own window.
C13 in `tools/sandbox/SANDBOX-CREEL-SUITE.md` then needs its stacked 3-row half rewritten against the window-per-strand layout.
Its hand-added half (a plain `lyx reed add --cmd claude` strand receiving its reminder) stays as is.
