After PR #280 changed `bouncer-template-seed` and `bouncer-template-judge` and was merged, every `lyx` command in the hub warned:

```
stencilstore: board copy has drifted from worktree source; see lyx stencil promote
```

`lyx stencil promote` copies a board copy back into `contracts/stencils/`. Here that would have reverted #280's stencil changes: the drift was the source moving ahead while the board copy was the older, untouched one a previous binary deployed. The actual fix was deploying the new binary (`update-plugins.sh`), which refreshed the untouched board copy — the warning then disappeared on its own.

So the remedy is wrong whenever the drift runs source-ahead, and following it verbatim does harm.

Wanted: the warning distinguishes the two cases —
- the board copy was hand-edited (it differs from what the binary deploys): `lyx stencil promote <name>` is right;
- the board copy is untouched and the source moved ahead: say the deployed binary is older than the source and the fix is a production deploy, never promote.

Source: https://github.com/Knatte18/loomyard/issues/286 (close it in the landing commit).