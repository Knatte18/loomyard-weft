# Report the resumed Codex darn trial run that landed

`docs/research/codex-engine-trial-darn-e2e-2026-11-05.md` gains a second section for the sandbox run `codex-greet`, resumed with the binary that carries the Codex writable roots.
The run went from Darn-Write through Code-Review, Describe, PR-Review and Finalize to `done`, with the change on `main` of `Knatte18/lym-test`.

- The first attempt's intro and its open bullet now point at the resumed run instead of saying it was not resumed or untested.
- The new section has a timeline, each check matched or not against `internal/shuttleengine/codexengine/doc.go` and `internal/creelengine/doc.go`, and the cost and turns of the resumed Codex session.
- The Codex writer committed through the git-dir roots with no sandbox refusal.
  The Go cache root, hook readings, and creel mail to the Codex strand are reported as fitting or not exercised, since the resumed session ran no Go command and the Codex events file left with the torn-down worktree.
- The landing commit credits Claude only, since the `landing.yaml` template's `credit_engines` lists no Codex entry; the report records that as an observation.

## Design decisions

- No bug note is filed: the run's one stall, the two-hour PR-Review wait, is the existing note `pr-review-wait-silent`, and the refused creel mail followed creel's sender rule.
- The cost figures come from the Codex rollout and the archived review usage record, since the archive holds no session record for the writer.
- The sandbox hub's operator decision to skip PR review came through the parent, and the continue flag was set by the orch, not by this run.

## To check by hand

Nothing: this change adds prose to one research document and runs no test.
