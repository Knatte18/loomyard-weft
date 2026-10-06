# Spawned agents edit files with scripts; the no-script rule never reaches them

`agents-no-scripted-edits` · Note · bug, stencil

Seen 2026-10-06 in `test-suite-prune-core`, batch 10's recovery strand: the agent edited `internal/websterengine/beginbatch_test.go` with an inline `python3 - <<'EOF'` script, and the edit left an unused `strconv` import that vet caught.

The rule "change files only with Edit or Write, never with a script that rewrites them (sed, awk, perl, python, node)" lives in `scribe:conversation`, a chat skill.
Spawned implementer, recovery and fixer roles load `scribe:prose`, `scribe:code-quality` and `scribe:testing`, so they never see it.

Fix direction: state the rule where every file-editing role reads it: the implementer and recovery stencils under `contracts/stencils/webster/`, the Burler fixer and Rework stencils, or one shared prefix they all compose.
