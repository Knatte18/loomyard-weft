---
verdict: APPROVED
---

The diff makes the change the amended entry asks for, and nothing else.
`beamHub` copies `beamTemplate`, a `hubforge.Registration` keyed `battencli-beam` with the `Shape{Anchor: "."}` base.
Its build holds the three former `AddPairWith` steps in their old order, slugs and options: `b` with `SkipPush` and `Beam`, `m` with `SkipPush` and `Parent: "b"`, `p` with `SkipPush`.

I ran `lym gate test ./internal/battencli` at tier 0 and tier 1; both exit 0 with no failures.
That settles the doubt the correctness lens left open: the template build passes `requireBoardAtUpstreamTip` with `SkipPush` adds.

Focus questions, each with its verdict:

- `beamTemplate` against the Review criterion: the build calls no `os.Setenv`, `os.MkdirTemp` or global git config.
  It keeps no `Context()` past the build, and it calls no package-global seam setter or once-only registration.
- Seam ordering: `TestResolveBattenPrime` and `TestVerbs_RunFromABeam` now take their copy before `boardkit.InstallReplay`.
  `TestCancelCmd_RemoveRefusesFromInsideTheBeam` installs no replay, so its ordering is unchanged and correct.
- D7's other conditions: the key `battencli-beam` is used once in the repository, and the base shape equals the old `CopyHub` shape.
  No step belongs in the test instead, since there is no path outside the hub, no board HEAD off its upstream tip and no push into a bare.
- Scope: the diff touches only `beamguard_integration_test.go`, leaves `notices_integration_test.go` untouched, and the commit message's claims hold in the diff.
  The commit has no `Pattern-Departure:` trailer and needs none.

The diff adds no test and no stencil text, and no PATTERN entry is declared for this part, so there is nothing further to check under those bullets.

## Rejected

- Lens code-correctness, LOW `design`: a parallel test's replay could be installed before another test's first template build.
  Rejected: the acceptance criterion binds each test's own copy-before-install order, which both tests meet.
  `InstallReplay` is a once-only registration that is never restored, so no ordering inside this package keeps it away from the build.
  The build passes in tier 1 with the replay installed by sibling tests in the same binary, and the lens itself marked the point unverified.
- Lens code-correctness: `requireBoardAtUpstreamTip` might fail the build under `SkipPush`.
  Rejected: the tier 1 run of `internal/battencli` passes, so the board is at its upstream tip after the build.
