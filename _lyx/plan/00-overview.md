---
format: 5
approved: false
language: go
---

# Plan: ubuntu-2404-support

lyx runs, and both gate tiers pass, on Ubuntu 24.04 with git 2.43 and tmux 3.4 under a 10-digit uid, without dropping behaviour on newer git and tmux, and a host below the floor fails plainly naming the tool, the version found and the floor.
The cards shorten lyxtemp's run directory names (#739), declare the floor in a new stdlib-only leaf with the D5 audit in its doc, replace `cherry-pick --empty=drop` (#743), check the floor once per process at gitexec's exec core, `planglyph.Delta`, reed's tmux runner, server spawn and attach (#747), rebuild reed's session row as two single-format loops with the prime first (#738, #740, #748), resynchronize creel's deliver tests on a probe that saw the post (#744), and make the merge-lock tests wait for the verb's lock signal.
Card 1 lands first, since no test package runs on the host before it.

## Card Index

1 — lyxtemp-short-run-names — name run directories `r` plus six hex digits, retried up to a bound
2 — toolfloor-package — the git 2.43 and tmux 3.4 floor, its version-line parser and the audit, a stdlib-only leaf
3 — cherrypick-without-empty — CherryPick runs a plain pick, skips an already-carried change and refuses an empty commit
4 — gitexec-floor-check — check git's floor once per process before the first git command, and log exec-core spawns
5 — planglyph-delta-floor — Delta checks git's floor before quarry's git spawn
6 — reed-tmux-floor-check — check tmux's floor once per resolved path before every reed tmux spawn
7 — reedcli-attach-floor — the attach command checks tmux's floor before its handover
8 — reed-session-row-prime-first — two single-format session loops, the marked prime's session first
9 — creel-deliver-marked-probe — deliver tests wait for a probe that saw the post, or for the outcome
10 — merge-lock-wait-signal — merge-lock tests wait for the verb's own lock-wait signal

## Shared Decisions

### Decision: floor-checks-cache-parsed-answers-only

- **Decision:** gitexec and reedengine each keep a process cache of the parsed floor answer, retry a probe that could not run, and admit an unparseable line with one Warn; toolfloor holds only the floor and the parser.
- **Rationale:** D4 puts the probe at each tool's first use and keeps toolfloor a leaf that spawns and logs nothing.
- **Applies to:** cards 2, 4 and 6

### Decision: no-pattern-entry

- **Decision:** no card adds a `PATTERN.md` entry line; the floor rule lives in `internal/toolfloor`'s `doc.go`, and `pattern/PATTERN-leaf-packages.md` lists the new leaf.
- **Rationale:** `PATTERN.md` stands one byte under its headroom line, and the mechanical headroom check wins over CLAUDE.md's ask, per the decision record's constraints.
- **Applies to:** cards 2, 4 and 6

## verify:

go build ./...
lyx gate test ./internal/lyxtemp ./internal/testkit/tmuxkit ./internal/toolfloor ./internal/gitrepo ./internal/fabricengine ./internal/gitexec ./internal/planglyph ./internal/reedengine ./internal/reedcli ./internal/creelengine ./internal/testkit/creelfake ./cmd/lyx
lyx gate test --tags integration ./internal/lyxtemp ./internal/testkit/tmuxkit ./internal/toolfloor ./internal/gitrepo ./internal/fabricengine ./internal/gitexec ./internal/planglyph ./internal/reedengine ./internal/reedcli ./internal/creelengine ./internal/testkit/creelfake ./cmd/lyx
lyx gate test --tags tmux ./internal/reedengine ./internal/reedcli ./internal/testkit/tmuxkit
