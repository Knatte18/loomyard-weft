# A hub survives a lyx binary change; the no-script rule reaches spawned agents

`binary-change-and-rules` · Task · bug, enhancement, config, stencil · running · Preflight

## Goal

A hub keeps working across a `lyx` binary change, and every rule a file-editing agent must follow reaches it.

## Part 1: config reconciles itself after a binary change

### Why

From webster-dead-ends PR-Gate (2026-10-08): that task retired the batcher keys `master_base` and `startup_context`, and Loom-Preflight now refuses a `batcher.yaml` that still carries them, pointing to `lyx config reconcile --apply`.
After the deploy the hub ran a manual reconcile in the prime and in every in-flight pair.

### Proposal (operator, 2026-10-08)

The trigger is a changed `lyx` binary, not a deploy: `update-plugins.sh` is loomyard's own route, and every other repo only gets a new binary.

- Each hub keeps a stamp under its `.lyx` naming the binary that last reconciled its config.
- The first `lyx` command in the hub that runs under a different binary reconciles the prime and every pair of the hub once, logs what it migrated, and writes the stamp.
- It runs before the command itself, so the orch, `lyx orch start` and `lyx batten run` already work with the new config; it cannot wait for the recipe's `Preflight` row, since the orch runs outside any recipe.
- The recipe's `Preflight` row checks the same stamp as a safety net and runs the reconcile when it is stale.
- A reconcile that cannot apply (a hand-edited conflict) stops the command with the reconcile's own way forward rather than running on a stale config.

### Open

- Concurrency: two `lyx` commands starting at once after a binary change; the reconcile takes a hub lock.
- Whether the same stamp also covers `stencil-drift-behind-main`: a pair behind the new binary is expected after a binary change, not a source-ahead fault.

### Also from that PR-Gate

`merriamFixedContext` is a constant from one measurement, so the system-prompt-and-tools figure drifts silently as Claude Code changes; a calibrate step in `tools/tokencount` could re-measure it.

## Part 2: stencil drift behind main is not source-ahead

Imported from [issue #445](https://github.com/Knatte18/loomyard/issues/445).

Seen: `lyx loom status` in each of three task pairs (plan-glyph-precision, shuttle-state, reed-view) logs
`stencilstore: board copy has drifted from worktree source; the board copy is untouched and a binary older than the source deployed it -- run a production deploy (update-plugins.sh)` with `stencil=webster-template-master class=source-ahead`.

The pairs branched from main at fd5b0f58e; main and prod then moved to c0bf19c54, which changed `contracts/stencils/webster/webster-template-master.md`.
The installed binary is c0bf19c54 and the board copy matches c0bf19c54, so the pair's source is behind the board copy, not ahead, and a deploy fixes nothing.
The prime logs no warning.

Expected: a worktree whose source is older than the board copy (a pair that branched before the last deploy) is not classified source-ahead and does not ask for a production deploy; at most it notes that the pair is behind main.

Workaround: none needed; the warning is ignored until the pairs sync with main at Finalize.

## Part 3: the no-script edit rule reaches spawned agents

Seen 2026-10-06 in `test-suite-prune-core`, batch 10's recovery strand: the agent edited `internal/websterengine/beginbatch_test.go` with an inline `python3 - <<'EOF'` script, and the edit left an unused `strconv` import that vet caught.

The rule "change files only with Edit or Write, never with a script that rewrites them (sed, awk, perl, python, node)" lives in `scribe:conversation`, a chat skill.
Spawned implementer, recovery and fixer roles load `scribe:prose`, `scribe:code-quality` and `scribe:testing`, so they never see it.

Fix direction: state the rule where every file-editing role reads it: the implementer and recovery stencils under `contracts/stencils/webster/`, the Burler fixer and Rework stencils, or one shared prefix they all compose.
