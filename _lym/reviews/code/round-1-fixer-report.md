---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: disputed
    ground: premise false
    rebuttal: "No recording holds a load-buffer call (a grep for \"load-buffer\" as an argument counts 0 in each testdata file; the name occurs only inside the list-commands output), and reed's launch text goes through send-keys -l in args, which the replay does match. A changed paste text also cannot replay as a match: every paste names a fresh random buffer in its args, so no recorded load-buffer call equals a later one, whatever its stdin."
    evidence:
      - internal/reedengine/io.go:135
      - internal/reedengine/io.go:161
      - internal/testkit/tmuxrec/tmuxrec.go:114
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "A bare {uid} binding replaces substrings, so a uid of 0 or one that occurs inside a number would rewrite pane ids, sizes and timestamps. The uid is bound only as tmux-<uid>, user-<uid> and user@<uid>, the places a recording spells it. The user and host name check is as proposed."
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Round 1 fixer report

## Changes

- `245270036` (F1, F2, F3, F4, F6, F7, F8, F9, F10, F11):
  - The real-tmux test is in `recordings_realtmux_test.go`, the scenarios in `scenarios_test.go`.
  - `PlaySession` no longer asserts; its callers check `Unanswered`.
  - The shell path is a `{shell}` placeholder, and a replay on another operating system skips.
    The kit may not import `os/exec`, so `reedengine.ResolveShellPath` is now exported and the kit calls it.
  - A repeated read is answered only since the last change, and a chained call is a read only when every command is.
  - The registration check is an exact pid comparison.
  - New tests cover the replay's refusals and the placeholding; `Replay.Remaining` is checked after each scenario.
  - The recordings were made again over real tmux.
  - Beyond the findings: the recording command spelled `--tier tier2`, which the flag refuses; it now reads `--tier 2`.
  - The commit carries `Loosened-Check` lines for the tests that now skip off Linux.
- `b78b3b8af` (F12): removed the paragraph from `PATTERN-test-stand-ins`.
- `d37d006fe` (F13): added the kit to `docs/overview.md`.
- `87232a981` (F15): the panic names `New` and `NewOver`.
- A follow-up commit rewraps the comment on `Replay.answer` in `tmuxrec.go` that broke mid-sentence (F4's text).
- F14: edited `_lym/landing/summary.md`, uncommitted.
  The same file also records `ResolveShellPath`, the new placeholders, the skip and the narrower repeated read.

Verification: tier0 passed over `internal/testkit/tmuxrec`, `internal/landingshed` and `internal/reedengine`.
`tmuxrec`'s tier2 passed against the new recordings.
After the F15 text change only `go vet ./internal/reedengine` ran.
`reedengine`'s tier2 tests that call the renamed function were compiled by that vet and not run.

## Self-check

- fix-consistency, MEDIUM, the OS skip against "No test skips by design": disputed, premise false.
  The PR-Review rows mean something on every operating system, so `pattern/PATTERN-test-speed.md:227` does not cover them.
  What a non-Linux host lacks is a recording made on it, which the skip line names (`internal/testkit/tmuxrec/session.go:166`),
  and `pattern/PATTERN-test-speed.md:231` allows a skip that names a missing prerequisite.
  A recording for that system removes the skip, where a `linux` build constraint would shut those tests out for good.
  This is a judgment on the pattern's wording that your reading may differ on:
  off Linux, every PR-Review test built on the shared fixture now skips, and `PATTERN-test-stand-ins`' rule
  that the reader runs over every recording at tier0 holds on Linux only.
- fix-consistency, LOW, the stale replay sentence in the change description: fixed as proposed.
- fix-proposal, MEDIUM, the same OS skip: disputed with the entry above.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

- F5, premise false: see the frontmatter.
  The recorder does drop stdin, but no recording and no replay can reach a call that carries it.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
