# Card 17 — teardown-sandbox-clear

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#New`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/selfexec#LymBinary`
- `internal/lymdirs#DotLymDirName`

**Intent:**
Clear a worktree's sandbox root before `internal/pairteardown` removes the worktree (D5), covering `lym fabric remove`, Worktree-Teardown and every other route through `RemovePair`.

`Teardown` gains the seam `clearSandbox func(slug string) (bool, error)`, called in `RemovePair` after the archive and before the removal.
`New` wires it to resolve the slug's worktree the way `RemovePair`'s own removal does, and, when that worktree holds the sandbox root, `.lym/` joined with `lymdirs.HardenSandboxDirName`, to spawn `lym harden sandbox clear` there through `selfexec.LymBinary`, logged at its spawn and waited on, answering true; a worktree with no root answers false and spawns nothing.
So pairteardown learns only the root's `lymdirs` path and the verb.
When the clear fails, `RemovePair` keeps the worktree and returns the failure with "way forward: run `lym harden sandbox clear` in <worktree>, then run the removal again: `lym fabric remove <slug>`, or `lym loom resume` for a batten run's Worktree-Teardown".

`fakeSubstrates` gains the seam's fake, its answer and error fields and its call record, assigned by `teardown()`.
`TestTeardown_CallSequence` gains rows: a pair whose clear answers true runs `clearSandbox` between the archive and the removal; a failing clear keeps the worktree, removes nothing and returns the way forward; a pair whose clear answers false removes as before.

**ImpactSummary:** One pre-removal step in RemovePair, which acts only for a worktree holding a sandbox root; every other removal runs as before.
