# Round 2 fixer-report — discussion

## Changes

All in `_lym/discussion/decision-record.md`, with a round entry in `_lym/discussion/support-log.md`.

- F1, MEDIUM, fixed as proposed.
  D2's last bullet reads the upstream tip from the records bare, never the board's tracking ref, and fails the build naming the key when the board's HEAD differs.
  D7's list of steps that stay in the test gains a push into a bare from anywhere but the board, naming `endedHub`'s `originBoard`, and the anchor-and-origin bullet says `originBoard` runs on the copy.
  A D2 criterion covers a build that pushes into the records bare after the board's push.
  The self-check refined the wording, below.
- F2, MEDIUM, fixed as proposed.
  D1 says the reader takes any directory inside a worktree and walks up as git's discovery does, and that a missing directory is an error with no walk-up, so the `junction.go` and `scratchlink.go` fallbacks still fire.
  A D1 criterion covers both.
- F3, MEDIUM, fixed as proposed.
  D2's Bound says a build runs under the seams and registrations installed when its first caller reaches it, and states the rule: no seam setter or once-only registration in a build, and a test takes its copies before installing a seam.
  The D7 review criterion checks the build, and a second D7 review criterion checks the test order.
- F4, LOW, fixed as proposed.
  D3's Bound now says a copy's board keeps the working tree its build left, with HEAD equal to its upstream.
  The first D3 criterion limits the clean-board check to a shape or a registration whose build commits what it writes.
- F5, LOW, fixed as proposed.
  Scope Out, D10 and the first Open risks line say `gate-tier-binaries-only` landed on `main` as `3aca2e50`, with `{test_targets}` opt-in and the hub's tiers not yet switched; D10 keeps both expectations, the 48 s one once the hub's tier commands carry `{test_targets}`.

## Self-check

- fix-consistency: no findings.
- fix-claims 1, MEDIUM, `originBoard` pushes through a clone in `t.TempDir()` (`originreconcile_integration_test.go:80-84`), not straight into the bare: fixed. D7 now names a push into a bare from anywhere but the board, with `originBoard` pushing through a clone outside the hub, and D2's bullet says "pushed into the bare from elsewhere".
- fix-claims 2, LOW, the D2 criterion built like `endedHub` would fail on the `TempDir` refusal before the new check: fixed. The criterion's commit comes from a clone inside the template root.
- fix-proposal 1, MEDIUM, the records bare holds several branches, so "the records bare's branch tip" is ambiguous: fixed. D2's bullet and criterion compare against the tip of the board's current branch in the records bare.
- fix-proposal 2 and fix-way-forward 1, LOW, merged: the rule covers restored seams but not a build whose result depends on a once-only registration an earlier test made: fixed. D2's Bound adds that a build reaches no code a once-only registration changes, such as a Bolt replay, and the D7 review criterion checks it.

## Disputed

None.

## Deferred

None.
