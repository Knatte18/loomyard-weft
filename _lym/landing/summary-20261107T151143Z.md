# battencli: beamHub onto a registered hubforge template

`beamHub` in `internal/battencli/beamguard_integration_test.go` builds the hub its beam-guard tests share:
the prime-anchored hub with a marked beam `b`, a member `m` forked from it, and a plain pair `p` forked from the prime, none of them pushed.
Each of its three callers paid those three pair adds per test.

The fixture now copies `beamTemplate`, a package-level `hubforge.Registration` keyed `battencli-beam`.
Its build function holds the former per-test `AddPairWith` steps, moved unchanged, so every test starts from the same hub as before.
The template is built once per test binary, and each test pays only the copy.
`beamHub` still reads the prime's worktree name from its own copy.

## Design decisions

- The base shape is the bare prime anchor and all three pairs are added in the build:
  a base shape's pairs are added with a push, and these pairs are added with `SkipPush` as before.
- `TestResolveBattenPrime` and `TestVerbs_RunFromABeam` now take their copy before `boardkit.InstallReplay`,
  so the template's build never runs under that process-global Bolt replay.
- `notices_integration_test.go`'s `noticeHub` and the other single-site fixtures are unchanged.

## Testing

The tests' assertions are unchanged.
`go vet` on `internal/battencli` with its integration build tags is clean.
