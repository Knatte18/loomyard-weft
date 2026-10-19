# Run lyx on Ubuntu 24.04's toolchain (git 2.43, tmux 3.4): audit, fix and a minimum-version check

`ubuntu-2404-toolchain` · Note · enhancement, fabric, reed

Imported from [issue #747](https://github.com/Knatte18/loomyard/issues/747).

The operator's work laptop runs Ubuntu 24.04.5 LTS with its packaged git 2.43.0 and tmux 3.4, and lyx has to work there; see the host report #745.
The other development machine has newer git and tmux, so it never shows these faults, and only this host can prove a fix.

Known faults on this toolchain:
- #743: `git cherry-pick --empty=drop` does not exist in git 2.43 (used by `gitrepo.Repo.CherryPick`, called from fabricengine's `dropSeedCommits` for Bolt), and two Bolt rebase tests fail with seed-commit conflicts.
- #738: tmux 3.4 has no separate current-session format in `#{S:}`, so reed's session row draws every session twice with a stray comma.
- #744 may belong here too; not yet known.

The test suite catches only what it exercises: tmux formats are set but never checked as rendered, and git or tmux flags no test runs go unseen.

Proposal, as one task:
1. Audit every git and tmux invocation in the code and note the version each flag or format feature needs.
2. Fix what fails on git 2.43 and tmux 3.4 today, plus what the audit finds, without dropping behaviour on newer versions.
3. Add a minimum-version check for git and tmux at start-up with a clear error, so a host below the floor fails plainly instead of in odd places.
4. Prove it with `lyx gate test ./...` and `lyx gate test --tags integration ./...` green on this host.

It needs #739 fixed first, since no test package can run on this host until then.