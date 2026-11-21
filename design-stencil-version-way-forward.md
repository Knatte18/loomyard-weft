# Stencil build-version refusal at Preflight appends an inapplicable config/model-spec way forward

`stencil-version-way-forward` · Note · bug, stencil, shed

Imported from [issue #175](https://github.com/Knatte18/loomyard/issues/175).

A loom run halted `blocked` at Preflight because the stencil store refused a stencil written by a newer lym build than the running one.

The refusal text ends with two way-forward clauses:

```
way forward: restart the run's driver on the current lym binary, and update that binary first if it is older than the board; way forward: fix the config or model spec it names, then re-step
```

The first clause is correct for this cause.
The second is a generic suffix that the halt path appends, and it does not apply: no config or model spec is named, and the cause is a stale binary.
Following it sends the operator to look for a config fault that does not exist.

The halt anomaly was `escalation-to-human`, and the run history holds a single `Preflight / stuck` entry.

Expected: a build-version refusal from `stencilstore` carries only its own way forward, without the generic config/model-spec suffix.

Full error as recorded:

```
seat stencils check failed: build the darn writer table: darn: DarnTable: Step 4 list: stencilstore: stencil "darn-fragment-make-change-gated" was written by lym build fded58794f7b02939b6bf8675b87c64fb3fa987d, which is newer than the running build 4c031f8b7a781e3f4fb17f48632782ded92f3a01, and this process may not render it
```

Provenance: task `bugfix-discussion-stencil`, from `loom-halt.md`.
