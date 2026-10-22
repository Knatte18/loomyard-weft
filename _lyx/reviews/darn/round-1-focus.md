---
round: 1
focus:
  - "`gitTagListsOnly` in `internal/burlerengine/cluster.go`: does any `git tag` form that creates, deletes, signs or moves a tag still pass, such as an option attached to its value, a short-option cluster, a positional tag name beside a list-mode option, or an option after `--`?"
  - "`commandRunsMutatingGit` skip path: does the exception stay confined to `git tag`, so every other subcommand keeps the subcommand-only rule the board entry keeps?"
  - "`--sort` and `--format` value skipping: can a skipped value word hide a tag-writing option or let a mutating command through?"
  - "`TestCommandRunsMutatingGit` rows against the covered behavior: is any listed form or option in `gitTagListsOnly` without a row, and does any new row duplicate an existing one?"
  - "Claims in `_lyx/landing/summary.md` about option sets and the unchanged halt message, checked against the diff and the `doc.go` fork-audit paragraph."
---

Seed focus from the change description; the diff and the board entry stay the measuring stick.
