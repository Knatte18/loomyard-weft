# Run agents ask a question and exit before the reply; drift judge asks in its pane

`agents-exit-before-reply` · Task · bug, shuttle, creel, drift, stencil · running · Drift-Expect

Imported from [issue #186](https://github.com/Knatte18/loomyard/issues/186).

Seen: on a loom run, the recovery agent and the code fixer each mailed the parent orch a question and ended their session before the reply arrived; the reply came back from creel as returned. The drift judge instead put its question in its own pane, where no notice reaches the orch, and the run sat waiting until the operator noticed.

Workaround: the orch watched the run's idle panes and answered by creel mail; the returned replies were dropped, and the runs went on without them.

Expected: an agent that asks with --expect-reply waits for the reply before it ends, and every run agent, the drift judge included, asks its parent by creel mail rather than in its pane.