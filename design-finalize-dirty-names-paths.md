# Finalize blocks on 'worktree dirty' without naming the tree or paths, and the loom's own run record counts as dirty

`finalize-dirty-names-paths` · Task · bug, fabric, loom

Depends on: [`machine-handover`](design-machine-handover.md)

Imported from [issue #776](https://github.com/Knatte18/loomyard/issues/776).

**What happened**

A loom run reached Finalize after Publish, PR-Align and PR-Gate were all done.
Finalize went stuck and the run halted `blocked` with anomaly `escalation-to-human` and this message:

```
fabricengine: merge preconditions failed: worktree dirty
```

The loom driver took a `goto` on Finalize and re-stepped it, and Finalize went stuck again with the same message, so a second halt note was written.
The halt notes' `parent fabric status:` section listed the dirty tracked paths in the parent pair:

- first halt: `docs/overview.md`
- second halt: `docs/overview.md` and `_lyx/shed/<slug>/status.json` in the weft (run-record) repo

**Why it is abnormal**

1. The `fabricengine` message says only "worktree dirty".
   It names neither the tree it checked nor the dirty paths, so the operator has to find them in the halt note's separate `parent fabric status:` section, or by hand.
2. `_lyx/shed/<slug>/status.json` is lyx's own run record.
   It appeared as a dirty tracked path in the parent's weft checkout between the two Finalize attempts, so lyx's own bookkeeping can make its own merge precondition fail.
   Whether the retry itself wrote it or it was already pending, the run record should not block the merge it describes.
3. The retry (`goto` then a second stuck) re-ran the same precondition without any change to the dirty state, so it could not succeed and only produced a second halt note.

**What to do**

- Make the `merge preconditions failed` error name the checked tree and list the dirty paths.
- Exclude lyx's own run-record paths (`_lyx/shed/**`) from the Finalize dirty-tree precondition, or commit them before the check.
- Do not re-step Finalize automatically when the precondition failure is unchanged; halt once.

Provenance: task `verify-diff-scope`; notes `loom-halt.md`, `loom-halt-2.md`.
