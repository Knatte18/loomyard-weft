# Report: Codex `astra[high]` as the darn writer through the real producer

Adds `docs/research/codex/darn-writer/codex-darn-writer-bench-neve-2026-11-17/`: a report and its data for level 2 of `codex-support-levels`.
One Codex session ran the real `Darn-Write` row of the darn recipe on `ide-beam-readme-preview` from the parent of `824eb2b6`, and the run stopped before `Drift-Verify`.
The landed Claude writer's work is the answer key.
No production code and nothing under Bolt changed, and nothing was landed or pushed from the bench pair.

The report opens with the manifest records: `lym shuttle manifest darn` for both engines and the record the real Codex launch wrote.
The Codex manifest holds every item of Claude's.

What the run showed:

- Codex finished the fix, and `template.yaml` and `config_test.go` are byte-identical to the landed ones.
- Its tree passes the verify gate, whose last tier runs all passed.
- It added four lines of preview troubleshooting to `docs/overview.md`, and its change description narrates verification the description spec leaves out.
- Its diff holds 20 files of comment-only rewrites outside the entry, because the bench pair's parent branch sits ahead of the base, so the gate's comment lint saw the whole span.
- Its figures against the landed writer's are in the report: 13 min 5 s against 5 min 41 s, and no Codex cost, since `codexengine` has no pricer.
- No skill-load or idle-wake fault shows in the run that counts.

The verdict on level 2 is the operator's.

## Design decisions

- The bench ran in a pair the orch made from this task's worktree, with the board entry's record restored from the landed run's archive tag, because the entry is no longer on the board.
- The first start seeded the wrong recipe and the first reset of the base tree was undone by a sync; both are recorded in the report's setup section with how they were corrected.

## To check by hand

- Read the report's diff section against `data/codex.diff` and `data/landed.diff`.
- Remove the bench pair `cwb-codex` once the report is read.
