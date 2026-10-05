# Round 1 fixer report

Every finding was fixed; the `approved:` key in `00-overview.md` is untouched.

## Changes

- F1 (MEDIUM): `18-test-socket-keys.md` now lists the eight further files with a raw `kill-server` on an unminted key, and says how they were found; the survey clause stays for sources the list misses.
- F2 (LOW): `17-tmuxkit-socket-cleanup.md` states the `KillOnCleanup` bound: one path for its key, never a glob, only a socket that no server answers on, and a `TMUX_TMPDIR`-relative directory.
- F3 (LOW): `00-overview.md`'s Shared Decision now says no card before 9 adds, removes or retags a tagged test file, and names the untagged files of cards 1 and 8.
- F4 (LOW): `20-testtiming-redundancy.md` gains `Uses` of `pkgResult` and `parseLine`, and its Intent says the report header takes its numbers from that aggregation.
- F5 (NIT): `10-pattern-testing-consolidation.md` no longer names `tiersleep_test.go` as reporting a retired ID.

## Deferred

None.
