# Plan agent can record a decision as --by parent

`plan-agent-parent-decision` · Note · bug, loom, stencil

Imported from [issue #453](https://github.com/Knatte18/loomyard/issues/453).

## What happened

In the `shuttle-send` run, the Plan-Write agent recorded a decision as the parent:

```
lyx loom decision add --by parent --title "PATTERN-shuttle-stop names the loom driver exception" ...
```

The Plan role writes its own output files only; a design call outside the decided scope goes to the parent.
`lyx loom decision add` accepted `--by parent` from inside the plan agent's session, so the record reads as the hub's decision though the hub never made it.

## Expected

`decision add --by parent` refuses when it runs inside a lyx-spawned agent session other than the parent's, or the Plan stencil's deny hooks block it, so a plan agent's scope call reaches the parent as an escalation or an open question in the plan.

## Context

The decision itself was narrow and the hub would have taken it (it names loomcli's raw removal of the loom driver's strand as an exception in the new PATTERN entry).
The defect is the forged authorship, not the content.
