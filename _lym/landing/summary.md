# loomcli: beamHub and the bad-reed fixture onto registered templates

`internal/loomcli`'s two repeated test hubs, `beamHub` and `newBadReedUpFixture`, are now built once per test binary by a `hubforge` registration.
Each test copies the registered template instead of forging the hub again.
This cuts the forge time the `fabric-test-speed` work targets, with no change to what any test asserts.

## Changes

- `projectrecord_integration_test.go`: `beamHub` calls `hubforge.CopyRegistered` on `beamHubRegistration`.
  The registration's build holds the former steps unchanged: the beam pair, the two forked pairs, the hand-written board file and the shortname record.
- `smoke_helpers_test.go`: `newBadReedUpFixture` calls `hubforge.CopyRegistered` on `badReedUpRegistration`.
  The template holds the hub with the corrupted `mouse:` reed line, the seeded configs and the pair `badReedUpSlug`.
  Each site's `seed` still runs on its own copy, and the pair slug is now the constant `badReedUpSlug`.

## Design decisions

- Build steps moved as they were, so each start state is the same by construction.
- Both builds take a `testing.TB` and avoid process-wide calls, as the registration contract requires.
- `seed` stays outside the template because it differs per site.

## To check by hand

- Run the `internal/loomcli` tests, including the integration tier, which was not run for this description.
- Confirm neither build calls `os.Setenv`, `os.MkdirTemp` or `git config --global`, keeps the `TB`'s `Context()` past the build, or calls a package-global seam setter.
- Confirm each test that installs a process-global seam takes its copies before installing it.
