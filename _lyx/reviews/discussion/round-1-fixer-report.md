# Round 1 fixer report — discussion

Every finding was checked against the code; each premise held, and every finding is fixed in `_lyx/discussion/decision-record.md`.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1** (D12, `lyxcwd`): `lyxcwd` now walks up from cwd to the nearest usable `.git` entry, as `--show-toplevel` does; the anchor read and gate are unchanged, `ErrCwdOutsideAnchor` and `ErrNotAGitRepo` keep their cases.
  The criterion adds a subpath anchor's directory and the outside-anchor case.
- **F2** (D2): hub builds (`NewHub`, `AddPair`, the template build) set the marker process-wide through a counter under a `hubforge` mutex, without serializing builds.
  The bound states that a parallel test's git during a build counts as fixture git, and that the report calls the split approximate where builds overlap.
  The criterion adds git spawned by `CloneAndWire` inside `NewHub`.
- **F3** (D7): only default-flag builds return the prebuilt binary; non-empty ldflags always build, once per binary and per ldflags value; the criterion adds it.
- **F4** (D11, Scope): the tier-purity banned tokens gain `hubforge.CopyHub` and `hubforge.SharedHub`; Scope's `cmd/lyx` scans and a criterion name it.
- **F5** (D11, Open risks): a changed fingerprint poisons the template; `SharedHub` and `CopyHub` check the fingerprint on entry and fail naming the fixture; the bound covers the check-to-use window; the open risk and a criterion are updated.
- **F6** (D11): a write is defined by effect, adding any in-process CLI verb and any spawned git subcommand; a read is a file read, a link read or an in-process go-git read.
- **F7** (Scope, D8): Scope adds `claudeengine` (deny pattern and steer text), the `loomcli`, `reedcli`, `orchcli` and `cmd/lyx` runners, the watchdog recognizer moving into `tmuxkit`, and `gitexec` (from F11); D8 states the move; the plan-writer note and the budget constraint follow.
- **F8** (D13, Constraints): the scan matches the selector `.RemoveStrand`, catching method values; the entry's exemption list is amended to the allowlist; the bound names wrappers under other names; the criterion covers method values.
- **F9** (D7): the bound now says the deny is a static-text guardrail that scripts, `bash -c` and shell variables pass.
  `lyx gate test` sets or strips the variable on every run, so a forged value reaches only an ungated raw run; the criterion covers the strip.
- **F10** (D6): the bound names the tier-1 transitive escape through a production call in a package that never calls `Main`.
- **F11** (D12): `gitexec` drops `GIT_DIR` and `GIT_WORK_TREE` from every git child, so spawns and in-process reads follow the same directory; the bound and a criterion are added.
- **F12** (D12): the `.git` entry reader is exported from `gitrepo` and imported by `lyxcwd`, so there is one reader; `go list -deps ./internal/gitrepo` shows no path back to `lyxcwd`.
- **F13** (D11): the template root's last element carries a fixed `hubforge` prefix no `t.TempDir` element carries, so no copy path has it as a prefix.
- **F14** (D8): the rationale quotes the Goal's wording.

`support-log.md` gains the rejected alternatives and question-ledger auto-picks behind the F2, F5, F11, F12 and F13 choices.

## Disputed

None.

## Deferred

None.
