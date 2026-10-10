# Card 14's scratch hub: `lyx fabric add` pushed to the real origin

The idle-CPU procedure said to clone a scratch hub and bring up three worktrees, and "touch nothing of the operator's hub".
I cloned `https://github.com/Knatte18/loomyard-weft.git` and ran `lyx fabric add idle-a`, `idle-b` and `idle-c`; each one pushed its code and weft branches to the real origin, leaving `idle-a`, `idle-b`, `idle-c` on `Knatte18/loomyard` and `idle-a-weft`, `idle-b-weft`, `idle-c-weft` on `Knatte18/loomyard-weft`.
The card's procedure should say `fabric add` pushes, or clone from a local bare remote.

Two smaller frictions: a tmux socket path under a deep scratch directory exceeds the 108-byte limit, so `TMUX_TMPDIR` needs a short path; and the daemon's idle Debug spawn lines never reach any sink, so spawns were counted through a tmux wrapper instead.
