# Card 27 — gitexec-drop-gitdir-env

**Create:**
- `internal/gitexec#childEnv`

**Edit:**
- `internal/gitexec#runCore`
- `internal/gitexec/gitexec_test.go`

**Prosa:**
- `internal/gitexec/doc.go`

**Intent:** D12: `gitexec` drops `GIT_DIR` and `GIT_WORK_TREE` from every git child's environment, so the surviving spawns follow the directory `lyx` names exactly as the in-process reads do, and one command never reads one repository and writes another.
`childEnv` returns the process environment without those two variables, plus `extra`; `runCore` sets `cmd.Env` from it for every spawn, local and remote, the remote case appending `lowSpeedEnv` as today.
`TestRun` gains a row that sets both variables through `t.Setenv` to a second repository and runs `rev-parse --git-dir` in the first, asserting the first's `.git`; the row names the environment as its global state.
Bound, in the doc: an operator who sets either variable to point `lyx` at another repository has it ignored; `lyx`'s own post-checkout hook already unsets them.

**ImpactSummary:** Every git spawn in the tree gains an explicit environment; behaviour is unchanged unless the two variables were set, which no `lyx` code path does.
