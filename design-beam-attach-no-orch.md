# A beam's reed session shows pawl running with no orch and no beam overview

`beam-attach-no-orch` · Note · bug, reed, orch, pawl, workspaces

Imported from [issue #11](https://github.com/Knatte18/loomyard/issues/11).

Seen in the Models hub, reported by the operator.

A beam is started for a project, and the operator runs reed attach in the beam's directory.
The session shows pawl running and no orch strand.

Expected: the orch and the beam overview, as lym beam start brings up the beam's dobby and its orch.
Not yet diagnosed: whether the orch was never started, died, or runs in another session.