# battencli: build beamHub from a registered hubforge template

`internal/battencli`'s `beamHub` fixture now copies a registered `hubforge` template instead of building its hub on every call.
The template, `beamTemplate` in `beamguard_integration_test.go`, holds the prime-anchored hub with a marked beam `b`, a member `m` forked from it, and a plain pair `p`, none of them pushed.
Each caller gets a copy through `hubforge.CopyRegistered`, so the three `AddPairWith` builds run once per test binary rather than once per test.

The start state is the same as before by construction: the build function holds the former steps unchanged.
No assertion changes, and `notices_integration_test.go` is untouched.

## Design decisions

- `TestResolveBattenPrime` and `TestVerbs_RunFromABeam` now call `beamHub` before `boardkit.InstallReplay`.
  Each test takes its copy before it installs the process-global replay seam.
- The registration's build calls no `os.Setenv`, `os.MkdirTemp` or `git config --global`, and calls no package-global seam setter.

## To check

- Confirm the `internal/battencli` integration tests pass.
  They were not run as part of this description.
