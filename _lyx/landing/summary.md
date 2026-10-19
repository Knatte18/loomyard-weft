# Run on Ubuntu 24.04: git 2.43, tmux 3.4, 10-digit uid
lyx now runs on a host with Ubuntu 24.04's packaged git 2.43 and tmux 3.4 under a 10-digit uid, with the same behaviour on newer git and tmux.
A host below that floor fails with an error naming the tool, the version found, the floor and the way forward.

## What changes

- **Run directory names** (`internal/lyxtemp`): a run directory is `r` plus six random hex digits, created 0700 with `os.Mkdir`.
  A taken name is retried a bounded number of times, then creation fails naming the root.
  The longest tmux socket path is 105 bytes for the widest uid, under the 107-byte limit.
  The `r` prefix is kept, so the sweep and directories left by older binaries still work.
- **`CherryPick` without `--empty`** (`internal/gitrepo`): it runs a plain `git cherry-pick <sha>`.
  When the pick stops because HEAD already carries the change, it runs `git cherry-pick --skip` and returns nil with HEAD unmoved, as before.
  A commit that was empty from the start is refused with the pick left in progress, so `dropSeedCommits` in `internal/fabricengine` aborts with `errDropReplayFailed` as on newer git.
  No `cherry-pick` argv carries `--empty` any more, which also fixes the two Bolt seed-commit rebase tests on git 2.43.
- **Reed's session row on tmux 3.4** (`internal/reedengine`): `status-format[1]` is two single-format `#{S:}` loops, so each session is drawn once with no stray comma.
  The first loop draws the prime's session, marked by the session option `@lyx_prime_session`, and the second every other reed session.
  The client's session is lit with its full name inside the loop body, with no second current-item format.
  The mark is set wherever the geometry pins run for a prime geometry, at boot and in the attach pre-flight.
  A standalone session and a pair's session are never marked.
- **Toolchain floor** (new leaf package `internal/toolfloor`): git 2.43.0 and tmux 3.4, a parser for each tool's version line, and a floor error type that is never a `*gitexec.GitError`.
  - `internal/gitexec` probes `git --version` once per process before its first git command, so `Run`, `RunStdin` and `RunGit` return the floor error on a below-floor git.
    `planglyph.Delta` calls gitexec's exported check before quarry's git spawn, which never reaches gitexec's exec core.
  - `internal/reedengine` probes `tmux -V` once per resolved tmux path, off Windows, before every tmux spawn: the `TmuxCmd` runner, the lifecycle server spawn and `lyx reed attach`.
  - `internal/gitexec` now logs every exec-core spawn through `internal/logger`, and its entry leaves the allowlist in `cmd/lyx/spawnobservability_test.go`.
  - `internal/toolfloor`'s `doc.go` records the audit: every git and tmux feature lyx uses that arrived after git 2.20 or tmux 3.0, with the version each needs.
- **Creel deliver tests** (#744): the cause was a race in the test, not the host.
  `probeWaits(before + n)` counted any probe that went back to sleep, so a wake left by an earlier post could satisfy the count before the post under test was delivered.
  The subtests in `deliver_test.go` now wait for a probe whose pause read, recorded by `creelfake.Delivery`, started after the post.
  A positive check waits for the typed line or journal event it asserts.
  The scheduler is unchanged.
- **Merge-lock tests** (`internal/fabricengine`): `launchBlockedOnLock` waited a fixed 150 ms and assumed the verb was parked on the weft write lock.
  Under slow git spawns the verb's pre-lock guard still ran and read the fixture's rewrite as tracked dirt, giving `worktree dirty` instead of `ErrForeignMergeState`.
  An unexported seam, set only from `export_test.go` and nil in production, now signals when a verb is about to wait for the lock, keyed by lock path.
  `launchBlockedOnLock` waits for that signal under a real-time deadline.

Closes #739
Closes #743
Closes #738
Closes #740
Closes #748
Closes #744
Closes #747
Closes #745

## Design decisions

- The floor check runs once per process at each tool's first use, not in the root command, so a command that needs neither tool, such as `lyx board get` without tmux, pays nothing.
  The board entry asked for a check at start-up.
- A version line that does not parse, such as a vendor or development build, is admitted with a Warn; only a parsed version below the floor is refused.
- Only a parsed result is cached; a probe that cannot run is retried on the next command.
  A long-lived daemon keeps its parsed result across a git or tmux upgrade.
- Tests reach the floor check through an injected version line, never a substituted binary.
  A `TmuxCmd` built with an exec hook never touches the process cache, so hook-driven tests keep their exact call counts.
- The tmux floor stays 3.4: a feature above it is kept optional or replaced, never met by raising the floor.
  `allow-set-title` (tmux 3.5) stays optional: on tmux 3.4 reed Warns and the pane title can be overwritten by the agent.
- On tmux 3.4 the non-prime sessions follow in name order, since `S:` has no sort option; newer tmux keeps session-id order.
- `PATTERN.md` has no headroom, so the rule that a feature must exist at the floor lives in `internal/toolfloor`'s `doc.go` and in review.
- `internal/gitkit`'s own git spawns stay unchecked; it is test plumbing no `lyx` command reaches.

## To check by hand

- Run `lyx gate test ./...` and `lyx gate test --tags integration ./...` on the Ubuntu 24.04 host with git 2.43 and tmux 3.4.
  No test installs those versions, so the real tools are covered only there; what still fails becomes a follow-up.
- Check the session row rendered on tmux 3.4 with `tmux display-message -p '#{T:status-format[1]}'`.
  `#{client_session}` inside an `#{S:}` loop was checked rendered on the development host's tmux only.
- A prime session already up when this binary arrives is marked at its next `lyx reed attach` to it, and draws in the second loop until then.
  An attach whose pre-flight degrades to the bare argv leaves it unmarked.
- The merge-lock cause rests on the code and a slowed reproduction, since the plain stress run stays green on the development host; if the Ubuntu host still fails, the cause lies elsewhere.
- A 10-digit uid with a `TMPDIR` longer than `/tmp` can still pass the socket limit; tmuxkit's existing error names `TMPDIR` as the remedy.
- Review `internal/reedcli/attach.go`: its package comment, command help and inline comments describe the attach pre-flight now that the floor check comes first, and the wording was a review finding about stale docs.
- Review the second line of the status bar in `internal/reedengine/bar.go`: a review found the time segment drawn twice in `statusFormatSessions`, and the last commit removed the duplicate; check that the time appears once and the line-2 test counts it.
- Review where the prime's session is marked at boot against an already-running prime session, in `internal/reedengine/doc.go` and `windowsize.go`: a boot marks only the session it spawns.
