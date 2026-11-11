# Dobby starts a released raddle task whose depends_on has not landed

`dobby-raddle-ignores-deps` · Task · bug, dobby, board · draft

Imported from [issue #107](https://github.com/Knatte18/loomyard/issues/107).

## What happened

In a beam, the orch promoted a Sketch with `lym board promote S1`, which made it a task with recipe `raddle`.
It then ran `lym board set-deps '{"slug":"<slug>","depends_on":["<other>"]}'`, where `<other>` was a task of the same beam still `running · PR-Review`, and only after that released the task with `lym board set-status '{"slug":"<slug>","status":null}'`.
`lym board get` showed `depends_on` set and the status already `running · Seed-Child` moments later: the beam's dobby started the raddle task although its dependency had not landed.

## Workaround

`lym batten pause <slug>` holds the run; the orch resumes it with `lym batten run <slug>` once the dependency lands.

## Expected

Dobby starts a released task only when every entry in its `depends_on` is done or gone, whatever its recipe, `raddle` included.
