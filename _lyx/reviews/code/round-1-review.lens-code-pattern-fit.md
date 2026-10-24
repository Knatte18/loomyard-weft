```yaml
lens: code-pattern-fit
outcome: reported
```

Task: code-pattern-fit lens. List each rule checked, held or broken.

**Rules checked**

- `PATTERN-agent-name` (forming, parsing or matching an agent name only via `internal/agentname`): held. `taskguard.go` uses `agentname.Parse` and `agentname.MatchesRole(name.Role, agentname.RoleOrch)`, and forms no name itself.
- `PATTERN-cliwire-sole-wiring` and the decision record's constraint (strand name read in boardcli, never in boardengine): held. `cli.go:145` sets `cfg.StrandName = strandNameFromEnv()`. boardengine has no `StrandNameEnv` or strand-name `Getenv`; its only `os.Getenv` calls are the BOARD_SKIP ones in `skipenv.go`. boardengine only imports `agentname` for parsing.
- `PATTERN-leaf-packages`: held. boardengine and boardcli are not listed there, so the new `agentname` import is unrestricted.
- `PATTERN-tolerant-records`: held. No record format changed, and `Config.StrandName` is tagged `yaml:"-"`.
- `PATTERN-refusal-way-forward`: held. This entry covers webster, shed and loom refusals, and the board refusal is outside it. The message names the note way forward anyway (`"kind":"note"`, `lyx board upsert`), and `taskguard_test.go` reaches it.
- `PATTERN-parent-directive` and `PATTERN-stencil-ownership`: held. The stencil edits only add a sentence beside the existing own-entry rule. The orch role stencil is untouched, which matches D5 being withdrawn. The darn, discussion and plan writer stencils and the discussion and plan review rubrics all carry the note-never-task rule, and no existing rule was dropped or weakened.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: held. `taskguard_test.go` has the `tier0` tag, `t.Parallel`, `t.TempDir`, `SkipGit` and no clock. The `verbs_test.go` step uses `t.Setenv`, which fits the sequential `TestCLI` step table.
- The "no new cross-cutting invariant" rule (PATTERN edit required only for a new cross-cutting invariant): held. The guard is board-module local and mirrors the existing run_settings and hub-config guards. The "Declared PATTERN changes landed" check has no declarations in the directive to verify.
- CLAUDE.md "Docs land in the same commit": held. `boardengine/doc.go`, `boardcli/doc.go` and `SKILL.md` all landed in the diff. No board entry body was edited.
- CLAUDE.md semantic line breaks and the `golang-comments` conventions: held. The new comments and doc sentences are one sentence per line, with no end-of-line comments. One long line in `boardcli/doc.go` carries a single sentence. `taskguard.go` has a file header. The doc comments start with the symbol name.
- CLAUDE.md "Notes go in git": held. Nothing was written to file-memory.
- Commit messages and `Pattern-Departure:` trailers: held. No trailer is present and none is needed, because no listed entry forbids the answer key's way.
- Parent-changed files (`cli.go`, `doc.go`, `verbs_test.go`, `boardengine/doc.go`, `SKILL.md`): held. The parent's changes (profiles defaults, run-profile wording) sit beside the task's additions without conflict. `cli.go:145` is still wired after the merge, and both docs and the SKILL.md section stay in place.
- Own-entry rule and the PATTERN-style invariants for board writes: held. The new check sits beside `checkRunLocks`, runs before `Save`, and the replay-of-a-local-commit path is not guarded (the doc says so).

**Findings:** none.

**Tool calls**
1. Read `pattern/PATTERN-leaf-packages.md` and grepped boardengine for `agentname` imports and `Getenv`: my context lacked the pattern file and a check that boardengine never reads the strand name.
