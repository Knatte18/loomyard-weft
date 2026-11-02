# Reed code button reports a wrong anchor path from inside the anchor directory

`reed-code-wrong-anchor` · Note · bug, reed

Imported from [issue #10](https://github.com/Knatte18/loomyard/issues/10).

Seen in the Models hub, reported by the operator.

A worktree has an anchor path other than ".".
A terminal runs in the anchor directory, and the operator presses the "code" button in Reed.
Reed answers that the terminal is in the wrong anchor path, though it is in the anchor directory.

Expected: the button accepts a terminal whose directory is the worktree's anchor, and refuses only a directory outside it.
Not yet diagnosed: whether the check compares against the worktree root instead of the anchor.