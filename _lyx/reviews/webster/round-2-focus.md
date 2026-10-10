---
round: 2
focus:
  - "Every place the lyx binary is built for a tagged test run, namely `prebuildLyx` in `internal/gatecli/prebuild.go` and in `cmd/testtiming/prebuild.go`: does each `go build` carry the slot's `-p` cap that `PATTERN-gate-slot` requires, and do the tests pin the build arguments?"
  - "`relocateHub` in `internal/hubforge/template.go` and `CopyHub`: does a copy's records checkout come out clean against its own HEAD, is that asserted by a test, and do `CopyHub`-based tests that assert a clean tree rely on it?"
  - "`Main` in `internal/testkit/tmuxkit/tmuxkit.go`: when a re-exec'd helper of the same test binary inherits `TMPDIR` and `TMUX_TMPDIR` from a parent that already ran `Main`, does its tmux socket path stay under the 107-byte `sun_path` limit when the per-package `TMPDIR` is already one level deep?"
  - "`Repo.Toplevel` in `internal/gitrepo/gitrepo.go` and anything that still resolves a worktree root outside `internal/lyxcwd`: does `PATTERN-cwd-resolution` still hold, and is any exported symbol left without a production caller?"
---

Focus entries name where to look; the review still covers the whole range.
